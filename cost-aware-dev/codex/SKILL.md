---
name: cost-aware-dev
description: Route repository development work through cost-aware planner, worker, reviewer, and deep-reviewer agents; use for implementation tasks that need risk-based orchestration and focused tests.
---

# Cost-aware development

Use this skill when the user asks to implement, fix, refactor, or review repository code and wants
the work routed through the project's cost-aware agent workflow. Invoke it explicitly as
`$cost-aware-dev <task>`.

## Agent roles

Use the project-scoped custom agents in `.codex/agents/` by their names. Their model, reasoning
effort, and sandbox settings are part of their role definitions:

- `worker`: GPT-5.6 Luna with `xhigh`, `workspace-write`; default implementation role.
- `planner`: GPT-6 Astra with `medium`, `read-only`; reserve for complex or ambiguous work.
- `reviewer`: GPT-5.6 Sol with `high`, `read-only`; use for normal or moderately complex work.
- `deep_reviewer`: GPT-6 Astra with `medium`, `read-only`; reserve for high-risk work.

Do not replace these role settings with a more expensive model or higher effort unless the user
explicitly requests it or a concrete technical risk justifies it. Pass concise plans and findings
between agents instead of duplicating the full repository context.

## Routing

Classify the request before spawning anything. Keep simple tasks on the cheapest sufficient path.

### Simple task

Use for localized bug fixes, straightforward UI adjustments, small validation changes, simple CRUD,
or obvious test fixes.

`worker -> focused tests -> finish`

Spawn only `worker`. The worker implements the change and runs the narrowest relevant tests. Do not
invoke `planner`, `reviewer`, or `deep_reviewer`.

### Normal or moderately complex task

Use when the change spans a few related files or layers but has no material architectural,
security, data-integrity, or concurrency risk.

`worker -> focused tests -> reviewer`

Have `reviewer` inspect the git diff first. If it reports substantive findings, send only those
concrete findings back to the same worker, then perform one targeted re-check. Do not run a second
full review or start another expensive agent for unchanged concerns.

### Complex task

Use when the work is ambiguous, architectural, cross-cutting, or planning materially reduces risk.

`planner -> worker -> focused tests -> reviewer`

Pass the planner's concise plan to `worker`. After implementation and focused tests, use the normal
review and one targeted fix/re-check cycle if substantive findings exist.

### High-risk task

Use for authentication or authorization, security boundaries, transactions, race conditions,
appointment-slot concurrency, important migrations, irreversible data changes, or major
architectural changes.

`planner -> worker -> focused tests -> deep_reviewer`

If substantive findings exist, send only those findings to the worker, then run targeted tests and
verification for the affected risk. Do not invoke `deep_reviewer` routinely. Use at most one
automatic fix/review cycle unless a remaining correctness or safety issue clearly requires another.

## Operating rules

- Preserve the user's scope and all existing repository instructions.
- Inspect the diff before unrelated files during review.
- Run focused tests first; broaden testing only for a justified change or failure.
- Do not refactor unrelated code.
- Keep planner and reviewer roles read-only; the worker owns edits.
- Close completed agent threads before spawning the next stage when they would consume the
  concurrency cap; keep the worker thread open when a reviewer needs to send it findings.
- Treat a reviewer finding as actionable only when it is introduced by the change and can be tied to
  a concrete scenario or call path.
- Do not commit or push unless the user explicitly asks.
