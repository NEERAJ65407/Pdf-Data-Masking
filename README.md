import os
import json
import glob
from datetime import datetime
from jinja2 import Template

def parse_allure_results(allure_results_dir):
    """Parse allure-results JSON and compute stats similar to Java version."""
    features = {}  # {feature_name: [scenario_data]}
    feature_stats = {}  # {feature_name: stats_dict}

    # Totals
    total_scenarios = passed_scenarios = failed_scenarios = 0
    total_steps = passed_steps = failed_steps = skipped_steps = 0
    total_duration = 0

    for json_file in glob.glob(os.path.join(allure_results_dir, "*.json")):
        with open(json_file, "r", encoding="utf-8") as f:
            data = json.load(f)

        # Skip non-testcase JSONs (like containers)
        if data.get("status") is None:
            continue

        # Feature and scenario
        labels = {label["name"]: label["value"] for label in data.get("labels", [])}
        feature_name = labels.get("feature", "Unknown Feature")
        scenario_name = data.get("name", "Unknown Scenario")
        status = data.get("status", "unknown").upper()

        steps_data = data.get("steps", [])
        scenario_steps = len(steps_data)
        scenario_passed = sum(1 for s in steps_data if s.get("status") == "passed")
        scenario_failed = sum(1 for s in steps_data if s.get("status") == "failed")
        scenario_skipped = sum(1 for s in steps_data if s.get("status") == "skipped")

        # Duration in seconds
        scenario_duration = sum((s.get("stop", 0) - s.get("start", 0)) for s in steps_data) // 1000

        # Totals
        total_scenarios += 1
        total_steps += scenario_steps
        passed_steps += scenario_passed
        failed_steps += scenario_failed
        skipped_steps += scenario_skipped
        total_duration += scenario_duration

        if status == "PASSED":
            passed_scenarios += 1
        else:
            failed_scenarios += 1

        # Add scenario to feature
        if feature_name not in features:
            features[feature_name] = []
            feature_stats[feature_name] = {
                "passed_scenarios": 0, "failed_scenarios": 0,
                "passed_steps": 0, "failed_steps": 0, "skipped_steps": 0,
                "total_steps": 0, "duration": 0
            }

        features[feature_name].append({
            "name": scenario_name,
            "status": status,
            "steps": scenario_steps,
            "passed_steps": scenario_passed,
            "failed_steps": scenario_failed,
            "skipped_steps": scenario_skipped,
            "duration": scenario_duration
        })

        # Update feature stats
        feature_stats[feature_name]["total_steps"] += scenario_steps
        feature_stats[feature_name]["passed_steps"] += scenario_passed
        feature_stats[feature_name]["failed_steps"] += scenario_failed
        feature_stats[feature_name]["skipped_steps"] += scenario_skipped
        feature_stats[feature_name]["duration"] += scenario_duration
        if status == "PASSED":
            feature_stats[feature_name]["passed_scenarios"] += 1
        else:
            feature_stats[feature_name]["failed_scenarios"] += 1

    # Overall summary
    summary = {
        "total_scenarios": total_scenarios,
        "passed_scenarios": passed_scenarios,
        "failed_scenarios": failed_scenarios,
        "total_steps": total_steps,
        "passed_steps": passed_steps,
        "failed_steps": failed_steps,
        "skipped_steps": skipped_steps,
        "total_duration": total_duration,
        "success_rate": (passed_scenarios / total_scenarios * 100) if total_scenarios else 0,
        "steps_success_rate": (passed_steps / total_steps * 100) if total_steps else 0
    }

    return features, feature_stats, summary


def format_duration(seconds):
    h = seconds // 3600
    m = (seconds % 3600) // 60
    s = seconds % 60
    return f"{h:02d}h:{m:02d}m:{s:02d}s"


def generate_cucumber_html_report(allure_results_dir, output_file="cucumber_style_report.html"):
    """Generate Cucumber-style detailed HTML report from Allure results."""
    features, feature_stats, summary = parse_allure_results(allure_results_dir)
    summary["duration_str"] = format_duration(summary["total_duration"])

    # HTML template (tabular format with per-feature grouping)
    template = Template("""
    <html>
    <head>
        <title>Cucumber Style Report</title>
        <style>
            body { font-family: Arial; margin: 20px; }
            h1 { color: #2183cf; }
            h2 { margin-top: 30px; }
            table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
            th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
            th { background-color: #2183cf; color: white; }
            .passed { background-color: #28a745; color: white; }
            .failed { background-color: #dc3545; color: white; }
            .skipped { background-color: #ffc107; color: black; }
            .summary-table td { font-weight: bold; }
        </style>
    </head>
    <body>
        <h1>Cucumber BDD Report</h1>
        <p><b>Date:</b> {{ date }}</p>

        <h2>Execution Summary</h2>
        <table class="summary-table">
            <tr><td>Total Scenarios</td><td>{{ summary.total_scenarios }}</td></tr>
            <tr><td>Passed Scenarios</td><td class="passed">{{ summary.passed_scenarios }}</td></tr>
            <tr><td>Failed Scenarios</td><td class="failed">{{ summary.failed_scenarios }}</td></tr>
            <tr><td>Success Rate</td><td>{{ "%.2f" % summary.success_rate }}%</td></tr>
            <tr><td>Total Steps</td><td>{{ summary.total_steps }}</td></tr>
            <tr><td>Step Success Rate</td><td>{{ "%.2f" % summary.steps_success_rate }}%</td></tr>
            <tr><td>Total Duration</td><td>{{ summary.duration_str }}</td></tr>
        </table>

        {% for feature, scenarios in features.items() %}
        <h2>Feature: {{ feature }}</h2>
        <table>
            <tr>
                <th>Scenario</th>
                <th>Status</th>
                <th>Passed Steps</th>
                <th>Failed Steps</th>
                <th>Skipped Steps</th>
                <th>Total Steps</th>
                <th>Duration</th>
            </tr>
            {% for s in scenarios %}
            <tr>
                <td>{{ s.name }}</td>
                <td class="{{ s.status|lower }}">{{ s.status }}</td>
                <td>{{ s.passed_steps }}</td>
                <td>{{ s.failed_steps }}</td>
                <td>{{ s.skipped_steps }}</td>
                <td>{{ s.steps }}</td>
                <td>{{ s.duration }}s</td>
            </tr>
            {% endfor %}
        </table>
        {% endfor %}
    </body>
    </html>
    """)

    html = template.render(
        features=features,
        feature_stats=feature_stats,
        summary=summary,
        date=datetime.now().strftime("%d-%b-%Y")
    )

    with open(output_file, "w", encoding="utf-8") as f:
        f.write(html)

    print(f"✅ Detailed Cucumber-style report generated: {output_file}")
    return output_file
