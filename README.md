# Codex workflow and skills

This repository contains three independent pieces:

- `multi-agent-workflow/`: the complete personal Codex multi-agent setup from the Codex home directory, including global orchestration instructions, custom agent definitions, and the required configuration fragment.
- `skills/`: independently installable Codex skills.
- `AGENTS.md`: general-purpose project instructions adapted from `modula/AGENTS.md`, with Modula-specific paths, experiment tracking and W&B conventions, and Oscar login-node rules removed.

## Repository layout

```text
.
|-- AGENTS.md
|-- README.md
|-- multi-agent-workflow/
|   |-- AGENTS.md
|   |-- config.toml
|   `-- agents/
|       |-- complex-worker.toml
|       |-- reviewer.toml
|       `-- simple-worker.toml
`-- skills/
    `-- oscar-sbatch-etiquette/
        |-- SKILL.md
        `-- agents/openai.yaml
```

## Install the multi-agent workflow

Codex uses `~/.codex` as its home directory unless `CODEX_HOME` is set. The workflow's `AGENTS.md` is global guidance, while the TOML files define the three custom roles it names.

1. Clone the repository and enter it:

   ```bash
   git clone https://github.com/Liuike/my-skills.git
   cd my-skills
   ```

2. Resolve the Codex home directory and create the custom-agent directory:

   ```bash
   MY_CODEX_HOME="${CODEX_HOME:-$HOME/.codex}"
   mkdir -p "$MY_CODEX_HOME/agents"
   ```

3. Review and back up any existing global files before replacing or merging them. This creates a unique backup directory on every install and preserves existing custom roles with the same names:

   ```bash
   mkdir -p "$MY_CODEX_HOME/backups"
   MY_BACKUP_DIR="$(mktemp -d "$MY_CODEX_HOME/backups/my-skills.XXXXXX")"
   mkdir -p "$MY_BACKUP_DIR/agents"
   test ! -f "$MY_CODEX_HOME/AGENTS.md" || cp -p "$MY_CODEX_HOME/AGENTS.md" "$MY_BACKUP_DIR/AGENTS.md"
   test ! -f "$MY_CODEX_HOME/config.toml" || cp -p "$MY_CODEX_HOME/config.toml" "$MY_BACKUP_DIR/config.toml"
   for MY_AGENT_FILE in complex-worker.toml reviewer.toml simple-worker.toml; do
       test ! -f "$MY_CODEX_HOME/agents/$MY_AGENT_FILE" || cp -p "$MY_CODEX_HOME/agents/$MY_AGENT_FILE" "$MY_BACKUP_DIR/agents/$MY_AGENT_FILE"
   done
   ```

4. Install the global workflow instructions and custom roles:

   ```bash
   cp multi-agent-workflow/AGENTS.md "$MY_CODEX_HOME/AGENTS.md"
   cp multi-agent-workflow/agents/*.toml "$MY_CODEX_HOME/agents/"
   ```

5. Merge `multi-agent-workflow/config.toml` into `$MY_CODEX_HOME/config.toml`. If `[features]` or `[agents]` already exists, add or update the keys inside the existing table; do not append a duplicate TOML table.

   The resulting configuration must contain:

   ```toml
   [features]
   multi_agent = true

   [agents]
   enabled = true
   default_subagent_model = "gpt-5.6-luna"
   default_subagent_reasoning_effort = "xhigh"
   ```

6. Restart Codex so it reloads the configuration and custom agent files.

The role files pin `complex_worker` and `reviewer` to `gpt-5.6-sol`, and `simple_worker` to `gpt-5.6-luna`. Change those values if the models are unavailable in your workspace or you prefer a different cost/quality balance.

## Install the Oscar sbatch skill

Install personal skills separately from the multi-agent workflow:

```bash
mkdir -p "$HOME/.agents/skills"
ln -s "$(pwd)/skills/oscar-sbatch-etiquette" "$HOME/.agents/skills/oscar-sbatch-etiquette"
```

The symlink keeps the installed skill synchronized with this clone and fails safely if that destination already exists. Codex supports symlinked skill directories and detects skill changes automatically. Restart Codex if the skill does not appear, then use `/skills` or mention `$oscar-sbatch-etiquette` explicitly.

For a repository-scoped installation, copy the skill to `<repo>/.agents/skills/oscar-sbatch-etiquette` instead.

## Use the general-purpose AGENTS.md

If the target project does not already have an `AGENTS.md`, copy the repository-root file into it, then add that project's architecture, commands, tests, and operational constraints:

```bash
cp AGENTS.md /path/to/project/AGENTS.md
```

If the project already has an `AGENTS.md`, do not run the copy command; merge the reusable sections instead. Project instructions are layered after the global file, and instructions in deeper directories take precedence over broader ones.

## Verify the setup

If `uv` is installed, validate the TOML syntax with an isolated Python 3.11 runtime:

```bash
uv run --no-project --python 3.11 python -c 'import pathlib, sys, tomllib; tomllib.loads(pathlib.Path(sys.argv[1]).read_text())' "$MY_CODEX_HOME/config.toml"
uv run --no-project --python 3.11 python -c 'import pathlib, sys, tomllib; [tomllib.loads(pathlib.Path(p).read_text()) for p in sys.argv[1:]]' "$MY_CODEX_HOME"/agents/*.toml
```

Start a new Codex session and verify global instruction discovery:

```bash
codex --ask-for-approval never "Summarize the current instructions."
```

In an interactive session, use `/agent` to inspect agent threads and `/skills` to confirm the Oscar skill is available.

## References

- [Custom instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md/)
- [Subagents and custom agent configuration](https://developers.openai.com/codex/subagents/)
- [Build and install skills](https://developers.openai.com/codex/skills/)
- [Codex configuration reference](https://developers.openai.com/codex/config-reference/)
