---
name: plan-handoff
description: >-
  Shared procedure for writing a durable, self-contained handoff file for
  out-of-scope work — the mechanics behind the root CLAUDE.md rule that
  handoffs replace spawn_task chips, TaskCreate backlog items, and "want me to
  also…?" offers. Not invoked directly: plan-draft, plan-execute, and
  CLAUDE.md's out-of-scope-work rule all point here for the "how".
category: Workflow
tags: [workflow, artifacts, handoff]
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Write, Bash(git log:*), Bash(git status:*), Bash(date:*), Bash(ls:*)
---

This is a shared procedure, not a standalone command. `/plan-draft` (phase 2 and the
handoff-writing phase) and `/plan-execute` (checking a listed handoff's status) both
follow these mechanics rather than duplicating them; the root `CLAUDE.md` rule that
out-of-scope work becomes a handoff instead of a `spawn_task` chip or `TaskCreate` item
also follows this procedure directly, any time it applies mid-task. See `PHILOSOPHY.md`
§"Handoff system" for why handoffs exist at all.

You are filing a **handoff**, not fixing anything. Do not edit or create any file
except the handoff itself under `.claude/handoffs/`. If you feel the urge to fix the
issue, stop — that urge is exactly what this procedure replaces. The only exception is
a trivial one-line fix on a line you're already editing for the current task; anything
more becomes this file.

**1. Determine target.**
Read this repo's local `.claude/CLAUDE.md` for a declared "Multi-repo workspace"
table.

- No table declared (this repo's current state) → `target:` is this repo's own name
  — the handoff is for a _future session_ of this same repo, not a sibling.
- Table declared and the issue clearly belongs to a listed sibling → `target:` is that
  sibling's name. If it's ambiguous which repo owns the issue, ask before writing.
- That table's cross-repo policy (inline vs handoff-only) governs whether you may also
  _edit_ a sibling directly for the current task — it does not change how you file a
  handoff for something out of scope.

**2. Gather evidence, don't invent it.**
Point at the actual `file:line`, error, or behavior that shows the problem. If this
surfaced from something you just read or ran in the current task, reuse that — don't
go re-investigate the whole area beyond what's needed to make the Problem section
concrete.

**3. Write the file.**
Template: `${CLAUDE_SKILL_DIR}/HANDOFF.md`. Date via `date +%F`. Slug: short
kebab-case from the issue. Path: `.claude/handoffs/YYYY-MM-DD-<slug>.md`.

- `status: open`
- `date:` today
- `target:` from step 1
- `origin_plan:` the plan/session that surfaced this if one exists, else
  `none — found during <task>` (name the actual task)
- `plan:` leave blank — the target repo's `/plan-draft` fills this in
- **Context** — what the caller was doing when this surfaced. One or two sentences,
  reference the current task, don't restate a plan that doesn't exist.
- **Problem** — the evidence from step 2, concrete `file:line`.
- **Contract** — the expected behavior/fix shape, precise enough that a cold
  `/plan-draft <this file>` in the target repo doesn't need to re-derive it.
- **Acceptance** — a checklist a future session can verify against (test, endpoint
  probe, manual check).

**4. Report back.**
State the handoff's path and a one-line summary — never silently. Then return to the
task that was in progress; do not start planning or executing the handoff yourself.
