---
name: debugging-method
description: "How to debug (esp. visual/rendering or unclear-behavior bugs) — measure-first via a clearly-marked TEMP logger, one pass, auto-read the logs, auto-clean before commit"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 4589d9a8-73ce-4446-975d-859b10b7769e
---

For bugs whose cause isn't obvious from the code — especially **visual/rendering** ones (stretch, squash,
mis-layout) or "why does it behave like X" — do NOT theorize across build cycles. **MEASURE FIRST.** Each wrong
turn is a full Xcode build + manual device/Mac test, so the cost is build cycles, not just tokens.
**The build is the USER's, not mine — do NOT run `xcodebuild`/compile the app myself.** When code is ready,
say so and ask the user to build + run in Xcode; they'll run it and I read the logs paste-free (per
[[token-economy]]) if needed. (Corrected 2026-06-30 on #1953 — I'd kicked off an `xcodebuild` unprompted.)

**Auto-add a TEMP logger (no need to be asked):**
- Drop in a clearly-marked temp logger that captures **ALL candidate values in ONE pass** — don't iterate logging
  round after round. For a rendering bug that's e.g. bounds + frame + transform + drawableSize + resizingMode +
  source-image size + rotation, all on one line, on every relevant code path.
- Mark every temp line with ONE greppable tag, e.g. `#<ticket> TEMP DEBUG` (+ a payload prefix like `#<ticket>-XX`
  so the emitted lines are easy to filter). ObjC: `LBLog(@"…")`; Swift: `LBLogger.shared.logInfo(message:)`.
- **READ the logs automatically** from the right place per [[read-google-cloud-logs]] — I already know
  when/where (Mac → container `tmp/GoogleCloudLogEntries.jsonl`; sim → `simctl get_app_container`; device →
  `xcrun devicectl device copy from`). No path/paste asked.

**Auto-CLEANUP before commit (no need to be asked):**
- Before committing, `grep` the marker (`#<ticket> TEMP DEBUG`) and remove EVERY temp artifact: log calls, helper
  methods, debug-only flags/ivars, and any import added only for the logging. Final grep must return 0 — nothing
  temp ever ships. (Exception: a temp logger the USER added on purpose — leave it, don't commit it.)

**Also (lessons from the #1823 rotation-stretch saga — many build/test/revert cycles):**
- **Lock the repro matrix early** — ask "when exactly: launch / open editor / select / resize / rotate?". The
  precise repro narrows the cause fast.
- **Check hard API constraints before proposing an approach** (e.g. `objc_subclassing_restricted`, availability).
- **Confirm root cause → pick the DETERMINISTIC fix → stop.** Don't thrash across variants (each = build/test/
  revert). Once #1823 was "Metal drawable wrong aspect," pinning drawableSize was the clean answer.
- **Risky multi-file experiments → isolated git worktree/stash** so a "reset" costs nothing and approaches can be
  diffed instead of losing ground.
- **Trust the user's domain leads** — test their hypothesis before mine; they know the app's behavior.

See [[issue-1823-resizable-liveview]] (dead-ends), [[token-economy]].
