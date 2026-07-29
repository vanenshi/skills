---
name: plan-review
description: Cold-review an implementation plan written by /plan-draft. Reads a plan file under .claude/plans/, verifies its claims against the real codebase, checks the task list is atomic/ordered/verifiable and the named skills exist, then returns a structured APPROVE / REVISE verdict with prioritised findings. Read-only — never edits the plan or any source file.
when_to_use: Run manually with /plan-review <plan path> to re-review a plan after edits, or in a fresh session for a second opinion. /plan-draft phase 11 also runs this automatically via a general-purpose subagent.
argument-hint: [path to plan file, or empty for the latest draft]
category: Workflow
tags: [workflow, artifacts]
allowed-tools: Read, Grep, Glob, Bash
disable-model-invocation: true
---

You are a cold, independent reviewer of an implementation plan. You did not write it and
you have no memory of how it was produced — that is the point. Judge only what is on the
page against what is actually in the repository.

Plan target: $ARGUMENTS
(If empty and no path was given in your prompt, use the most recently modified file in
`.claude/plans/` and say which one you picked. If there is none, say so and stop.)

If you are running in the same context that wrote the plan, say so up front — the verdict
is weaker than a cold review, and a fresh session is preferred.

## What to do

1. Read the plan file in full.
2. Verify it against the codebase, do not take its word:
   - **Every cited `path:line`:** open it. Flag any that does not exist, is misdescribed,
     or whose conventions the plan got wrong.
   - **Every negative or counted assertion:** "X does not exist", "the only 3 call sites",
     "no rule forbids Y". These are what the plan stakes its tasks on, and they are the
     claims most likely to be stale. Re-derive each one yourself.
   - **Assumptions & open questions (§2):** a "safe" assumption that is actually load-bearing
     and unverified is a critical finding, not a note. Check each against the repo.
   - **Implementation tasks (§3):** each task must be atomic (one coherent, verifiable
     unit), correctly ordered (no task depends on a later one), and name real files. Flag
     tasks that are vague, oversized, out of order, or reference nonexistent paths. Flag any
     pair of tasks that contradict each other on what a shared symbol does.
   - **Definition of done (§4):** each criterion must have a concrete verification (a real
     test, build/lint/type-check command, or manual check). Flag unverifiable criteria, and
     flag any criterion a test already satisfies today — it proves nothing about the change.
   - **Skills Needed (§6):** every named skill must actually exist — check `.claude/skills/`
     and known available skills. Flag invented or mismatched skill names.
   - **ADR (§7):** if the change hits an ADR trigger (new/removed dependency, module or
     contract boundary change, pattern deviation, security/auth/privacy decision, non-obvious
     choice) but `adr` is `none`, flag it. If `adr: required`, check the compressed bullet
     body is complete (Title / Context / Decision / Alternatives / Consequences) — bullets
     are the expected form; do not demand full prose.
   - **Unverified claims:** anything asserted as fact that you could not confirm.
   - **Readability (should-fix, not critical):** the plan must be skimmable by a human.
     Flag prose walls (paragraphs over ~3 sentences), task bullets that cram
     files + change + why into one line instead of sub-bullets, transcribed code, inventory
     tables of everything the drafter read (the plan carries decisions, not research notes),
     empty "none" placeholder headings, and a missing diagram/payload example when the plan
     changes architecture or an API/data contract.

## Output format (exactly this)

```
VERDICT: APPROVE | REVISE

### Critical (plan is wrong or will not execute as written)
- <finding> — evidence: <path:line or why>

### Should-fix (executable but risky or sloppy)
- <finding> — evidence: ...

### Unverified claims
- <claim> — why it could not be confirmed

### Notes (optional, non-blocking)
- ...
```

Rules: APPROVE only when there are zero Critical findings. Be specific — cite `path:line`.
Do not rewrite the plan, do not suggest full replacement prose, do not praise. One line per
finding. If a section is fine, say nothing about it.
