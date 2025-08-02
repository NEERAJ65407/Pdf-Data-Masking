def generate_custom_html_report(output_path="step_report.html"):
    html = """
    <html>
    <head>
        <title>Step-by-Step Report</title>
        <style>
            body { font-family: Arial, sans-serif; padding: 20px; }
            .step { margin-bottom: 40px; }
            .step h2 { font-size: 20px; color: #333; }
            .step img { border: 1px solid #ccc; max-width: 100%; }
        </style>
    </head>
    <body>
        <h1>Step-by-Step Test Report</h1>
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

    print(f"[✅] Custom HTML report generated: {output_path}")
