# Personal Multi-Agent Workflow

## Agent routing

- Keep the primary agent as the orchestrator, decision-maker, and final reviewer. The primary agent owns requirements, decomposition, integration, verification, and the user-facing response.
- Treat ambiguous, multi-step, cross-cutting, architectural, security-sensitive, research-heavy, or difficult debugging work as complex. Keep that work on the primary agent or delegate an independent complex subtask to `complex_worker`.
- Delegate clear, narrow, low-ambiguity, independently verifiable work to `simple_worker`. Good candidates include targeted exploration, mechanical edits, focused test additions, formatting, summarization, and repetitive checks.
- Use `reviewer` for an independent correctness review when a change is risky, complex, or explicitly requested for review.
- Do not give `simple_worker` architectural decisions, unresolved requirements, security-critical judgment, final review ownership, or broad multi-file tasks.
- Do not delegate work whose handoff and integration cost exceeds doing it directly.

## Coordination

- Give each subagent one bounded objective, explicit scope, relevant constraints, and a concrete return format.
- Parallelize only independent work. Avoid overlapping write scopes; prefer parallel reads and checks.
- Wait for delegated work that the result depends on. Inspect the evidence and diff yourself rather than accepting a subagent conclusion uncritically.
- The primary agent makes final decisions, resolves conflicts between agents, performs or confirms appropriate verification, and produces the final response.

## Proactive multi-agent delegation

- The user explicitly requests proactive multi-agent delegation for all tasks governed by this file.
- For every non-trivial task containing at least one independent, bounded workstream, the primary agent must spawn an appropriate subagent.
- Delegate narrow exploration, mechanical implementation, focused tests, and repetitive checks to `simple_worker`.
- Use `complex_worker` for an independent complex workstream and `reviewer` for risky changes.
- The primary agent retains requirements, integration, verification, and final-response ownership.
- A task may remain entirely on the primary agent only when it is short or cannot be divided into independent work. Briefly state that reason.
