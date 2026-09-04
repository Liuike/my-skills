# Repository Instructions

## Scope and precedence

- Follow the nearest `AGENTS.md` or `AGENTS.override.md`; instructions closer to the current working directory take precedence.
- Prefer repository evidence over assumptions.
- Keep project-specific architecture, commands, and operational rules in the repository that owns them instead of adding them to global guidance.

## Working style

- Optimize for correctness, clarity, and minimal scope. Fix the root cause without unrelated refactors.
- Follow existing architecture, naming, formatting, and dependency patterns. Search for prior art before adding helpers or abstractions.
- Make low-risk assumptions explicit. Ask the user when missing information could materially change behavior, scope, cost, or external state.
- Parallelize independent reads or checks when supported; do not force parallelism when later work depends on earlier results.
- For complex or multi-step work, maintain a short plan and keep it current. Skip planning overhead for straightforward tasks.
- Lead long responses with a concise technical takeaway or executive summary.

## Implementation

- Preserve current behavior unless the task explicitly changes it.
- Keep edits cohesive and reviewable; avoid speculative cleanup and repeated micro-patches.
- Reuse existing utilities and patterns. Do not duplicate logic that already has a clear home.
- Preserve type safety. Avoid broad casts, broad exception handlers, silent fallbacks, and swallowed errors.
- Validate inputs and surface failures using repository conventions; do not return success-shaped results after errors.
- Add comments only when intent or a non-obvious constraint is not clear from the code.
- Do not add or upgrade production dependencies without explicit user approval.
- Do not hand-edit generated files; change their source or generator and regenerate them.

## Repository safety

- The worktree may contain user changes. Never discard, overwrite, or reformat unrelated changes.
- Inspect overlapping edits before modifying a dirty file. Stop and ask only when the overlap makes a safe merge ambiguous.
- Never run destructive commands such as `git reset --hard`, `git clean -fd`, or `git checkout -- <path>` unless explicitly requested.
- Do not amend commits, force-push, create commits, or push branches unless explicitly requested.
- Keep secrets, credentials, machine-specific paths, and private data out of committed files and logs.

## Verification

- Define completion in observable terms: requested behavior is implemented, relevant checks pass, and the diff contains no unrelated changes.
- Run the narrowest relevant tests first, then broader lint, type-check, build, or integration checks when justified.
- Add or update tests when behavior changes or a regression would otherwise be plausible.
- Review the final diff for correctness, regressions, error handling, security, and accidental scope growth.
- Never claim a check passed unless it was run. Report skipped or failed checks with the reason and exact command.

## Reviews and research

- For review requests, lead with concrete findings ordered by severity and include file and line references. Emphasize bugs, regressions, security risks, and missing tests.
- If no findings are found, say so and name any residual risks or untested areas.
- For unfamiliar or fast-changing APIs and tools, verify current behavior before coding. Prefer primary documentation, then corroborate with issue trackers, maintainer discussions, and reputable community reports.
- Treat forum advice as anecdotal unless supported by code, documentation, tests, or multiple independent reports.

## Experiments and reproducibility

- Put durable experimental choices in version-controlled configurations or executable launchers rather than shell history.
- Record an experiment's purpose, configuration, identifiers, status, results, and follow-up in the repository's established tracking system.
- Record failed and cancelled runs as diligently as completed runs.
- Prefer semantic experiment and run names that remain understandable without external scheduler logs.
