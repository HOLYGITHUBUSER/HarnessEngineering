# Git And Artifacts

- Check `git status --short --branch` before committing or pushing.
- Commit only when the user asks, unless the workflow explicitly requires a commit.
- Inspect staged changes before commit; do not accidentally include secrets or unrelated files.
- Follow the repository artifact policy. If release artifacts must be pushed, include them intentionally.
- If generated files are not meant to be reviewed, put them in the agreed output directory or `backup/`.
- After pushing, verify the branch is synced with its remote.
