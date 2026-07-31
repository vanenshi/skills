# Writing an ADR

An ADR has exactly one job: **stop an LLM agent from writing code that breaks
an invariant it could not have inferred from the codebase.**

That is the whole test, and it is the only test. Not "was this a real
decision." Not "will I want to remember why." Assume the author remembers their
own reasoning; prose no agent acts on is prose that costs context on every read.

## Before writing one: four questions, in order

Answer all four. A "yes" to any of the first three means **no ADR** — the rule
belongs somewhere better, and a second copy is drift waiting to happen.

1. **Is it already a written code rule?** Check the project's convention docs
   (in this repo, `docs/conventions/README.md`'s routing table, by ID: `con-`,
   `str-`, `data-`, `cmp-`, `sty-`, `i18n-`, `ajt-`, `log-`). If a rule covers
   it, correct or extend that rule in place and keep its ID. Conventions outrank
   ADRs in review — putting the rule in an ADR instead makes the weaker copy the
   one people read.
2. **Would a machine catch the violation?** A schema column, a CHECK
   constraint, a dependency-cruiser rule, a branded type, the type-checker. If
   so, the enforcement IS the record. Do not narrate it.
3. **Is it a comment's job?** A single call site, a single file, a "why this
   looks odd" — that goes at the code, where it cannot drift out of sight.
4. **Would an agent plausibly violate it while writing ordinary code?** If you
   cannot name the wrong code an agent would write, there is no invariant here,
   only a preference.

Only a rule that survives all four gets a file.

## What actually earns an ADR

The ADRs worth keeping almost always encode **an absence or a deliberate
contradiction** — something no amount of reading the code reveals, because the
evidence is a thing that is *not there*. Real examples that survived a hard
prune:

- No destructive procedure ever accepts a password in its input;
  re-authentication is a separate call.
- AI `cost_irr` never enters a company-facing payload.
- Sends are at-most-once, `failed` is terminal, there is no attempt counter —
  deliberately contradicting the convention that says jobs must be idempotent
  and safe to retry.
- Persian message bodies live in the template registry, not the i18n catalog —
  a deliberate exception to the locale rule.
- A foreign key is recorded but never read to resolve the live set.

Hunt for that shape. An agent "helpfully fixing" one of these is the failure
mode the file exists to prevent — so name the wrong fix explicitly in the
bullet.

## Explicit non-triggers

None of these earns an ADR on its own. Each reliably produces a file that is
worthless within a month:

- Adding or removing a dependency.
- Choosing library A over library B, however carefully reasoned.
- A new module, service, or slice boundary that follows the existing pattern.
- A schema or contract change whose rules are visible in the schema.
- "This was a real decision and took me a while." Effort spent is not an
  invariant.

## Format — exactly this, nothing more

    # NNNN. <Title stated as the assertion, not the topic>

    Status: accepted · Date: YYYY-MM-DD

    ## Invariants

    - **<the rule, imperative>** — the wrong code an agent would write, and
      where the rule is enforced or visible: `file:line`, a CHECK constraint,
      a convention rule ID.

    ## Where it lives

    - `path/to/file.ts` — one line on what it owns.

    ## Known gaps

    - Only if real. Omit the section otherwise.

**No `## Context`. No `## Alternatives considered`. No `## Consequences`. No
Deciders.** Those three sections are typically 80% of a bloated ADR and none of
it changes what an agent writes. Under 45 lines total; if it is longer, most of
it is not an invariant.

Titles are assertions: "Stages are data, Stage Groups are code", not "Pipeline
stage storage". An agent skimming the directory listing should learn one rule
per line.

Every `file:line` is verified by reading the file at write time. A wrong pointer
is worse than no pointer — it sends the next session to the wrong place with
confidence.

## Numbering

Read `ls docs/adr/` and take the next free number — do not infer it from the
highest number you remember. Add the new ADR's one-clause summary wherever the
project indexes them (in this repo, `AGENTS.md` → Key docs) in the same change.

## Maintenance — ADRs are live, not archival

- **Keep them current.** A stale `file:line`, or a claim the code has outgrown,
  is a bug in the ADR and gets fixed like any other bug. There is no
  "point-in-time record" excuse — nothing here is history.
- **When the rule moves into a convention rule ID, delete the ADR.** Do not
  supersede it, do not leave a tombstone. The convention is now the single copy.
- **When a decision is reversed, delete the file.** A superseded ADR left in
  place is an agent reading a world that no longer exists.
- **Renaming or renumbering means fixing every live citation** in `src/`,
  `tests/`, and `docs/` in the same change, plus recording the old→new map in
  `docs/adr/README.md` for anything outside the working tree (commit messages,
  archived plans, an earlier session's context).
