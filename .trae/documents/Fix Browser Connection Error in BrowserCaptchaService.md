The user encountered an error when starting the application: `Exception: Failed to connect to browser`. This is happening in `src/services/browser_captcha_personal.py` when initializing `nodriver`.

The error message suggests:
```
Failed to connect to browser
One of the causes could be when you are running as root.
In that case you need to pass no_sandbox=True
```
Although the user is on Windows (where root/sandbox issues are less common than Linux), the error indicates `nodriver` is failing to connect to the browser instance it tried to start.

I need to investigate `src/services/browser_captcha_personal.py` and modify the `uc.start` call.

**Plan:**
1.  **Modify `src/services/browser_captcha_personal.py`**:
    *   In the `initialize` method, update the `uc.start` arguments.
    *   It seems `nodriver` might be having trouble finding the browser executable or connecting to the debug port.
    *   I will try removing `sandbox=False` (which might be the default or causing issues) or explicitly setting it to `True` if that's what the error suggests (though usually `no_sandbox=True` is the fix).
    *   Wait, the error says "you need to pass no_sandbox=True". The code currently has `sandbox=False`. I should check if `nodriver`'s `start` function accepts `sandbox` or if it should be part of `browser_args`.
    *   The code currently has:
        ```python
        self.browser = await uc.start(
            headless=self.headless,
            user_data_dir=self.user_data_dir,
            sandbox=False,  # nodriver 需要此参数来禁用 sandbox
            browser_args=[
                '--no-sandbox',
                ...
            ]
        )
        ```
    *   The error message explicitly says "you need to pass no_sandbox=True".
    *   I will change `sandbox=False` to `sandbox=True` (which in `nodriver` usually means "use sandbox", but the error message might be referring to a specific `no_sandbox` argument or the logic is inverted).
    *   Actually, looking at `nodriver` documentation (or common usage), `sandbox=False` usually *disables* the sandbox (which is what we want).
    *   However, let's try to remove `sandbox=False` and rely on `browser_args=['--no-sandbox']` which is already there, OR try to find the browser executable path if it's missing.
    *   A common issue on Windows with `nodriver` is not finding the Chrome binary.
    *   I'll try to explicitly find the Chrome binary path on Windows and pass it to `uc.start`.

**Refined Plan:**
1.  **Analyze the code**: The current code passes `sandbox=False`. The error says "pass no_sandbox=True".
2.  **Edit `src/services/browser_captcha_personal.py`**:
    *   I will verify if I should use `browser_executable_path` to explicitly point to Chrome/Edge if it's not found.
    *   But first, I will try to follow the error message's advice literally if possible, or adjust the sandbox setting.
    *   Wait, `nodriver`'s `start` function signature usually takes `sandbox` (bool). If `sandbox=True` (default), it enables sandbox. If `sandbox=False`, it adds `--no-sandbox`.
    *   The error message from `nodriver` source code `raise Exception("Failed to connect to browser ... need to pass no_sandbox=True")` suggests that maybe it failed *because* it couldn't connect, and it's *guessing* the cause.
    *   A more likely cause on Windows is that the browser didn't start correctly or the port was blocked.
    *   I will try to add `browser_executable_path` auto-detection for Windows.
    *   I will also try to comment out `sandbox=False` to see if the default behavior works better, or strictly follow "pass no_sandbox=True" if that's a valid kwargs (it might be `no_sandbox` instead of `sandbox`).
    *   Let's check `nodriver` source code or common patterns.
    *   Actually, `uc.start` usually takes `browser_executable_path`.
    *   I will add a helper to find Chrome on Windows.

3.  **Action**:
    *   I'll search for where Chrome is installed on Windows.
    *   Then update `src/services/browser_captcha_personal.py` to:
        1.  Detect Chrome path.
        2.  Pass it to `uc.start(..., browser_executable_path=path)`.
    *   Also, I'll try changing `sandbox=False` to `sandbox=True` just in case the error message is misleading (or the opposite).

Let's start by trying to find the Chrome path.