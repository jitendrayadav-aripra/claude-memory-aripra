---
name: token-economy
description: Standing rules to keep token/context usage minimal across all tickets and sessions
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

Across every ticket and session, optimize for minimum tokens to reach the same result. The two big cost drivers in this repo are (a) large files (storyboards ~2,600 lines, AVCamera.swift, LBFilterUtils) and (b) context growing cumulatively — every tool result rides along in all later calls that turn.

**Why:** the user has repeatedly flagged a single change eating a large share of the context window; the cost is structural (big files × many round-trips), not a one-off.

**How to apply:**
1. NEVER read a large file whole — grep or Read with offset/limit to the target region. For a tweak, `git diff`/`git show` the related commit instead (see [[check-recent-commit-before-tweaking]]).
2. Delegate broad/uncertain "where is X" searches to an Explore/general-purpose subagent — it absorbs the file dumps and returns only the conclusion, keeping main context lean. Biggest single saver.
3. Batch + one-pass: independent reads/edits in ONE message; make all edits in one pass; never re-read after Edit to "verify" (Edit errors loudly on a bad match).
4. Fresh session per unrelated ticket (`/clear`) so one ticket's loaded files don't carry into the next.
5. Trust memory; don't re-derive settled decisions. NOW block first (≤~400 tokens), HISTORY only if needed; keep commit refs recorded.

**Refinements (2026-06-17 review — biggest remaining levers; user controls 6+7, I control 8):**
6. **Large recurring pastes → file path, not the payload.** Event JSON / log dumps are huge and recurring; the user drops them in a `/tmp` file (or I read the source) and I grep the ~10 lines that matter instead of ingesting ~300. For event JSON only the `themeTemplate.layers` (or one layer) is usually needed. Single biggest remaining saver.
7. **Visual storyboard work → user does in Xcode, I do code.** `CaptureSettingsView.storyboard`/`MainStoryboard` are ~2,100+ lines and whitespace-sensitive; the user re-tweaks+tests in Xcode anyway. Split: user takes pure-visual (fonts/frames/drag-position), I take `.m`/`.swift` logic + constraint constants. (move-to-end = good split; the restyle = the costly part I had to read big XML for.)
8. **Terser in-chat prose.** Output tokens count too — keep FULL rigor for posted GitLab notes, but shorten in-chat status/recaps to a few lines.
9. **Long-running (multi-day) session — free the oldest context safely.** In a session that isn't `/clear`-ed, the CHAT HISTORY is what bloats context, not memory. Because durable state is externalized (ticket NOW blocks, conventions, log/recipe refs), the old conversation is redundant → it's safe to drop: `/compact` (summarize, keep continuity — best mid-ticket) or `/clear` (full reset — best between tickets). SAFETY RULE: the active ticket's NOW block + any new conventions MUST be saved to memory FIRST. So keep NOW blocks current as we go (so the chat can be dropped any time without losing state). I OWN this — don't wait to be asked. Concrete moments to act (self-triggering): **(a) at every ticket boundary** (a ticket is done / parked / handed off) → I've just saved its NOW block, so offer `/clear` before starting the next unrelated ticket; **(b) mid-ticket after a heavy read** (big file, large log/JSON, long thread) → save the NOW block, then offer `/compact` to keep continuity. Each time: a one-line "context is getting heavy; NOW block is saved — `/compact` (or `/clear`)?" (I can't run it myself; the user runs the command.)

10. **23K-token pre-flight gate (2026-06-18).** Before issuing a new request/tool batch that would pull in a large chunk (~23K tokens or more — e.g. reading a big file/storyboard whole, a broad multi-file sweep, a large log/JSON ingest), STOP and ask the user first: tell them exactly what I'm about to read/look for and why, and let them narrow it (give a path, a grep target, or a smaller scope). Don't silently spend a big read — confirm scope at the ~23K threshold. Pairs with #1 (never read big files whole) and #6 (paths not payloads).
   - **The gate ALSO applies to OUTPUT, not just ingest, and to the paste-free auto-read logs (#11 / [[read-google-cloud-logs]]).** I VIOLATED this on #1823 (2026-06-18): repeatedly `tail -40`'d `GoogleCloudLogEntries.jsonl` and dumped 40+ raw JSONL rows into chat — each a ~20K+ turn. FIX: never `tail -N`/cat a raw log or big JSON into the chat — `grep` the `#<ticket>` marker (or specific field) and show only the matching rows, cap ~10. The auto-read convention removes the USER'S paste; it is NOT a licence to ingest/echo the whole file. One-line scope heads-up before any big read even when it's "routine."
   - **NOW HOOK-ENFORCED (2026-06-18), not advisory:** a PreToolUse(Bash) hook in `~/.claude/settings.json` BLOCKS any command that reads `GoogleCloudLogEntries.jsonl` with `tail`/`cat`/`head` and no `grep` (deny + the token-gate message); `grep`-based reads pass. Backup at `~/.claude/settings.json.bak`; manage via `/hooks`. So to read those logs I MUST `grep -F` the marker/field. (Other token rules remain advisory — only this one is mechanically enforced.)

**Read LumaBooth logs WITHOUT a paste whenever the user runs from Xcode (skip the paste / skip waiting for cloud):** LBLogger writes a JSONL queue at `<app data container>/tmp/GoogleCloudLogEntries.jsonl` (GoogleCloudLogging SPM pkg). Three channels by run target, ALL paste-free (VERIFIED 2026-06-17): (a) **Mac Catalyst** → read `~/Library/Containers/co.lumasoft.lumabooth/Data/tmp/GoogleCloudLogEntries.jsonl` directly; (b) **simulator** → `$(xcrun simctl get_app_container booted co.lumasoft.lumabooth data)/tmp/GoogleCloudLogEntries.jsonl`; (c) **physical iOS device via Xcode** → PULL it with `xcrun devicectl device copy from --device <UDID> --domain-type appDataContainer --domain-identifier co.lumasoft.lumabooth --source tmp/GoogleCloudLogEntries.jsonl --destination /tmp/iphone_gcl.jsonl` (UDID from `xcrun devicectl list devices`; "No provider" warning is non-fatal). Parse JSONL: `timestamp` is APPLE epoch (+978307200 → Unix UTC), plus `textPayload`/`labels`. So I read logs without a paste for EVERY target. Full recipe + caveats in [[read-google-cloud-logs]].

Extends [[reuse-existing-code]] and [[check-recent-commit-before-tweaking]].
