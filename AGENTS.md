# Engineering and Research Instructions

## Scope and precedence

- Follow the nearest `AGENTS.md` or `AGENTS.override.md`; instructions closer to the working directory take precedence.
- Prefer repository evidence over assumptions. Keep project-specific architecture, commands, and operational rules in the repository that owns them.
- Optimize for correctness, clarity, reproducibility, and minimal scope. Fix root causes without unrelated refactors.

## Working style

- Follow existing architecture, naming, formatting, and dependency patterns. Search for prior art before adding helpers or abstractions.
- Make low-risk assumptions explicit. Ask the user when missing information could materially change behavior, scientific conclusions, scope, cost, or external state.
- Parallelize independent reads and checks when useful; do not force parallelism when work is coupled.
- Maintain a short plan for complex work. Lead long responses with a concise technical takeaway.
- Do not leave partial implementations or unsupported conclusions. State exactly what remains when completion is blocked.
- For unfamiliar or fast-changing APIs, tools, and research methods, verify current behavior with primary documentation or source. Treat forum advice as anecdotal unless corroborated by code, documentation, tests, or multiple independent reports.

## Engineering standards

- Preserve behavior unless the task explicitly changes it. Keep edits cohesive and reviewable.
- Do not add silent fallbacks, swallowed exceptions, or success-shaped results after failures. Supported alternate paths must be explicit and tested; unexpected states must fail loudly with actionable context.
- Diagnose from evidence. Before applying a bug fix, identify a concrete signal such as a log, traceback, failing assertion, response, configuration value, or minimal reproduction.
- Reuse existing utilities and preserve type safety. Avoid broad casts, broad exception handlers, duplicated logic, and speculative abstractions.
- Validate inputs at system boundaries and propagate errors according to repository conventions.
- Document modules, classes, public APIs, and non-trivial functions. Use inline comments to explain intent, constraints, or tradeoffs that the code cannot express clearly.
- Make behaviorally significant parameters explicit at call sites. Avoid defaults that hide environment, product, or scientific choices; use conventional defaults only when they are stable and unambiguous.
- Pin runtime and research dependencies to exact versions in the repository's lockfile or environment specification. Treat updates as deliberate, reviewable changes.
- Do not add or upgrade production dependencies without explicit user approval.
- Do not hand-edit generated files; change their source or generator and regenerate them.

## Integrations, state, and observability

- Before integrating an unfamiliar external tool, service, or library, read its official documentation or source and use its intended extension mechanism.
- Design for the deployed environment. Do not hardcode localhost, machine-specific paths, or development-only configuration. Require environment-specific values and fail clearly when they are missing.
- Treat database writes and irreversible external effects as one consistency problem. Do not commit state that falsely claims an effect succeeded. Use deliberate ordering, idempotency, transactions, or an outbox as appropriate, and document retry behavior.
- Log significant actions with enough context to identify what happened, to which resource, and by which actor. Prefer stable identifiers and one event per line.
- Never log credentials, keys, secrets, tokens, passwords, password hashes, full JWTs, raw sensitive payloads, or unnecessary personal data.

## Research and reproducibility

- Put scientific choices in version-controlled configurations or executable launchers, including datasets, preprocessing, seeds, model variants, hyperparameters, resource requirements, and evaluation criteria.
- Separate exploratory work from durable experiments. Promote important runs into auditable, rerunnable artifacts rather than relying on notebooks or shell history alone.
- When the user asks to execute an experiment, run the full relevant experiment unless they explicitly request a smoke test or dry run and the required compute is authorized and available. For ordinary implementation work, use proportionate verification instead. Never present dry-run output as a scientific result.
- Record purpose, hypotheses, configuration, identifiers, status, results, failures, and follow-up in the repository's established experiment ledger.
- Record failed and cancelled runs as diligently as completed runs. Preserve provenance for datasets, checkpoints, and generated results.
- Prefer semantic experiment names that remain understandable without scheduler logs.

## W&B experiment tracking

When a project uses W&B, apply these rules unless nearer repository instructions define a different tracking contract:

- Before a durable experiment, add a planned entry to the repository's run log, select or create a versioned protocol preset, and put scientific choices in a versioned executable launcher.
- After W&B creates a run, record its exact run ID, URL, resolved configuration, status, experiment ID, preset or protocol ID, and purpose in the repository's machine-readable registry and narrative run log.
- Record failed and cancelled W&B runs, including the outcome and intended follow-up.
- Run the repository's experiment-registry or preset validator after changing tracked experiment metadata. If no validator exists, validate the registry format and referenced artifacts directly.
- Use one durable executable launcher per scientific purpose, with semantic W&B metadata and a dry-run path. When the repository uses shell or Slurm launchers, prefer a self-submitting `.sh` file with `DRY_RUN=1`.
- Treat W&B as the source of truth for scientific metrics and results. Durable experiments must not create untracked local plots or result summaries; versioned publication artifacts remain allowed when the repository requires them.
- Verify that submitted runs start successfully. Do not continuously monitor them unless the user explicitly requests monitoring.

### W&B naming

- Project: environment or top-level research project only.
- Group: shared experiment context.
- Run name: factors that vary within the group.
- When the trainer appends seed and timestamp, do not duplicate them in the run-name core. Otherwise include the identifiers needed to distinguish runs.
- Prefix obsolete experiments or groups with `[OUTDATED]` instead of silently reusing their names.

## Verification and review

- Define completion in observable terms: requested behavior is implemented, relevant checks pass, scientific claims are supported, and the diff contains no unrelated changes.
- Run the narrowest relevant tests first, then broader lint, type-check, build, integration, or experiment checks when justified.
- Add regression tests when behavior changes or a recurrence is plausible.
- Before sign-off, explicitly consider edge cases, race conditions, unintended downstream consequences, and inconsistencies with data models, APIs, or scientific assumptions.
- For reviews, lead with concrete findings ordered by severity and include file and line references. If no findings exist, say so and identify residual risks or untested areas.
- Never claim a check or experiment passed unless it was run. Report skipped or failed checks with the reason and exact command.

## Repository safety

- Preserve unrelated user changes. Inspect overlapping edits before modifying a dirty file, and ask only when a safe merge is ambiguous.
- Never run destructive Git commands, amend commits, force-push, create commits, or push branches unless explicitly authorized.
- Keep secrets, credentials, private data, and machine-specific paths out of committed files and logs.

## Maintaining these instructions

- When a user states a preference that appears to be a durable engineering or research standard, ask whether it should be added to the applicable `AGENTS.md`.
- Add a new standard only after confirmation, and include enough rationale to make its intended scope clear.
