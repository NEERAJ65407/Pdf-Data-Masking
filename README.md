# step_logger.py
import functools
import inspect
from datetime import datetime
import os

def log_step(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        page = None
        for arg in args:
            if hasattr(arg, 'screenshot'):
                page = arg
                break
        if page is None:
            print(f"[WARNING] No Playwright page found in args for {func.__name__}")
            return func(*args, **kwargs)

        step_name = func.__name__.replace("_", " ").capitalize()
        print(f"[STEP] {step_name}")

        result = func(*args, **kwargs)

        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        screenshot_dir = "screenshots"
        os.makedirs(screenshot_dir, exist_ok=True)
        screenshot_path = os.path.join(screenshot_dir, f"{func.__name__}_{timestamp}.png")

        page.screenshot(path=screenshot_path)
        print(f"[SCREENSHOT] Saved: {screenshot_path}")

        return result
    return wrapper


