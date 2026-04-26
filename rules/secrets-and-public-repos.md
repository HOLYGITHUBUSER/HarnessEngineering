# Secrets And Public Repos

- Never commit live API keys, passwords, tokens, private cookies, or credential files.
- Treat public GitHub repositories as hostile environments for secrets.
- Before adding skills, docs, logs, screenshots, or config examples, scan for credentials.
- Replace sensitive values with placeholders such as `sk-REPLACE_WITH_YOUR_KEY`, `<your-password>`, or `<your-token>`.
- If a secret was already exposed, warn the user that the secret should be rotated.
- Do not rely on `.gitignore` alone after a file has already been tracked.
