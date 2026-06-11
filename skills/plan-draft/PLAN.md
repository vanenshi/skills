---
status: draft # draft -> approved -> done
date: YYYY-MM-DD
adr: none # none | required | NNNN (set to the committed ADR number once written)
skills: [] # skills /plan-execute should preload, e.g. [coding-standards, ui-testing]
---

# <Plan title>

## 1. Goal & scope

<Goal restated in plain words.>

**In scope:**

**Out of scope:**

## 2. Codebase findings

<Concrete files, modules, and existing patterns this touches. Real paths only.>

## 3. Assumptions & open questions

<What remains after the grill phase. Decisions settled during grilling don't belong here —
they live in the tasks, the ADR, or CONTEXT.md.>

**Safe assumptions:**

**Blocking (resolve before approving):**

**Terms resolved → `CONTEXT.md`:** <list canonical terms added/changed, or "none">

**ADRs / decisions that constrained this plan:** <docs/adr refs read in phase 2, or "none">

**Glossary conflicts surfaced:** <terms that clashed with CONTEXT.md and how resolved, or "none">

**Residual risks / unknowns for reviewer:**

## 4. Implementation tasks

<The execution checklist. /plan-execute walks these top-to-bottom and flips - [ ] -> - [x]
as each lands. One task = one coherent, verifiable unit (openspec apply-loop style):
keep each small enough to land and check on its own; order them so each builds on the
last. Every task names the files it touches, the change, and why.>

- [ ] **<task>** — files: `...` — change: `...` — why: `...`
- [ ]

## 5. Definition of done

- [ ] <criterion> — verified by: `<test / command / manual check>`
- [ ]

## 6. Risk & rollback

- **Blast radius:** <scope of impact — other modules, services, users>
- **What could break:** <specific failure modes and their severity>
- **Rollback:** <how to undo if needed>
- **Irreversible parts:** <migrations, data changes, contract/API changes that can't easily revert>

## 7. Skills Needed

<The skills /plan-execute should load to carry out section 4. List by exact skill name so
/plan-execute can invoke each one (and so it can preload them via the `skills:` frontmatter).
For each, say which task(s) it serves and why. Use only skills that exist in this project
— check `.claude/skills/` and the available-skills list. If none apply, write "None" and
one line of why.>

- <skill-name> — for: `<task ref>` — why: `<reason>`
- ...

## 8. ADR (the documentation output)

<This repo documents non-trivial decisions with an ADR under docs/adr/ instead of spec
files. If a trigger applies (see /plan-draft phase 9), set frontmatter `adr: required` and draft
the decision body here; /plan-execute commits it as docs/adr/NNNN-<slug>.md and writes the number
back to frontmatter. Otherwise write "No ADR needed" + one line of why.>

## Execution log

<Filled in by /plan-execute: files changed, task/verify results, divergences and how they were resolved.>
