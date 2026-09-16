# Cost-Aware Development

A Codex skill for routing repository work through a risk-based development workflow. It helps choose the smallest suitable combination of implementation, testing, and review.

## Usage

Invoke the skill with a repository task:

```text
$cost-aware-dev <task>
```

The skill uses the project-scoped agent definitions in `.codex/agents/`.

## Agent Roles

| Agent | Model and effort | Access | Purpose |
| --- | --- | --- | --- |
| `worker` | GPT-5.6 Luna, xhigh | Workspace write | Implements changes and runs focused tests |
| `planner` | GPT-6 Astra, medium | Read-only | Plans complex or ambiguous work |
| `reviewer` | GPT-5.6 Sol, high | Read-only | Reviews normal or moderately complex changes |
| `deep_reviewer` | GPT-6 Astra, medium | Read-only | Reviews high-risk changes |

Keep these defaults unless the user requests a different setup or a concrete technical risk justifies it.

## Workflow

Choose the least expensive workflow that adequately addresses the task:

| Task type | Workflow |
| --- | --- |
| Simple, localized change | `worker → focused tests` |
| Normal or moderately complex change | `worker → focused tests → reviewer` |
| Complex, ambiguous, or cross-cutting change | `planner → worker → focused tests → reviewer` |
| High-risk change | `planner → worker → focused tests → deep_reviewer` |

High-risk work includes authentication or authorization, security boundaries, transactions, race conditions, important migrations, irreversible data changes, and major architectural changes.

## Working Principles

- Preserve the user's scope and the repository's existing instructions.
- Run the narrowest relevant tests first.
- Avoid unrelated refactoring.
- Keep planners and reviewers read-only; the worker owns implementation.
- Treat review findings as actionable when they identify a concrete issue introduced by the change.
- For substantive findings, allow at most one fix-and-recheck cycle by default.
- Do not commit or push unless the user explicitly asks.
