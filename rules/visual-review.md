# Visual Review

- Treat layout, wrapping, row height, selection, hover, focus, color, and webview changes as visual changes.
- Do not rely only on DOM assertions for visual behavior.
- Generate screenshots for the important states and review them before reporting success.
- For webviews, test the real bundled script or a close harness with Playwright.
- Include dark/light theme and small/large viewport coverage when the change is theme or layout sensitive.
- Keep generated screenshot directories out of git unless the project explicitly wants baseline images committed.
