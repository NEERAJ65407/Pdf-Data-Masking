# html_reporter.py or end of test_lxs_declined.py
from step_logger import STEP_LOGS
import os

def generate_custom_html_report(output_path="step_report.html"):
    html = """
    <html>
    <head>
        <title>Test Step Report</title>
        <style>
            body { font-family: Arial; padding: 20px; }
            h2 { color: #444; }
            .step { margin-bottom: 30px; }
            img { border: 1px solid #ccc; max-width: 100%; }
        </style>
    </head>
    <body>
        <h1>LXS Test Step Report</h1>
    """

    for step in STEP_LOGS:
        html += f"""
        <div class="step">
            <h2>{step['name']}</h2>
            <img src="{step['screenshot']}" alt="{step['name']}">
        </div>
        """

    html += "</body></html>"

    with open(output_path, "w") as f:
        f.write(html)

    print(f"[✅] Custom HTML report generated at: {output_path}")
