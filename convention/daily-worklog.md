---
name: daily-worklog
description: "How to produce the user's end-of-day change list (ticket + changes of the day) for pasting into Excel"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 949bc40a-0319-4fca-a1a5-306f086b798d
---

The user keeps a daily Excel worklog of "ticket + changes of the day." On request, produce that list.

**Trigger phrases** (auto-detect, no reminder): "show my changes", "show today's changes", "show me the
change list", "daily changes", "change list", "EOD log" — optionally scoped to a date ("…for yesterday",
"…on 2026-06-17", "…this week"). Default scope = **today** (use the session's current date).

**Dating rule: a code change is dated by its COMMIT date, not when it was written.** Work coded yesterday but
committed today belongs to TODAY's entry (matches the git cross-check, which keys on commit date). Non-commit
work (support reviews, investigations, infra) is dated to the day it was done.

**Sources:**
1. **[[worklog-daily]] = single source of truth** — dated, project-grouped Details blocks I append to EVERY
   session as work happens (see Maintenance below). For recall, read the scope date's `## <date>` section and
   output its `### <Project>` Details block(s) basically verbatim. This MUST work from a brand-new session with
   no prior context — that's the whole point — so the worklog has to already contain the day's work.
2. **Git = cross-check / backfill — ALWAYS run on `/changes`, not optional** (survives `/clear`; the safety net
   for any commit I forgot to log). For each local repo in `~/Desktop/Projects/MikeStuff/` (lumabooth_ios,
   lumashare_ios, lumabooth_windows, claude-skills) run
   `git log --all --oneline --since="<date> 00:00" --until="<date> 23:59" --author="ppogra\|Prakash Pogra"`.
   If a commit's `ref #<num>` isn't already reflected in the worklog block, fold it in.
   **Token guard (cheap by construction):** `--oneline` / `--format='%h %s'` ONLY — subjects are all we need to
   match `ref #<num>`; **NEVER `-p`/`--stat`/diffs.** One day-window + author filter = a handful of lines across
   the 4 repos (~grep cost, a few hundred tokens). The expensive path (diffs) is never needed for the cross-check.

**Output format — the user's Excel sheet columns are: `Date | Project | Developer | Details | Hours`.**
What they paste is the **Details cell text**. So by default OUTPUT JUST THE DETAILS BLOCK as plain text in a
code fence (line breaks = in-cell line breaks). NO commit hashes, NO status, NO table — they don't want those.
- **Details block** = per ticket worked that day, a line `ref #<num> <short title-case summary of what was done>`
  followed by `-` sub-bullet lines for the specific changes. Stack multiple tickets in one block (one `ref #`
  line each). Match the user's own voice: terse, technical, past tense ("Fixed…", "Added…", "Started testing…"),
  backticks for code symbols OK. Mirror the style of the sheet's existing Details cells.
- **One block per (date × project).** If the user worked on >1 project the same day (lumabooth_ios=LumaBooth,
  lumashare_ios=LumaShare, lumabooth_windows=LumaBooth Windows), output a SEPARATE Details block per project
  (their sheet rule: "create a duplicate date row per project"). Label each block with its Project + Date so they
  know which row.
- **Date** column format (only if asked / for labeling): `Ddd-DD-MM-YY`, e.g. 2026-06-18 → `Thu-18-06-26`.
  Developer = `Prakash`. Hours = left blank (user fills).
Only add the TSV table / commits / status if the user explicitly asks for the full table.

**Maintenance — APPEND to [[worklog-daily]] as work happens, EVERY session, auto (no reminder).** This is the
key habit: a future fresh session can only show the day's changes if they were logged when done. Log BOTH:
- **Commit work** — after committing, add/extend the day's `ref #<num>` line + sub-bullets.
- **Non-commit work** — reviewed/analysed a support ticket, read debug/cloud logs for an issue, posted a
  GitHub note/reply, locked a decision, tested on device without committing, spiked something. These have no
  git trace, so if I don't log them they're lost. Capture them as their own `ref #<num>` line + `-` bullets
  (e.g. `ref #1948 Reviewed Mike's reply + pulled the event JSON; confirmed OOM (45 full-res images); replied`).
- **Client/project infra without a ticket** — repo migration (e.g. GitLab→GitHub), CI/release setup, build/
  environment work on the client repos. This IS billable client work → log it as a plain line (no `ref #`) under
  the relevant Project.
- **DO NOT LOG Claude-internal / meta work** — memory edits, this worklog system itself, conventions/preferences/
  token-economy/process discussions, anything that isn't work ON the client's product or repos. The user does not
  time-track those. When unsure: "would this go on the client timesheet?" — if no, skip it.
  **This is a HARD rule (reinforced 2026-06-25 — I slipped once).** Explicitly NEVER log: saving/updating a
  memory file (pattern/feedback/convention/ticket), committing or pushing the memory repo (`/backup`), recalling
  or reorganising memory. A worklog entry must describe a change to the CLIENT product/repo or a client-facing
  action (code, review, investigation, posted note, decision, test) — nothing about my own bookkeeping. Before
  appending a sub-bullet, drop it if it's about "memory", "pattern", "feedback", "backup", or "worklog" itself.
- **Log the WORK, not the git/tooling mechanics.** Routine ops — `git pull`/`fetch`/`clone`, switching branches,
  opening a file — aren't billable on their own. Don't write "Pulled repo X (N commits)"; write the client-facing
  thing it enabled ("Re-checked the latest changes against the findings — none affect the template model"). If the
  mechanical action produced no investigation/decision/code, it doesn't belong in the worklog at all.
  **Same for ship/handoff mechanics — NOT billable bullets:** committing, pushing, opening a PR, assigning/
  labeling/reassigning an issue. The billable work is what was built/fixed/decided/tested; the handoff is at most
  a short OUTCOME clause folded into that bullet ("…handed to Mike to test, PR #1962"), never its own line. Don't
  write "Built the fix (3323c052 on 5.3, not pushed)… / Pushed to 5.4 / Opened PR + assigned + labeled". Commit
  SHAs may sit in the worklog FILE as tiny `(abc1234)` traceability tags, but the `/changes` OUTPUT strips them
  and all status (per the output rule above) — show only billable substance + the client-facing outcome.
Group under today's `## <date>` → `### <Project>`; create the section/block if missing. Terse, the user's voice.
Append at each milestone during the day, not just at end — don't batch-and-forget across a `/clear`.

**AUTOMATION + TOKEN ECONOMY (this must be cheap — see [[token-economy]]):**
- **Automatic, no prompting.** Append on milestones (commit, posted note, finished investigation, decision)
  without being asked; recall only on the trigger phrases. Don't ask permission to log routine work.
- **Appending is ONE small Edit** anchored on the `## <date>` header — never read the whole worklog to append.
  If today's section doesn't exist, prepend it under the `## Entries`/top marker. No re-read to "verify."
- **Recall is cheap:** (1) read ONLY the target date's section of [[worklog-daily]] (grep/offset to that `##`
  block, not the whole file); (2) ONE `git log --all --since/--until` loop over the 4 MikeStuff repos. Nothing
  else — no opening tickets, no scanning the codebase, no other memories.
- **Keep the file small:** prune `##` day-sections older than ~30 days once they're safely in the user's sheet.
- **Don't load this convention or the worklog proactively** — only when logging a milestone or on a trigger.
