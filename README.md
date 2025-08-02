import os
import functools
import traceback
from datetime import datetime

def log_step(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        page = None
        for arg in args:
            if hasattr(arg, 'screenshot') and hasattr(arg, 'locator'):
                page = arg
                break
        if page is None and args and hasattr(args[0], 'page'):
            page = args[0].page

        step_name = func.__name__.replace("_", " ").capitalize()
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        screenshot_dir = "screenshots"
        os.makedirs(screenshot_dir, exist_ok=True)

        before_path = os.path.join(screenshot_dir, f"{func.__name__}_{timestamp}_before.png")
        after_path = os.path.join(screenshot_dir, f"{func.__name__}_{timestamp}_after.png")

        print(f"[STEP] {step_name}")

        # Take BEFORE screenshot
        if page:
            try:
                page.screenshot(path=before_path, full_page=True)
                print(f"[📸] BEFORE screenshot: {before_path}")
            except Exception as e:
                print(f"[⚠️] Failed BEFORE screenshot: {e}")

        status = "passed"
        error_message = ""
        try:
            result = func(*args, **kwargs)
        except Exception as e:
            status = "failed"
            error_message = f"{type(e).__name__}: {str(e)}\n{traceback.format_exc()}"
            result = None
        finally:
            # Take AFTER screenshot
            if page:
                try:
                    page.screenshot(path=after_path, full_page=True)
                    print(f"[📸] AFTER screenshot: {after_path}")
                except Exception as e:
                    print(f"[⚠️] Failed AFTER screenshot: {e}")

        # Log step result
        log = {
            "step": step_name,
            "status": status,
            "before_screenshot": before_path if os.path.exists(before_path) else None,
            "after_screenshot": after_path if os.path.exists(after_path) else None,
            "error": error_message if status == "failed" else None,
            "timestamp": timestamp
        }

        # Optional: Write to a step log file or JSON if needed
        print(f"[LOG] Step completed: {log}")

        if status == "failed":
            raise AssertionError(f"[❌] Step failed: {step_name}\n{error_message}")

        return result

    return wrapper
