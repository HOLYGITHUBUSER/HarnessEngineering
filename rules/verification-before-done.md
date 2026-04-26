# Verification Before Done

- Do not report a code change as done until the relevant verification has run or the reason for skipping is clear.
- Start with focused tests for the touched behavior, then run broader tests when risk is higher.
- For TypeScript projects, compile before packaging or installing.
- For extension projects, verify both source behavior and packaged VSIX behavior when packaging is part of the delivery.
- If a test command fails because project configuration is missing, report that exact cause instead of calling it a product failure.
- Keep the final report concise: what changed, what passed, what was skipped, and remaining risk.
