# skills

Skills for coding agents: a plan-first workflow, diff reading, Azure DevOps review, and
runtime debugging.

Installable with the [skills](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add vanenshi/skills
```

## Personalizing

Several skills ship with `<org>`, `<project>`, `<repo>` placeholders and inline
`<!-- PERSONALIZE: … -->` markers rather than hardcoded values. They are meant to be filled in
for your setup — each such skill has a **Personalize before first use** section explaining
exactly what to set and how to discover the right value. Placeholders are never literals.

## Planning

| Skill | What it does |
| --- | --- |
| `plan-draft` | Interviews you ("grill" phase), investigates the codebase, then writes one self-contained plan under `.claude/plans/` and delegates a cold review. No code changes. |
| `plan-review` | Cold-reviews a plan in a fresh context: verifies its claims against the real codebase and returns an `APPROVE` / `REVISE` verdict with prioritised findings. Read-only. |
| `plan-execute` | Walks an approved plan's task checklist apply-loop style, verifies against the plan's definition of done, and commits the ADR if one is required. |
| `plan-handoff` | Shared procedure (not a command) for writing durable handoff files when work belongs in another repo or a future session. Used by `plan-draft`, `plan-execute`, and out-of-scope-work rules. |

Workflow: `/plan-draft <goal>` → review the draft and set `status: approved` →
`/plan-execute`.

## Review and debugging

| Skill | What it does |
| --- | --- |
| `abridge-diff` | Condenses a long diff into a "reading diff" — same rows and structure, mechanical bulk removed. The model emits coordinates and a Python applier writes the output, so no line can be invented. A local port of [boldsoftware/meat](https://github.com/boldsoftware/meat) that needs no API key. |
| `azure-devops` | Driving an Azure DevOps remote, MCP-first: the `mcp__ado__*` tools are the primary surface (typed schemas, native markdown fields), the `az` CLI kept as fallback. Covers tool selection, API-level gotchas, work-item linking, and the commit-split/PR create-update workflow. Self-updating — each gotcha exists because a real run hit it. |
| `code-review-azure` | Reviews an Azure DevOps PR: a change brief (what changed, what was decided, what to ask the author) followed by parallel review agents whose findings are independently validated before anything is posted. MCP-first for all PR metadata and thread posting. Builds on `azure-devops`. |
| `debug-mode` | Hypothesis-driven debugging with runtime log instrumentation and human-in-the-loop reproduction, for bugs that cannot be diagnosed by reading code alone. Strips every trace of instrumentation when done. |

## Conventions

| Skill | What it does |
| --- | --- |
| `conventional-commits` | Formats commit messages per the qoomon conventional-commits spec: types, scopes, breaking changes, body, footer. |

Besides the throwaway plan, the workflow maintains two durable docs: `CONTEXT.md` (the
project's ubiquitous-language glossary) and `docs/adr/` (architecture decision records).

Multi-repo workspaces can use handoff files under `.claude/handoffs/` to coordinate
cross-repo work — the origin plan gates on handoff status before execution.

## Credit

The plan skill is inspired by [mattpocock/grill-me](https://github.com/mattpocock/grill-me).
