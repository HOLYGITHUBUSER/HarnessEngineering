# Rules

This folder stores reusable operating rules for AI-assisted engineering work. Rules are short, durable instructions that should shape how an agent behaves across projects.

## Rule Index

| Rule | Purpose |
|------|---------|
| [`engineering-baseline.md`](./engineering-baseline.md) | Read first, preserve existing work, keep changes scoped. |
| [`secrets-and-public-repos.md`](./secrets-and-public-repos.md) | Never publish live secrets; sanitize before committing. |
| [`verification-before-done.md`](./verification-before-done.md) | Match verification depth to risk before reporting done. |
| [`visual-review.md`](./visual-review.md) | Screenshot-review visual UI changes, especially webviews. |
| [`git-and-artifacts.md`](./git-and-artifacts.md) | Commit/push safely and follow project artifact policy. |

## How To Use

Copy relevant rules into a project-specific rule system, or ask the agent to read this folder before starting a task. Prefer small rules with concrete behavior over broad principles.
