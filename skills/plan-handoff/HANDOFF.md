---
status: open # open -> planned (target /plan-draft consumed it) -> done (target plan executed)
date: YYYY-MM-DD
target: <repo that must act on this handoff, e.g. Match.Backend>
origin_plan: <absolute path to the plan that generated this handoff>
plan: # set by the target repo's /plan-draft: path to the plan drafted from this handoff
---

<!--
A handoff is a PLANNING artifact. It lives in the ORIGIN repo's .claude/handoffs/, next
to the plan that generated it, and is consumed by /plan-draft in the target repo (the
requester passes its absolute path there). It carries the cross-repo contract so the
receiving planner does not re-derive it. Keep it as skimmable as a plan: short bullets,
payload examples in fenced json, no prose walls.
-->

# Handoff: <one-line title>

## Context

<What the origin plan is building, why the target repo is involved. Reference the origin
plan path — do not restate its content.>

## Problem

<What is missing or wrong in the TARGET repo that blocks the origin plan.>

## Contract

<The API shape / event / schema / behavior the origin side depends on. Endpoints, DTOs,
error shapes — verbatim, with before/after payload examples if a contract changes. This
section is the whole point of the file: the receiving plan builds to exactly this.>

## Acceptance

- [ ] <how the origin side verifies this handoff is satisfied — test, endpoint probe,
  generated-client compile, manual check>
