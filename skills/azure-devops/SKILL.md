---
name: azure-devops
description: Use when driving an Azure DevOps remote through the `az` CLI — "make a PR", "open a PR", "update the PR", "link work item <id>", or any `az repos`/`az devops`/`az boards` command. Covers flag-parity gotchas, work-item linking and field formats, and the commit-split/PR create-update workflow. Single source of truth for az CLI gotchas; self-updates when a command fails (see Self-healing below). The `code-review-azure` skill builds on this one for reviewing PRs.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git restore:*), Bash(git reset:*), Bash(git commit:*), Bash(git log:*), Bash(git push:*), Bash(git fetch:*), Bash(git rev-parse:*), Bash(az repos pr create:*), Bash(az repos pr update:*), Bash(az repos pr list:*), Bash(az repos pr show:*), Bash(az repos pr work-item:*), Bash(az boards work-item show:*), Bash(az boards work-item update:*), Bash(az account get-access-token:*)
---

# Azure DevOps CLI

This skill is the CLI-usage reference (gotchas, self-healing) plus the PR create/update workflow.
For reviewing an existing PR, use the **`code-review-azure`** skill — it depends on the gotchas
here rather than restating them. Every gotcha below exists because a real run got it wrong — do
not skip them, and do not re-discover them from scratch.

## Personalize before first use

This skill ships with placeholders. `<org>`, `<project>`, and `<repo>` below are never literals —
resolve them once and the rest of the skill works without passing flags on every call.

**Preferred: set CLI defaults**, so `--org`/`--project` can be omitted entirely.

```bash
az devops configure -d organization=https://dev.azure.com/<org>/ project=<project>
az devops configure --list                       # verify what is set
```

If defaults are not set, pass `--org https://dev.azure.com/<org>/` explicitly, or rely on
`--detect true` to read them from the git remote.

**Getting the org segment right matters more than it looks.** The URL segment is the org's
*canonical* name, which is often not the short name people say out loud (`ContosoEngineering`,
not `contoso`). A wrong segment returns a misleading `TF400813 "not authorized"` rather than a
404, so it reads as a permissions problem and sends you debugging the wrong thing. Recover the
canonical segment from any work item:

```bash
az boards work-item show --id <id> --query url      # → https://dev.azure.com/<org>/...
```

**Team conventions are yours to fill in.** Two things below are per-team, not per-tool, and are
written here as options rather than rules: the work-item commit trailer (§ Work-item linking) and
whether your project caps PR descriptions differently. Record your team's actual choice by editing
this file — the self-healing protocol applies to conventions too.

## Self-healing protocol

**This skill updates itself.** Whenever an `az`/`git` command in this skill's scope fails and you
figure out the fix:

1. Fix the command and continue the task — don't stop to edit the skill first.
2. Before ending the turn, add the gotcha to **§ CLI reference & gotchas** below: the exact
   failing command/flag, the correct form, one line on why, dated. Use `Edit` on this file
   directly — this is the "fix it in the source file now" rule, not a note-to-self.
3. If the new finding contradicts or refines an existing bullet, correct that bullet in place
   instead of appending a duplicate.
4. If a whole subcommand's flag surface turns out to differ from what's documented, add a line
   pointing at `az <command> --help` rather than trying to enumerate every flag.

Do this for genuine CLI/API discoveries (wrong flag, wrong org format, unexpected response shape).
Don't log one-off transient failures (network blip, expired token) — only things a future run of
this skill would hit again.

## CLI reference & gotchas (grows via self-healing)

### Flag support is narrower on some subcommands than others — never assume parity

- `az repos pr create` accepts `--project`, `--repository`, `--work-items`.
- `az repos pr update` does **not** accept `--project`, `--repository`, or `--work-items`. A PR
  `--id` is unique org-wide, so project/repo aren't needed to address it. Valid flags: `--id`,
  `--org`, `--title`, `--description`, `--draft`, `--status`, `--auto-complete`,
  `--bypass-policy`, `--bypass-policy-reason`, `--delete-source-branch`,
  `--merge-commit-message`, `--squash`, `--transition-work-items`.
- `az repos pr show` also does **not** accept `--project` (learned 2026-07-21:
  `unrecognized arguments: --project <project>`). Same reason as `pr update` — `--id` alone (plus
  `--org` if not configured as default) is enough.
- Work items are **not** settable via `pr update` at all — link them separately:
  ```bash
  az repos pr work-item add --org <org> --id <pr-id> --work-items <id> [<id2> ...]
  az repos pr work-item list --org <org> --id <pr-id> --output table   # verify
  ```
  On **`pr create`** `--work-items <id>` does work up-front (confirmed 2026-07-27) — the separate
  call is only needed for an already-existing PR.
- `az repos pr create --draft true` creates the PR in draft (`isDraft: true`, `status: active`).
- When unsure whether a flag carries over between subcommands, run `az <subcommand> --help`
  rather than guessing from a sibling command's surface.

### Org/project/work-item field gotchas

- The org URL segment is the canonical org name, not the short name people say — a wrong segment
  returns `TF400813 "not authorized"` instead of a 404. See § Personalize for how to recover it.
- Work item multiline fields (description, repro steps, acceptance criteria) default to **HTML**
  — markdown pasted in shows literal `**`/`-`/backticks. `az boards work-item create/update
  --description` cannot change that. Write the field as markdown **and** flip its format via a
  REST JSON-patch (`az devops invoke` is flaky for PATCH; use `curl` + a bearer token):
  ```bash
  TOKEN=$(az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798 --query accessToken -o tsv)
  curl -s -X PATCH -H "Authorization: Bearer $TOKEN" \
    -H "Content-Type: application/json-patch+json" \
    --data-binary @patch.json \
    "https://dev.azure.com/<org>/<project>/_apis/wit/workitems/<id>?api-version=7.2-preview.3"
  ```
  `patch.json`:
  ```json
  [
    {"op": "add", "path": "/fields/System.Description", "value": "<markdown>"},
    {"op": "add", "path": "/multilineFieldsFormat/System.Description", "value": "Markdown"}
  ]
  ```
  Same pattern for other multiline fields (e.g. `/multilineFieldsFormat/Microsoft.VSTS.TCM.ReproSteps`).
  PR **comments** (`pullRequestThreads` `content`) render markdown natively — this HTML gotcha is
  work-items only.
- **`--query` on `az boards work-item show` chokes on dotted field names** (learned 2026-07-27):
  `--query "{title:fields.'System.Title'}"` fails with `argument --query: invalid jmespath_type
  value` — jmespath needs the identifier double-quoted (`fields."System.Title"`), which then
  fights shell quoting. Simplest reliable form is to skip `--query` and parse the JSON:
  ```bash
  az boards work-item show --id <id> --org <org> -o json | python3 -c "import sys,json; f=json.load(sys.stdin)['fields']; print(f['System.Title'], f['System.State'])"
  ```
- `az repos pr show` returns **`commits: null`** — it does not embed the commit list. Get commits
  from git (`git log <target>..<source>`) or the REST `pullRequests/<id>/commits` resource.
- `mergeStatus: succeeded` on a PR does **not** mean it was merged/completed — it only means the
  source branch merges cleanly (no conflicts) into target. The only fields that mean a PR is done
  are `status` (`completed`/`abandoned`) and a non-null `closedDate`.
- Azure CLI has no `pr diff` and no `pr comment` command:
  - **Diff** comes from git (`git diff target...source`) or the REST `pullRequestIterations`/`changes` resource.
  - **Comments/threads** are posted via `az devops invoke` against the `git` area,
    `pullRequestThreads` resource (see the `code-review-azure` skill for the full posting flow).
- **PR description is capped at 4000 characters** (learned 2026-08-03):
  `az repos pr create --description "$(cat body.md)"` fails with
  `ERROR: Invalid argument value. Parameter name: A description for a pull request must not be
  longer than 4000 characters.` — nothing is truncated, the whole call is rejected. Check
  `wc -c body.md` first, and move overflow (extra findings, long tables, drift lists) into a PR
  comment thread — `pullRequestThreads` has no such limit and renders markdown natively.
- `az repos pr create` fails hard with `TF401179: An active pull request for the source and
  target branch already exists` if one is already open for that exact branch pair — check first
  with `az repos pr list --status active --source-branch <branch>` before creating.

### Work-item linking convention in commit messages

**Per-team setting — confirm it once, then record it here.** Two forms are in common use and they
are not interchangeable:

```
Related work items: #<id>      # plain trailer: human-readable, no Boards automation
AB#<id>                        # Azure Boards smart link: auto-links the commit to the work item
```

Put whichever your team uses on its own line, typically last in the commit body/footer. Never
substitute one for the other on the assumption they are equivalent — the smart link drives Boards
automation that the plain trailer does not.

<!-- PERSONALIZE: replace this block with your team's actual choice once confirmed, and delete
     the alternative. Ask a human rather than inferring it from commit history: a repo can contain
     both forms from different eras. -->

**This team uses:** _(unset — ask before writing your first work-item trailer)_

## PR create/update workflow

Split a changeset into logical commits, push, and create-or-update the PR against the repo's
Azure DevOps remote.

### 1. Before splitting a changeset into multiple commits

**Never trust that the index is empty before you start.** A prior session (or an auto-sync hook)
may have already staged files. `git add <specific files>` only *adds* those files to whatever is
already staged — it does not isolate them.

- Run `git status --porcelain` first. If anything shows as staged (`M `, `A `, first column
  non-blank) before you've added anything, unstage everything first: `git restore --staged .`
- After each targeted `git add` for one commit group, verify the staged set matches your intended
  file list before committing:
  ```bash
  git status --porcelain=v1 | grep -c '^[AM]'   # count must equal your intended file count
  ```
  A mismatch means unrelated files rode along — find them with `git diff --cached --stat` and
  `git restore --staged` the ones that don't belong, before committing.
- To regroup commits already made (wrong split discovered after the fact) and nothing has been
  pushed yet: `git reset --soft <parent-of-first-wrong-commit>` re-stages everything as one blob,
  then `git restore --staged .` to unstage, then redo the per-group `git add` + `commit` loop.

### 2. Before assuming a branch is "local only"

Branches on this remote can already be pushed even if you never ran `git push` yourself (e.g. an
auto-sync hook, or a previous session). **Always check before rewriting history:**

```bash
git fetch origin <branch>
git rev-parse HEAD origin/<branch>      # same SHA = already pushed, rewrite needs force-push
```

If it's already on origin, rewriting commits (e.g. to add a work-item trailer) requires
`git push --force-with-lease` afterward. This is a force-push to a remote branch — **ask the user
to confirm before doing it**, even if they asked for the content change; they may not know the
branch is already remote.

### Typical flow

1. `git status --porcelain` → confirm/clear the index (§1).
2. Group changed files by concern, `git add` + `git commit` each group, verifying the staged count
   every time (§1). Append `Related work items: #<id>` if one was given.
3. `git fetch origin <branch>` and compare SHAs (§2) before any push if commits were rewritten.
4. `git push` (or `--force-with-lease` with confirmation if rewriting an already-pushed branch).
5. `az repos pr list --source-branch <branch> --status active` — update the existing PR if found
   (see the "check first" gotcha above), otherwise `az repos pr create`.
6. `az repos pr work-item add` to link the work item — `pr create --work-items` works up-front,
   but `pr update` cannot set it, so an existing PR always needs this separate call.
