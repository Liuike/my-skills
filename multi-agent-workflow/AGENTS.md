# Multi-Agent Workflow

## Agent routing

- Keep the primary agent as the orchestrator, decision-maker, integrator, and final reviewer. The primary agent owns requirements, decomposition, verification, and the user-facing response.
- Treat ambiguous, multi-step, cross-cutting, architectural, security-sensitive, research-heavy, or difficult debugging work as complex. Keep that work on the primary agent or delegate a bounded, independent complex subtask to a high-capability worker.
- Delegate clear, narrow, low-ambiguity, independently verifiable work to a lightweight worker. Good candidates include targeted exploration, mechanical edits, focused test additions, formatting, summarization, and repetitive checks.
- Use an independent reviewer for correctness, regression, security, or test-coverage review when a change is risky, complex, or explicitly requires review.
- Do not assign architectural decisions, unresolved requirements, security-critical judgment, broad multi-file changes, or final-review ownership to a lightweight worker.
- Do not delegate work when the coordination and integration cost is likely to exceed the value of delegation.

When the environment provides named roles, map high-capability complex work to `complex_worker`, narrow work to `simple_worker`, and independent review to `reviewer`.

## Coordination

- Give every delegated task one bounded objective, explicit scope, relevant constraints, and a concrete return format.
- Tell workers that other agents may be editing the same workspace. Assign non-overlapping write scopes and require workers to preserve and accommodate changes made by others.
- Parallelize only independent work. Prefer parallel exploration and verification when implementation scopes would overlap.
- Wait for every delegated result that the final outcome depends on.
- Inspect returned evidence and diffs before integration; do not accept a subagent's conclusion without appropriate verification.
- The primary agent makes final decisions, resolves conflicts, confirms verification, and produces the final response.

## Proactive delegation

- When subagent tooling is available, delegate at least one independent, bounded workstream for every non-trivial task that can be usefully decomposed.
- Use lightweight workers for narrow exploration, mechanical implementation, focused tests, and repetitive checks.
- Use high-capability workers for independent complex workstreams and reviewers for risky or high-impact changes.
- Keep requirements, integration, final verification, and communication with the user on the primary agent.
- Keep a task entirely on the primary agent only when it is short, cannot be divided safely, or delegation would add more overhead than value. Briefly state the reason.
- If subagent tooling is unavailable, complete the work directly while preserving the same decomposition, verification, and review standards.
