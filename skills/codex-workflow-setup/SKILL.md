---
name: codex-workflow-setup
description: Install or update the multi-agent workflow, general AGENTS.md, and skills stored in Liuike/my-skills. Use when setting up Codex from this repository or refreshing an existing installation; do not use for unrelated Codex configuration.
---

# Codex Workflow Setup

Install only the components the user requests from the repository containing this skill.

## Procedure

1. Locate the repository root containing `AGENTS.md`, `multi-agent-workflow/`, and `skills/`. If those files are unavailable, ask for the clone's location rather than reconstructing them.
2. Inspect the destination before writing. Resolve the Codex home from `CODEX_HOME`, falling back to `~/.codex`.
3. Before changing any destination file, create a unique backup directory and preserve every existing file that will be replaced or merged, including config-only updates.
4. Install `multi-agent-workflow/AGENTS.md` as the global Codex instructions and its TOML role files under the Codex home's `agents/` directory.
5. Merge the workflow's `[features]` and `[agents]` keys into the existing `config.toml`. Preserve unrelated settings and never create duplicate TOML tables. Do not copy credentials, caches, databases, logs, project trust entries, plugins, or MCP configuration from another Codex home.
6. Install requested personal skills into `$HOME/.agents/skills/`, or repository-scoped skills into the target repository's `.agents/skills/` directory. Prefer symlinks for a maintained clone. Do not overwrite an existing skill without comparing it and preserving local changes.
7. Treat the repository-root `AGENTS.md` as a project-instruction template. Copy it only when the target has no instructions; otherwise merge it with the target's project-specific rules.
8. Validate changed TOML and skill files, confirm the expected roles and instruction files are discoverable, and tell the user to restart Codex if the running client has not reloaded them.

Keep model choices from the packaged role files unless the user requests alternatives or the configured models are unavailable. Report every installed, merged, skipped, or backed-up path.
