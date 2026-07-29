# skills

A compact plan-first workflow for coding agents: interview → execution-ready plan → cold
review → human approval → apply-loop execution.

Installable with the [skills](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add vanenshi/skills
```

## Skills

| Skill | What it does |
| --- | --- |
| `plan-draft` | Interviews you ("grill" phase), investigates the codebase, then writes one self-contained plan under `.claude/plans/` and delegates a cold review. No code changes. |
| `plan-review` | Cold-reviews a plan in a fresh context: verifies its claims against the real codebase and returns an `APPROVE` / `REVISE` verdict with prioritised findings. Read-only. |
| `plan-execute` | Walks an approved plan's task checklist apply-loop style, verifies against the plan's definition of done, and commits the ADR if one is required. |
| `plan-handoff` | Shared procedure (not a command) for writing durable handoff files when work belongs in another repo or a future session. Used by `plan-draft`, `plan-execute`, and out-of-scope-work rules. |

Workflow: `/plan-draft <goal>` → review the draft and set `status: approved` →
`/plan-execute`.

Besides the throwaway plan, the workflow maintains two durable docs: `CONTEXT.md` (the
project's ubiquitous-language glossary) and `docs/adr/` (architecture decision records).

Multi-repo workspaces can use handoff files under `.claude/handoffs/` to coordinate
cross-repo work — the origin plan gates on handoff status before execution.

## Credit

The plan skill is inspired by [mattpocock/grill-me](https://github.com/mattpocock/grill-me).
