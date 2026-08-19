---
name: code-review-azure
description: Code review an Azure DevOps pull request. Use when the user asks to review an Azure Repos / Azure DevOps PR, or says "code-review-azure". Also use when the user wants to understand or decide on an Azure DevOps PR — "what changed in PR 482", "summarize this PR", "what did they decide", "what should I ask the developer". MCP-first — uses the `mcp__ado__*` tools when the ado MCP server is connected; falls back to `az repos`/`az devops invoke` otherwise. Mirrors the GitHub review flow.
allowed-tools: Bash(az repos pr show:*), Bash(az repos pr list:*), Bash(az repos pr policy list:*), Bash(az repos pr reviewer list:*), Bash(az devops invoke:*), Bash(az devops configure:*), Bash(git fetch:*), Bash(git diff:*), Bash(git log:*), Bash(git rev-parse:*), Bash(git show:*), Bash(python3:*)
---

# Code review (Azure DevOps)

> Tool preference (MCP first, az fallback), org/project conventions, API-level gotchas, and the
> self-healing protocol live in the **`azure-devops`** skill — read it first. This skill is the
> review workflow that sits on top of it; it points back there for ADO mechanics instead of
> restating them, so gotcha fixes only ever need one home.

Provide a code review for the given Azure DevOps pull request.

**Surface selection:** when the ado MCP server is connected (`mcp__ado__*` tools, possibly
deferred — load `repo_pull_request`, `repo_pull_request_thread`, `repo_pull_request_thread_write`
via ToolSearch in one call), use it for all PR metadata, thread reads, and thread posting. The
`az` CLI paths below are kept as the fallback for MCP-less sessions, and for the one case MCP
cannot do (left-side/deleted-line anchors — see step 9). Diffs and commit lists always come from
local git in either surface.

## Usage

```
/code-review-azure <pr-id> [--brief] [--comment] [--org <url>] [--project <name>] [--repository <name>]
```

`<org>`, `<project>`, and `<repo>` are placeholders throughout this skill, never literals. For
MCP, `project` + `repositoryId` are call parameters (project required when repositoryId is a
name). For the az fallback, set them once via `az devops configure` — see the `azure-devops`
skill. Inline `<!-- PERSONALIZE: … -->` comments below mark the places where recording your
own repo's specifics turns a multi-step investigation into a single grep.

| Arg | Effect |
| --- | --- |
| `<pr-id>` | **Required.** Numeric Azure DevOps PR id (no `#`). |
| `--comment` | Post findings back to the PR as inline threads (and a summary if clean). Without it, findings only print to the terminal — nothing is posted. |
| `--brief` | Stop after the change brief (step 3). Orientation only: what changed, what was decided, what to ask. Skips the review agents entirely. Implies no `--comment`. |
| `--org <url>` | Azure DevOps org URL. Only meaningful for the az fallback (MCP connection fixes the org). |
| `--project <name>` | Project name or id. |
| `--repository <name>` | Repo name or id. |

Examples:
- `/code-review-azure 482` — review PR 482, print the change brief then the findings.
- `/code-review-azure 482 --brief` — change brief only, no review agents.
- `/code-review-azure 482 --comment` — review and post inline threads.

**Prereqs (assume already true; do NOT test tools):**
- MCP path: the ado MCP server is connected and authenticated.
- az fallback: `az` with the `azure-devops` extension, authenticated; org/project configured,
  passed, or `--detect true` from the git remote.

**Agent assumptions (applies to all agents and subagents):**
- All tools are functional and will work without error. Do not test tools or make exploratory calls. Make sure this is clear to every subagent that is launched.
- Only call a tool if it is required to complete the task. Every tool call should have a clear purpose.
- **Review is READ-ONLY. Say so in every subagent prompt.** No subagent may modify repo files,
  and none may run a package-manager install (`npm`/`pnpm`/`yarn install`) — learned 2026-07-29,
  a reviewer set up an "isolated" typecheck worktree, its `npm install` resolved through
  a symlink and **pruned the real repo's `node_modules/@types`**, which it then had to relink from
  the pnpm store. Reading files, `git show`, `git diff`, `git log`, and read-only greps are the
  tools for this job; a typecheck is a nice-to-have, never worth writing to the user's tree.
  If a finding genuinely can't be validated without building, report it as unvalidated instead.
- After the review, `git status --porcelain` the repo and report any unexpected working-tree
  change a subagent left behind. Never silently revert it — it may be the user's in-progress work.

Create a todo list before starting, then follow these steps precisely:

1. Launch a haiku agent to check if any of the following are true. PR metadata:
   `mcp__ado__repo_pull_request` `action: get` with `pullRequestId` (fallback:
   `az repos pr show --id <id>` — no `--project`/`--repository`, see the `azure-devops` skill's
   flag-parity gotcha).
   - The PR is abandoned/completed (`status` != `active`).
   - The PR is a draft (`isDraft: true`).
   - **Gotcha (learned 2026-07-21, from a false-stop):** `mergeStatus: succeeded`
     does NOT mean the PR was merged/completed — it only means the source
     branch merges cleanly (no conflicts) into target. Do not infer
     "already merged" from it. The only fields that mean the PR is done are
     `status` (`completed`/`abandoned`) and a non-null `closedDate`. Tell the
     subagent explicitly: judge completion by `status`/`closedDate` only,
     never by `mergeStatus`.
   - The PR does not need code review (automated PR, trivial obviously-correct change).
   - Claude has already commented on this PR. Fetch existing threads:
     `mcp__ado__repo_pull_request_thread` (list threads for the PR). Fallback:
     ```bash
     az devops invoke --area git --resource pullRequestThreads \
       --route-parameters project="<project>" repositoryId="<repo>" pullRequestId=<id> \
       --api-version 7.1 -o json --http-method GET
     ```
     If any thread comment was left by Claude, stop.

   If any condition is true, stop and do not proceed. (Still review Claude-generated PRs.)

2. Launch a haiku agent to return a list of file paths (not contents) for all relevant CLAUDE.md files:
   - The root CLAUDE.md, if it exists.
   - Any CLAUDE.md in directories containing files modified by the PR.
   - Any rule/convention files those CLAUDE.md files route to (a common layout is `AGENTS.md` →
     `docs/conventions/*`). Those routed files ARE the rules — a reviewer citing a rule id
     must have read them.
     <!-- PERSONALIZE: name your repo's actual routing target here once you know it. -->
   To know which files changed, get the diff (step 3).

   **Read rule docs from the PR BRANCH, never from the working tree** (learned 2026-07-29:
   3 of 13 candidate findings were false positives citing rule ids that existed only on the
   local checked-out feature branch, not on the PR's base or source branch — the convention
   file defining them did not exist on either). The local
   checkout is usually a *different, unmerged* branch than the PR under review, so its
   conventions may be ahead of, behind, or divergent from what governs this PR. Always:
   ```bash
   git show <sourceBranch>:docs/conventions/<file>.md     # rules in force for this PR
   git diff <targetBranch> <sourceBranch> -- docs/         # rules the PR itself amends
   ```
   Tell every reviewing and validating subagent this explicitly. A rule that isn't on the PR's
   branch is not in force — citing it is wrong on the public record. If the PR amends a rule,
   judge the code against the **amended** version; a rule the PR legitimately changes is not a
   violation. Validators must confirm the cited rule id actually exists on the branch and quote
   it verbatim before confirming.

3. Get the PR metadata and diff, then build the change brief.

   **3a. Metadata and diff.**
   - `mcp__ado__repo_pull_request` `action: get` → title, description, `sourceRefName`,
     `targetRefName`, `lastMergeSourceCommit`, `lastMergeTargetCommit`. (Fallback:
     `az repos pr show --id <id> -o json`.)
   - Fetch the branches and diff locally — the PR read does not embed commits or a diff in
     either surface. Save it to a file; later steps read it more than once:
     ```bash
     git fetch origin
     # refs look like refs/heads/<branch>; strip the prefix
     git diff <targetBranch>...<sourceBranch> > /tmp/pr-<id>.diff
     git log --oneline <targetBranch>..<sourceBranch>
     ```
     If local branches aren't available, fall back to the REST iterations/changes:
     ```bash
     az devops invoke --area git --resource pullRequestIterations \
       --route-parameters project="<project>" repositoryId="<repo>" pullRequestId=<id> \
       --api-version 7.1 -o json --http-method GET
     ```

   **3b. Condense the diff.** If the diff exceeds ~400 changed rows, invoke the
   **`abridge-diff`** skill on `/tmp/pr-<id>.diff` first and brief from the reading
   diff instead of the raw one. Below that, brief from the raw diff — abridging a
   small diff costs more than it saves. Commit subjects (`git log --oneline`) are
   evidence too: they often name the decision the diff only implies.

   **3c. Launch a sonnet agent to write the change brief.** Give it the PR title,
   description, commit subjects, and the (possibly abridged) diff. Its entire output
   is the brief below — no preamble, no closing offer of help.

   The brief has exactly four parts, in this order:

   ```markdown
   ## Change brief — PR <id>: <title>

   <One to three sentences: what this PR does, in the reviewer's terms.>

   ### Major changes
   - **<Intent, not filename>** — <what it does now that it did not before>. `path/File.cs:41`
   <Grouped by intent, so one entry may span several files. Max 7; if more were
   found, add a final line saying how many were left out.>

   ### Decisions
   | Decision | Where | Evidence it was deliberate | Alternative not taken |
   |---|---|---|---|
   | <the choice made> | `path/File.cs:41` | <quoted comment, commit subject, PR description line, or the code shape itself> | <the option a reviewer would expect instead> |

   ### Questions for the author
   - <Question whose answer would change whether you approve.> — `path/File.cs:41`
   ```

   **What counts as a decision** (the reviewer already knows what changed; this
   section is why it changed *this way*):
   - A chosen approach where another was plainly available — new abstraction vs.
     extending an existing one, new dependency vs. stdlib, new table/column vs.
     reusing one, sync vs. async, client vs. server.
   - A changed contract — API shape, DB schema, public signature, config default,
     migration, anything an external caller can observe.
   - A scope boundary — what was deliberately left alone, TODOs added, code behind a
     feature flag, a workaround with a stated ceiling.
   - A behavioral default — timeout, retry count, page size, cache TTL, permission
     gate, ordering.

   **Evidence rule: every Decisions row cites its evidence — a `file:line`, a quoted
   comment, a commit subject, or a line of the PR description.** A choice you can see
   but whose rationale you cannot evidence is not a decision, it is a question: move
   it to Questions and phrase it as one. Inventing plausible rationale is the failure
   mode this section exists to avoid — an invented "why" that the author never
   intended sends the reviewer into a conversation about something that isn't there.

   **Questions** are only for things the diff genuinely does not answer *and* whose
   answer would change the approve/reject call. Max 5. Not style preferences, not
   "did you consider", not anything answerable by reading one more file — read the
   file instead.

   Print the brief to the terminal verbatim. **The brief is never posted to the PR**,
   with or without `--comment`: it describes the author's own work back to them, and
   `--comment` governs findings only.

   If `--brief` was passed, stop here.

4. Launch 4 agents in parallel to independently review the changes. Each returns a list of issues; each issue has a description and the reason it was flagged (e.g. "CLAUDE.md adherence", "bug"). Give each subagent the PR title and description for author intent.

   Agents 1 + 2: CLAUDE.md compliance sonnet agents — audit the diff for CLAUDE.md compliance in parallel. Only consider CLAUDE.md files that share a path with the file or its parents.

   Agent 3: Opus bug agent (parallel with agent 4) — scan for obvious bugs in the diff itself, no extra context. Flag only significant bugs; ignore nitpicks and likely false positives. Do not flag issues you cannot validate from the diff alone.

   Agent 4: Opus bug agent (parallel with agent 3) — look for problems in the introduced code: security issues, incorrect logic, etc. Only within the changed code.

   **CRITICAL: only HIGH SIGNAL issues.** Flag issues where:
   - The code will fail to compile or parse (syntax/type errors, missing imports, unresolved references).
   - The code will definitely produce wrong results regardless of inputs (clear logic errors).
   - Clear, unambiguous CLAUDE.md violations where you can quote the exact rule broken.

   Do NOT flag:
   - Code style or quality concerns.
   - Potential issues that depend on specific inputs or state.
   - Subjective suggestions or improvements.

   If you are not certain an issue is real, do not flag it. False positives erode trust.

5. For each issue from agents 3 and 4, launch parallel subagents to validate it (given PR title, description, issue description). The subagent confirms with high confidence the issue is real. For CLAUDE.md issues, validate the rule is scoped to that file and actually violated. Use Opus subagents for bugs/logic, sonnet for CLAUDE.md.

6. Filter out any issues not validated in step 5. This yields the high-signal list.

7. Write the final terminal output. Steps 4-6 run many agents, so by now the step-3
   brief has scrolled far out of view — the final message must stand alone. It has
   three parts, in this order:

   1. **The change brief from step 3, verbatim.** Re-emit it; do not summarize it
      again or replace it with a sentence saying it appeared earlier.
   2. **`### Findings`** — each validated issue with a brief description and its
      `file:line`. If none: "No issues found. Checked for bugs and CLAUDE.md
      compliance."
   3. **`### Where the review and the brief disagree`** — include this section only
      when a finding contradicts a Decisions row or answers a Question. One line
      each: which brief entry, and what the review found. Omit the heading entirely
      when nothing conflicts.

   A finding that turns a Question into a confirmed problem is the highest-value
   output this skill produces. Keep both: the question shows what to ask, the
   finding shows why it matters.

   If `--comment` was NOT provided, stop here. Do not post anything.

   If `--comment` IS provided and NO issues were found, post a summary thread (see below) and stop.

   If `--comment` IS provided and issues were found, continue to step 8.

8. Build the list of comments you plan to leave (for your own check; do not post it anywhere).

9. Post each issue as an inline PR thread.

   **MCP path (preferred):** one `mcp__ado__repo_pull_request_thread_write` call per issue —
   no JSON body files, no shell escaping:
   ```
   action: create
   repositoryId: <repo>          # project required too when this is a name, not a GUID
   project: <project>
   pullRequestId: <id>
   filePath: /src/Foo.cs         # repo-absolute, leading slash
   rightFileStartLine: 41
   rightFileEndLine: 42
   status: Active                # Active only when asking for a change; Closed for FYI-only
   content: <markdown comment>
   ```
   - **MCP limitation: only right-side (new file) anchors exist** — there are no
     `leftFile*` params. A comment on *deleted* lines needs the az fallback below with
     `leftFileStart`/`leftFileEnd`.
   - For a general (non-inline) summary comment, omit `filePath` and the line params.
   - `status` defaults to `Active`, which shows as unresolved and **gates PR completion** —
     see the `azure-devops` skill. Findings are `Active`; anything informational is `Closed`.

   **az fallback** (MCP absent, or left-side anchor needed): write a JSON body per thread and
   POST via `az devops invoke`:
   ```jsonc
   // thread.json — inline comment; use leftFileStart/leftFileEnd for deleted lines
   {
     "comments": [
       { "parentCommentId": 0, "content": "<markdown comment>", "commentType": 1 }
     ],
     "status": 1,                                  // 1 = active, 4 = closed
     "threadContext": {
       "filePath": "/src/Foo.cs",
       "rightFileStart": { "line": 41, "offset": 1 },
       "rightFileEnd":   { "line": 42, "offset": 1 }
     }
   }
   ```
   ```bash
   az devops invoke --area git --resource pullRequestThreads \
     --route-parameters project="<project>" repositoryId="<repo>" pullRequestId=<id> \
     --api-version 7.1 --http-method POST \
     --in-file thread.json --media-type application/json -o json
   ```
   Build the JSON bodies with a `python3` heredoc writing real files (`json.dump`) rather than
   hand-escaping markdown into shell strings — comment bodies contain backticks, `$`, and
   newlines that shell quoting mangles.

   **Both paths:**
   - For small self-contained fixes, include the suggested code in the markdown `content`
     (Azure DevOps has no committable-suggestion syntax like GitHub — put a fenced code block
     in the comment). For larger fixes, describe the fix in prose.
   - **Only ONE thread per unique issue. No duplicates.**
   - **Never retry a post on a parse/response failure — list the threads first.** Learned
     2026-08-03 on the az path: all six POSTs succeeded, but the wrapper that piped each
     response through an inline `python3 -c` inside `$(...)` mis-reported every one as `FAIL`.
     Retrying would have double-posted six threads on a public PR. Do not judge success by your
     own parsing of the post output; confirm with the read-only thread list
     (`mcp__ado__repo_pull_request_thread`, or the GET above) and compare `filePath` +
     `rightFileStart.line` against your planned set. Simplest robust loop: post each thread,
     ignore per-call output, then list once at the end and verify the thread count and anchors.

Use this list when evaluating issues in steps 4 and 5 (false positives, do NOT flag):
- Pre-existing issues.
- Something that looks like a bug but is actually correct.
- Pedantic nitpicks a senior engineer would not flag.
- Issues a linter will catch (do not run the linter to verify).
- General code quality / coverage / generic security concerns unless CLAUDE.md explicitly requires them.
- Issues mentioned in CLAUDE.md but explicitly silenced in the code (e.g. a lint-ignore comment).
- **"A Tailwind config change alters utility X app-wide" claims, in any repo carrying a legacy
  CSS layer.** Learned 2026-07-29. Where a project still loads a Bootstrap-derived (or other
  legacy) stylesheet whose utility rules carry `!important` — e.g.
  `.rounded { border-radius: 0.375rem !important }` — a new Tailwind `theme.extend` token can be
  **completely inert**. Tailwind emits utilities unlayered and non-important, and `!important`
  beats a normal declaration regardless of cascade-layer origin, so the legacy rule keeps
  winning. Before asserting any such finding: grep the legacy CSS for the exact class, and check
  the bare *side* variants (`rounded-t`, `rounded-l`, …) separately — those often have no
  `!important` twin and are the only live path to a real regression.
  <!-- PERSONALIZE: record your repo's legacy-layer entry points here (the stylesheet, what
       imports it, and any ADR documenting the isolation strategy) so this check is one grep
       instead of an investigation. -->
- **Repeated global styling tokens (e.g. `rounded-[30px]`) are usually pre-existing.**
  Same lesson. Count occurrences on the *base* branch before flagging an "arbitrary value
  should be a token" violation: `git grep -c '<token>' origin/main`. A refactor that
  carries a large house value forward while *reducing* the count by one is not
  introducing it.

Notes:
- **Sonnet subagents can fail with `API Error: 529 Overloaded` in bursts.** Learned
  2026-07-29, 4 consecutive failures in one run. Relaunch the agent on `opus` rather than
  retrying the same model — capacity differs per model — and keep the prompt identical.
- Use the MCP tools (or the `az` CLI fallback) for all Azure DevOps interaction. Do not use web fetch.
- **Formatting differs by surface** (learned 2026-07-10, a work item rendered
  raw markdown as plain text):
  - PR **thread** `content` renders **markdown** natively.
  - **Work item** multiline fields default to **HTML** — with MCP, pass
    `format: "Markdown"` (see the `azure-devops` skill); on the az path it needs the
    REST JSON-patch with `multilineFieldsFormat`.
- When linking to code in comment markdown, link to the file at the PR's source commit in the Azure DevOps web UI, e.g.
  `https://dev.azure.com/<org>/<project>/_git/<repo>/commit/<fullSha>?path=/src/Foo.cs&line=41&lineEnd=42&lineStartColumn=1&lineEndColumn=1`
  Use the full commit SHA (`lastMergeSourceCommit.commitId` from the PR metadata).

Summary comment format when `--comment` is set and no issues found (post as a thread with no file anchor; status `Closed` — it asks nothing of the author):

---

## Code review

No issues found. Checked for bugs and CLAUDE.md compliance.

---
