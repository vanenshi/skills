---
name: conventional-commits
description: >-
  Formats git commit messages per the qoomon conventional commits specification
  (types, scopes, breaking changes, body, footer). Apply automatically when
  drafting commit messages, running git commit, reviewing staged changes, or
  when the user mentions commits, commit messages, or conventional commits.
---

When drafting or running git commits, format every message per the qoomon
conventional commits convention (compatible with
[conventionalcommits.org](https://www.conventionalcommits.org/)). Do not invent
types or formatting rules outside this skill.

## Workflow

1. Inspect the change — `git diff`, staged files, or the user's description.
2. Choose **type** and optional **scope**.
3. Write the **subject** (imperative, lowercase, no trailing period).
4. Add a **body** when motivation or prior behavior needs explanation.
5. Add a **footer** for issue references and breaking-change details.
6. Commit via HEREDOC (see below).

## Message structure

```
<type>(<optional scope>): <description>

<optional body>

<optional footer>
```

**Special cases** — use these as-is; do not conventionalize merge/revert messages:

| Case           | Format                                    |
| -------------- | ----------------------------------------- |
| Initial commit | `chore: init`                             |
| Merge          | `Merge branch '<branch name>'`            |
| Revert         | `Revert "<reverted commit subject line>"` |

## Types

Pick exactly one:

| Type       | Use when                                           |
| ---------- | -------------------------------------------------- |
| `feat`     | Add, adjust, or remove an API or UI feature        |
| `fix`      | Fix a bug in a prior `feat`                        |
| `refactor` | Restructure code without changing API/UI behavior  |
| `perf`     | Improve performance (special case of refactor)     |
| `style`    | Formatting or whitespace only — no behavior change |
| `test`     | Add or correct tests                               |
| `docs`     | Documentation only                                 |
| `build`    | Build tools, dependencies, project version         |
| `ops`      | IaC, CI/CD, deployment, monitoring, backups        |
| `chore`    | Misc maintenance (`.gitignore`, init, etc.)        |

## Scope

Optional. Use project-specific context (module, area, component).
**Do not** use issue identifiers as scopes.

## Subject

- Mandatory.
- Imperative present tense — think "This commit will…" (`add`, not `added`).
- Lowercase first letter.
- No trailing period.
- Breaking change: `!` before `:` — e.g. `feat(api)!: remove status endpoint`.

## Body

Optional. Explain **why** and contrast with previous behavior. Imperative present tense.

## Footer

Optional unless the commit is breaking.

- Issue refs: `Closes #123`, `Fixes JIRA-456`
- Breaking changes **must** start with `BREAKING CHANGE:`
  - One line: space after the colon
  - Multi-line: blank line after `BREAKING CHANGE:`

If the subject with `!` is not sufficiently informative, describe the break in the footer.

## Commit command

Always pass the message via HEREDOC:

```bash
git commit -m "$(cat <<'EOF'
<type>(<optional scope>): <description>

<optional body>

<optional footer>
EOF
)"
```

## Examples

**Feature:**

```
feat: add email notifications on new direct messages
```

**Feature with scope:**

```
feat(shopping cart): add the amazing button
```

**Bug fix:**

```
fix(shopping-cart): prevent ordering an empty shopping cart
```

**Fix with body:**

```
fix: add missing parameter to service call

The error occurred due to a missing userId in the payload.
```

**Breaking change:**

```
feat!: remove ticket list endpoint

refers to JIRA-1337

BREAKING CHANGE: ticket endpoints no longer support listing all entities.
```

**Other types:**

```
perf: decrease memory footprint for unique visitors using HyperLogLog
```

```
build: update dependencies
```

```
refactor: implement fibonacci calculation as recursion
```

## Versioning

When inferring semver from commits:

- Breaking changes → **major**
- `feat` or `fix` → **minor**
- Everything else → **patch**
