---
name: azure-devops
description: Use when driving an Azure DevOps remote — "make a PR", "open a PR", "update the PR", "link work item <id>", work-item reads/writes, PR threads. MCP-first — use the `mcp__ado__*` tools when the ado MCP server is connected; the `az` CLI is the fallback only. Covers MCP tool selection, markdown/format handling, the 4000-char PR description cap, work-item linking, and the commit-split/PR create-update workflow. Self-updates when a call fails (see Self-healing below). The `code-review-azure` skill builds on this one for reviewing PRs.
allowed-tools: Bash(git status:*), Bash(git diff:*), Bash(git add:*), Bash(git restore:*), Bash(git reset:*), Bash(git commit:*), Bash(git log:*), Bash(git push:*), Bash(git fetch:*), Bash(git rev-parse:*), Bash(az repos pr create:*), Bash(az repos pr update:*), Bash(az repos pr list:*), Bash(az repos pr show:*), Bash(az repos pr work-item:*), Bash(az boards work-item show:*), Bash(az boards work-item update:*), Bash(az account get-access-token:*)
---

# Azure DevOps (MCP-first)

This skill is the Azure DevOps usage reference (tool selection, gotchas, self-healing) plus the
PR create/update workflow. For reviewing an existing PR, use the **`code-review-azure`** skill —
it depends on the gotchas here rather than restating them. Every gotcha below exists because a
real run got it wrong — do not skip them, and do not re-discover them from scratch.

## Tool preference: MCP first, az CLI fallback

**When the `ado` MCP server is connected (`mcp__ado__*` tools available, possibly deferred —
load via ToolSearch), use it for everything it covers.** It talks to the same REST API the CLI
wraps, but with typed schemas that eliminate whole classes of CLI gotchas (flag parity, jmespath
quoting, the HTML-vs-markdown PATCH dance). Drop to `az` only when MCP is absent from the session
or lacks the operation.

| Task | MCP tool (preferred) | az fallback |
|---|---|---|
| List projects | `mcp__ado__core_list_projects` | `az devops project list` |
| My work items | `mcp__ado__wit_work_item` `action: my` | `az boards query` |
| Read work item(s) | `mcp__ado__wit_work_item` `action: get`/`get_batch` | `az boards work-item show` |
| Ad-hoc WIQL | `mcp__ado__wit_query` `action: wiql` | `az boards query --wiql` |
| Create/update work item | `mcp__ado__wit_work_item_write` | `az boards work-item create/update` |
| Work-item comment | `mcp__ado__wit_work_item_comment_write` | *(REST only)* |
| Link work item ↔ PR/item | `mcp__ado__wit_work_item_link_write` | `az repos pr work-item add` |
| Create/update PR | `mcp__ado__repo_pull_request_write` | `az repos pr create/update` |
| Read PR | `mcp__ado__repo_pull_request` | `az repos pr show/list` |
| PR comment threads | `mcp__ado__repo_pull_request_thread_write` / `..._thread` | `az devops invoke` (no native command) |
| Search work items / code / wiki | `mcp__ado__search_workitem` / `search_code` / `search_wiki` | *(REST only)* |

**Not covered by MCP (always local git):** diffs (`git diff target...source`), commit lists
(`git log target..source`), all staging/commit/push mechanics. The MCP server has no git tools.

**MCP niceties that replace old CLI workarounds:**

- **Markdown in work-item fields is a first-class parameter.** `wit_work_item_write` takes
  `format: "Markdown"` per field (`fields[].format` on create, `batchUpdates[].format` on
  update_batch); `wit_work_item_comment_write` defaults to Markdown. No REST JSON-patch, no
  bearer-token curl. *(The old curl + `multilineFieldsFormat` PATCH recipe lived here until
  2026-08-19 — struck as obsolete; see git history of this file if you're ever stuck on raw REST.)*
- **PR thread status is settable at creation.** `repo_pull_request_thread_write` `action: create`
  takes `status` — but it **defaults to `Active`**, which shows as an unresolved comment and gates
  PR completion. For an informational note (a translation, a summary, overflow from the 4000-char
  description) pass `status: "Closed"` so it never asks a human to resolve something that was
  never a question. Use `Active` **only** when asking the author for a change. Retire an earlier
  thread with `action: update_status`.
- **The 4000-char PR description cap is enforced in the tool schema** (`description` has
  `maxLength: 4000`), so an oversized body fails client-side instead of as a server rejection.
  The caveat that survives: **the cap counts characters, not bytes — never check with `wc -c`.**
  Non-ASCII bodies (Turkish `ı ş ğ`, `→`, `·`) are 2–3 bytes each, so `wc -c` overcounts badly.
  Check with `python3 -c "print(len(open('body.md',encoding='utf-8').read()))"`. Move overflow
  (extra findings, long tables, drift lists) into a PR comment thread — threads have no such
  limit and render markdown natively.
- **`repositoryId` by name needs `project` too** — the `repo_*` write tools require `project`
  whenever `repositoryId` is a name rather than a GUID.
- Work-item `my`/read tools need `project` — without it the call fails with
  "Project selection cancelled" in non-interactive sessions. Resolve it once via
  `core_list_projects` and reuse.

## Self-healing protocol

**This skill updates itself.** Whenever an MCP/`az`/`git` call in this skill's scope fails and you
figure out the fix:

1. Fix the call and continue the task — don't stop to edit the skill first.
2. Before ending the turn, add the gotcha to the relevant section below: the exact failing
   call/flag/param, the correct form, one line on why, dated. Use `Edit` on this file directly —
   this is the "fix it in the source file now" rule, not a note-to-self.
3. If the new finding contradicts or refines an existing bullet, correct that bullet in place
   instead of appending a duplicate.
4. If a whole tool's parameter surface turns out to differ from what's documented, point at the
   tool's schema (ToolSearch `select:` re-fetches it) rather than enumerating every param.

Do this for genuine API/tool discoveries (wrong param, unexpected response shape). Don't log
one-off transient failures (network blip, expired token) — only things a future run would hit again.

## API-level gotchas (apply to MCP and CLI alike)

These are server behaviors, not CLI quirks — the MCP tools hit them too.

- `mergeStatus: succeeded` on a PR does **not** mean it was merged/completed — it only means the
  source branch merges cleanly (no conflicts) into target. The only fields that mean a PR is done
  are `status` (`completed`/`abandoned`) and a non-null `closedDate`.
- PR create fails hard with `TF401179: An active pull request for the source and target branch
  already exists` if one is already open for that exact branch pair — list active PRs for the
  source branch first, and update the existing one instead.
- A PR read does not embed its commit list — get commits from git (`git log <target>..<source>`).
- PR thread `content` renders markdown natively; work-item multiline fields need the explicit
  `format: "Markdown"` (see above) or they render as HTML with literal `**`/backticks.

## az CLI fallback gotchas (only when MCP is unavailable)

<details>
<summary>Kept for MCP-less sessions — flag parity, org segment, jmespath quoting</summary>

- **Set CLI defaults first** so `--org`/`--project` can be omitted:
  `az devops configure -d organization=https://dev.azure.com/<org>/ project=<project>`.
- **Org URL segment is the canonical org name**, often not the short name people say. A wrong
  segment returns a misleading `TF400813 "not authorized"`, not a 404. Recover it from any work
  item: `az boards work-item show --id <id> --query url`.
- Flag parity is NOT uniform across `az repos pr` subcommands:
  - `pr create` accepts `--project`, `--repository`, `--work-items`.
  - `pr update` and `pr show` do **not** accept `--project`/`--repository`/`--work-items` — a PR
    `--id` is unique org-wide (learned 2026-07-21).
  - Work items on an existing PR: `az repos pr work-item add --id <pr-id> --work-items <id> ...`.
  - When unsure, `az <subcommand> --help` — never guess from a sibling command's surface.
- `--query` on `az boards work-item show` chokes on dotted field names (learned 2026-07-27) —
  skip `--query`, pipe `-o json` to python and read `['fields']`.
- Work-item markdown via CLI requires the REST JSON-patch with `multilineFieldsFormat`
  (`az devops invoke` is flaky for PATCH; use `curl` + bearer token from
  `az account get-access-token --resource 499b84ac-1321-427f-aa17-267ca6975798`). With MCP this
  is obsolete — use `format: "Markdown"`.
- Azure CLI has no `pr diff` and no `pr comment` command — diff from git, threads via
  `az devops invoke` against the `git` area / `pullRequestThreads` resource.
- PR description >4000 chars: the whole `pr create`/`pr update` call is rejected, nothing is
  truncated (learned 2026-08-03). Gate on the python char count before calling, and verify the
  remote took it afterward (Azure stores CRLF — normalize before comparing).

</details>

## Work-item linking convention in commit messages

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
   every time (§1). Append the team's work-item trailer if an id was given.
3. `git fetch origin <branch>` and compare SHAs (§2) before any push if commits were rewritten.
4. `git push` (or `--force-with-lease` with confirmation if rewriting an already-pushed branch).
5. List active PRs for the source branch (`mcp__ado__repo_pull_request` / `az repos pr list
   --source-branch <branch> --status active`) — update the existing PR if found (TF401179
   gotcha above), otherwise create (`mcp__ado__repo_pull_request_write` `action: create`; pass
   `workItems` up-front).
6. For an already-existing PR, link the work item separately —
   `mcp__ado__wit_work_item_link_write` (or `az repos pr work-item add`); PR update cannot set
   work items in either surface.
