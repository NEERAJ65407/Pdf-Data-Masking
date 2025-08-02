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


# step_logger.py
import functools
import os
from datetime import datetime

# Collect all steps here for report generation
STEP_LOGS = []

def log_step(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        page = None

        # Detect page object in arguments
        for arg in args:
            if hasattr(arg, 'screenshot') and hasattr(arg, 'locator'):
                page = arg
                break

        if page is None and args and hasattr(args[0], 'page'):
            page = args[0].page

        if page is None:
            print(f"[WARNING] No Playwright page found in args for {func.__name__}")
            return func(*args, **kwargs)

        step_name = func.__name__.replace("_", " ").capitalize()
        print(f"[STEP] {step_name}")

        result = func(*args, **kwargs)

        # Take screenshot
        try:
            timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
            screenshot_dir = "screenshots"
            os.makedirs(screenshot_dir, exist_ok=True)
            screenshot_path = os.path.join(screenshot_dir, f"{func.__name__}_{timestamp}.png")
            page.screenshot(path=screenshot_path)
            print(f"[SCREENSHOT] Saved: {screenshot_path}")

            # Save step info for HTML
            STEP_LOGS.append({
                "name": step_name,
                "screenshot": screenshot_path
            })

        except Exception as e:
            print(f"[WARNING] Failed to take screenshot: {e}")

        return result
    return wrapper
