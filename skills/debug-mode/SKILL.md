---
name: debug-mode
description: Cursor-style Debug Mode — hypothesis-driven debugging loop with runtime log instrumentation and human-in-the-loop reproduction. Use whenever a bug can't be diagnosed by reading code alone and someone must run the app to observe it — "debug mode", "debug this with me", "I can reproduce it but don't know why", "help me trace this", intermittent or race-condition bugs, works-on-web-but-not-device, crashes with no useful stack, or suspected bugs inside a node_modules dependency (JS or its native iOS/Android code). Instruments code with a marked JSONL append-helper writing runtime evidence straight to a session log file (the default sink in every environment — backend, web, RN JS, native Swift/Kotlin — with console/logcat/log-stream relays only as fallbacks), shows the user reproduction steps in a dialog with "Proceeded" / "It's fixed" choices, reads the captured logs after each run, refines the hypothesis, repeats — then strips every trace of instrumentation. Also covers upstream fixes via pnpm patch / patch-package.
---

# Debug Mode

An evidence loop: **hypothesize → instrument → user reproduces → read logs → fix or narrow → repeat**. Never guess blind; every iteration must produce runtime evidence. Modeled on Cursor's Debug Mode.

If the bug is obvious from reading the code, just fix it — don't ritualize the loop.

## 0. Session setup

- Pick a session id: `dbg-<short-slug>` (e.g. `dbg-login-freeze`).
- Every instrumentation line carries the literal marker `DBGLOOP:<session-id>` in a comment or log tag, so cleanup is one project-wide grep.
- Write down (in your response or a scratchpad note) the running list of instrumented `file:line` locations. You will delete all of them later.

## 1. Hypotheses first

Read the relevant code and state 2–4 concrete, competing hypotheses about the root cause. Instrumentation exists to *discriminate between them* — probes go at decision points where the hypotheses predict different values, not everywhere.

**The user's reproduction run is the scarce resource** — every dialog costs them a manual walk through the bug. Instrument for ALL live hypotheses in one pass, so a single run produces evidence that separates them. One-probe-per-round debugging burns a user round-trip per hypothesis; batched probes burn one for all.

## 2. The sink — default everywhere: a marked JSONL append-helper

**Default method, all environments: the instrumented process writes its own evidence, as JSON lines, directly to the file you will Read — via one small marked helper.** No intermediary. Every relay between probe and Read (console, syslog, unified logging, logcat, a stream-capture process) adds failure modes the direct write doesn't have: a capture process to start/track/kill, filter/predicate syntax that silently returns empty on mismatch, OS rate-limiting and truncation, interleaved noise from other processes. Fall back to a relay **only when the instrumented process cannot open a file you can read** — and then own the relay yourself (see fallbacks below).

**The helper contract** — same shape in every language:

1. Takes `(tag, hypothesisId, data)`; builds payload `{tag, h, run, data, ts}` (`run` = reproduction-run counter you bump between runs, `ts` = epoch millis).
2. Serializes to one JSON line, **appends** to the absolute session-dir path, creates the file if missing.
3. Never throws — a failed probe must not change program behavior. Swallow serialization/IO errors.
4. Carries the `DBGLOOP:<session-id>` marker in a comment on the helper; probes are one-line calls to it. Cleanup = delete helper + call sites.

Worked example (Swift; the same four steps translate directly to `fs.appendFileSync` in Node, `open(..., 'a')` in Python, etc.):

```swift
// DBGLOOP:<session-id> — debug helper, delete with probes
private func agentDebugLog(_ tag: String, h: String, run: Int, _ data: [String: Any]) {
  var payload: [String: Any] = ["tag": tag, "h": h, "run": run, "data": data,
                                "ts": Int(Date().timeIntervalSince1970 * 1000)]
  guard JSONSerialization.isValidJSONObject(payload),
        let json = try? JSONSerialization.data(withJSONObject: payload),
        let line = String(data: json, encoding: .utf8) else { return }
  let path = "<abs repo root>/.claude/debug-mode/<session-id>/native.jsonl"
  if let handle = FileHandle(forWritingAtPath: path) {
    handle.seekToEndOfFile(); handle.write((line + "\n").data(using: .utf8)!); try? handle.close()
  } else {
    FileManager.default.createFile(atPath: path, contents: (line + "\n").data(using: .utf8))
  }
}
```

Log **data, not prose**: variable values, branch taken, argument snapshots, timestamps, call ordering. The `h` + `run` fields make every line self-identifying — no cross-referencing against your notes, and stale lines from earlier runs can't be misread as fresh evidence even if truncation is missed.

**Verify the read path BEFORE instrumenting for real.** Emit one test line through the helper, trigger it yourself (app launch, module import — anything that runs without the user), and Read it back. If you can't read it, the environment belongs in the fallback table. An unreadable sink wastes an entire user round-trip.

**All evidence lands in one directory: `<repo-root>/.claude/debug-mode/<session-id>/`** (create it at setup). One known place to Read, one directory to delete at cleanup, and `.claude/` is conventionally gitignored so probes' output can't leak into a commit.

**File sinks use ABSOLUTE paths only.** A relative path resolves against the *process* cwd — dev servers, test runners, and spawned workers often run from somewhere other than the repo root, so the log lands where you won't look. Hardcode the full absolute path into the helper itself; never `./`, never path-joining against cwd.

### Can the process reach the session dir?

| Environment | Direct helper works? | If not — fallback |
|---|---|---|
| Node / backend / CLI / tests / Python / anything on the host | **Yes** — default helper | — |
| iOS **simulator** processes (Swift/ObjC native code) | **Yes** — simulator runs on the host filesystem; `FileHandle` with the absolute path just works | — |
| Web app JS in browser | No (no filesystem API) | `console.log('[DBGLOOP:<id>]', data)` with the same `{tag,h,run,…}` payload → `read_console_messages` / dev-server `preview_logs` |
| React Native JS (Hermes) | No fs built-in — but a file-system module already in the app (e.g. `expo-file-system`) restores the direct helper on simulator | else `console.log('[DBGLOOP:<id>]', JSON.stringify(data))` → Metro `preview_logs`; Metro you don't own: device-level capture sees JS console output (Android tags it `ReactNativeJS`; iOS dev builds route it through unified logging) |
| Android **emulator** native (Kotlin/Java) | No — emulator FS is not the host FS | `Log.d("DBGLOOP:<id>", json)` → background `adb logcat -s "DBGLOOP:<id>" > <session-dir>/android.log`; or helper writes to app sandbox + `adb pull` after the run |
| iOS **real device** | No — sandboxed FS unreadable from host | `NSLog`/`os_log` with marker → background `log stream --predicate 'eventMessage CONTAINS "DBGLOOP:<id>"' > <session-dir>/ios.log` |

Fallback rule: keep the **same JSON payload** through whatever relay you're forced into, run the capture yourself as a background process redirected into the session dir, and Read the file — never ask the user to copy-paste logs; a human relaying machine output is the least reliable link available. Background captures count as instrumentation: track their PIDs alongside the probe list — they get killed at cleanup.

### React Native native-side caveats

- **Native edits need a rebuild** — JS probes hot-reload; Swift/Kotlin probes do not. After instrumenting native code, rebuild yourself (`pnpm run ios` / `pnpm run android`) *before* showing the dialog, and wait for install to finish. Never make the user reproduce against a stale binary.
- **Clear before each run:** truncate the session-dir capture files (`: > <session-dir>/ios.log`); on Android also `adb logcat -c` so the buffer itself starts clean.
- **Bridge boundary bugs need probes on BOTH sides** — one JS probe at the call site, one native probe in the module method, same session id. Log the marshalled arguments on each side; a value that differs across the boundary is the finding.
- **Native crash (no log line at all):** pull crash reports — iOS `xcrun simctl spawn booted log show --last 5m --predicate 'messageType == fault'` or `~/Library/Logs/DiagnosticReports/`; Android `adb logcat -d -b crash`. Prefer `mobile_get_crash` / `mobile_list_crashes` MCP tools when available.
- Expo config plugins / prebuild output (`ios/`, `android/` generated dirs): probes there are wiped by the next `expo prebuild` — fine for the loop, but cleanup still means grep-verify both native dirs.

## 3. The dialog (hand off to the user)

Use **AskUserQuestion**. This IS the debug-mode dialog:

- `question`: the **numbered reproduction steps** ("1. Open the app. 2. Go to Settings. 3. Toggle dark mode twice. 4. Observe the crash.") plus one line on what you instrumented and what you expect the logs to reveal. Start the steps from a stated known state ("from a cold start, logged in as any user…") — an ambiguous starting point makes the run's evidence uninterpretable and wastes the round-trip.
- `header`: `Debug loop`
- `options`:
  1. **"Proceeded — I ran the steps" (Recommended)** — description: "I reproduced the issue; go read the logs."
  2. **"It's fixed"** — description: "Bug is gone; remove all debug instrumentation and wrap up."

The automatic "Other" free-text input is the extra-instructions field — users add observations, corrections to the repro steps, or pasted log lines there. Treat that text as instructions for the next iteration.

## 4. On "Proceeded"

1. Read the sink.
2. **Empty log is evidence too** — the instrumented path never executed. Move probes one level up the call chain; don't conclude "no data".
   - **Timestamps discriminate state from rendering.** When probes show internal state converging much faster than the user-visible artifact lasts (state settles in milliseconds, artifact lingers for hundreds), the bug is not in the state pipeline — it's in what draws it: an animation, a snapshot/portal, a transition. Stop instrumenting the state layer at that point; no state-side fix will reach it.
3. Compare evidence against each hypothesis. Tell the user in 1–2 sentences which hypothesis survived and why.
4. Then either:
   - **Confident in root cause** → apply the fix, *leave the probes in* (they verify the fix on the next run), and loop to step 3 with fresh repro steps framed as "verify the fix".
   - **Not confident** → refine/move probes to discriminate further, loop to step 3.
5. Rotate the sink between iterations (truncate the session-dir files / note the console timestamp) so old runs don't pollute the next read.

## 5. On "It's fixed" — mandatory cleanup

1. Kill every background capture process the loop started (tracked PIDs).
2. Grep project-wide for `DBGLOOP:<session-id>`; delete every hit, plus any helper functions, then delete `<repo-root>/.claude/debug-mode/<session-id>/`.
3. Verify: the grep returns nothing; run the project's lint/typecheck (e.g. `pnpm run lint && pnpm run compile`).
4. Summarize: root cause, the fix, and the log evidence that confirmed it.

Shipping instrumentation is a bug. Cleanup is part of "fixed", never optional, never deferred.

## Upstream bugs (the root cause is in node_modules)

The loop does not stop at the dependency boundary. Library code — including a RN library's native `ios/`/`android/` sources — is instrumentable and fixable like your own code. Pods and Gradle build RN libraries from their sources inside `node_modules`, so native probes there work after a rebuild.

**Probes** (temporary): edit `node_modules` directly — cheapest option, no patch ceremony for throwaway lines. Caveats:

- **pnpm hard-links from the global store.** An in-place edit can mutate the store copy and leak into every project on the machine. Break the hardlink first (`cp <file> <file>.tmp && mv <file>.tmp <file>`) or do the probing inside a `pnpm patch <pkg>` sandbox dir. npm/yarn copy files — direct edit is safe there.
- Metro does not hot-reload `node_modules` JS — restart the bundler with cache clear (`expo start -c` / `--reset-cache`) after JS probes; native probes need the usual rebuild.
- Cleanup: next install wipes these edits, but don't rely on it — revert them explicitly and grep-verify like any other probe.

**Fix** (permanent): never leave a bare `node_modules` edit as the fix — it dies on the next install. Escalate in this order:

1. **Package-manager patch**: `pnpm patch <pkg>` → apply fix → `pnpm patch-commit` (or `patch-package` on npm/yarn). Patch file is committed; works for JS *and* the library's native sources.
2. **Upstream it**: the patch is named technical debt with a known ceiling (dies on every version bump). File the upstream issue/PR as the follow-up that retires it, and link it in a comment next to the `patchedDependencies` entry or in the patch's commit message.
3. Only if patching is impossible (generated/prebuilt binaries): fork or vendor, and say so explicitly.

Verify a patched fix through the same dialog loop before calling it done — probes stay in during the patch-verification run.

## Rules

- **Stall guard:** ~5 iterations with no surviving hypothesis → stop, tell the user, and re-derive hypotheses from scratch, questioning the assumptions the first set shared (wrong layer, wrong process, wrong build?).
- Probes must be side-effect free — never change control flow, timing-sensitive code paths excepted (say so if logging could mask a race).
- Keep the diff small: probes are single lines; no refactoring while the loop is open.
- One session id per bug; a second concurrent bug gets its own id.

## Self-healing gate (mandatory before closing a session)

At "It's fixed" (step 5), after cleanup and before the final summary, distill what the session taught. The failure mode to avoid: accumulating incident-specific facts. A skill full of hardcoded incantations makes a stronger future model *dumber* — it follows stale specifics instead of reasoning. So the gate writes **mechanisms, not incidents**.

1. **Collect friction.** List what failed, surprised you, or cost a wasted user round-trip this session.
2. **Extract the mechanism.** For each item, ask "why did that fail?" until the answer describes how the system works, not what happened to this bug. ("`log show` returned nothing" → mechanism: unified logging filters by predicate syntax and time window, and both silently return empty on mismatch — empty result ≠ no logs.)
3. **Route by durability:**
   - **Mechanism / invariant** — would stay true in another project, next year, next tool version → update this SKILL.md. Write it as *principle + why*; any command next to it is an illustration and the sentence must survive that command going stale.
   - **Project- or version-specific fact** — this repo's layout, this tool version's flag, this library's quirk → the project's own `.claude/` skill or CLAUDE.md, never this file.
   - **Unexplained one-off** — you worked around it but can't state the mechanism → write nothing. A guessed rule constrains every future run; a gap just gets rediscovered.
4. **Prefer correction over accretion.** First try to generalize or fix an existing bullet; delete anything the session proved wrong or obsolete. Net growth of this file should be rare.
5. **Litmus before saving:** would a stronger model, reading the new line cold, be *helped* (understands the system better) or *fenced in* (told to mimic a past session)? Only save what helps.

Nothing to update is a valid outcome — the check itself is not skippable. Session-only workarounds that never reach the right file don't count; the next session inherits only what's written down.

Corollary for readers: commands in this file are worked examples of the mechanisms they sit under, not gospel — if your toolchain disagrees with a command, trust the mechanism, derive the current command, and route the discovery through this gate.
