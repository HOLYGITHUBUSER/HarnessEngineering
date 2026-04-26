# HarnessEngineering

HarnessEngineering is a practical knowledge base for repeatable AI-assisted engineering workflows. It collects reusable workflows, rules, memories, and sanitized skills that help agents work more reliably across real projects.

The repository focuses on concrete execution loops rather than abstract prompts: reading project context, planning changes, validating behavior, reviewing visual output, packaging artifacts, installing into real environments, and pushing verified results.

## Contents

- `workflows/` — Step-by-step operating procedures for engineering tasks.
- `rules/` — Persistent guidance and conventions for agents.
- `skills/` — Reusable task-specific instructions, with secrets removed.
- `memories/` — Lightweight contextual notes and reusable background knowledge.

## Current Workflows

- `workflows/extension-dev-loop-插件开发循环.md` defines a full VS Code extension development loop covering unit tests, webview automation, screenshot review, VSIX packaging, Cursor and Windsurf installation, configuration checks, lifecycle validation, and release hygiene.

## Principles

- Prefer project-specific evidence over assumptions.
- Validate code in the environment where users actually run it.
- Treat visual UI changes as screenshot-review tasks, not only DOM assertions.
- Keep generated artifacts and release outputs explicit.
- Remove secrets before publishing reusable skills or documentation.

## Usage

Use these files as checklists for AI agents or human operators. Copy a workflow into a project, adapt the commands to that repository, and keep the verification steps close to the actual release path.

For public repositories, never commit live API keys, passwords, tokens, or private endpoint credentials. Use placeholders and keep real credentials in local secret stores.
