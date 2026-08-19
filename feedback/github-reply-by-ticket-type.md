---
name: github-reply-by-ticket-type
description: "ROUTER for writing GitLab ticket replies — classify the ticket (log-investigation / new-requirement / existing-code-change) then use that type's reply structure. Auto-apply when working any GitLab ticket."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

How the user wants ticket replies written, distilled from his own high-quality past notes (ppogra) across
many tickets. **(2026-06-18: tickets are on GitHub now — same structure/tone, just post via `gh` not glab;
ticket numbers unchanged. This router is platform-agnostic.)** **On any ticket: first CLASSIFY the ticket type,
then follow that type's structure below.** Always write in the user's own human voice ([[github-reply-tone]]),
post via `gh issue comment` (see [[github-command-cookbook]] gh section), and pull evidence
(logs/firebase) before claiming a cause.

**Why:** user asked to lock in the best of his past+current reply craft so future sessions reproduce
it automatically by ticket type (2026-06-15). Confirmed his real notes are the gold standard.

## Cross-cutting (all types)
- **Conclusion first.** Lead with the answer ("Short version: …", "This is a hardware ghost-touch
  issue, not a button"), then the evidence.
- **Verbatim evidence in code fences**, annotated inline (log lines with `// what this means`, firebase
  JSON, diffs). Never paraphrase a log you can quote.
- **Distinguish symptom vs root cause** ("Line 298 is where it crashes — the symptom — not the bug").
- **Cite commit short hashes** for any archaeology ("at `0412e2ab8` (pre-#1926) it already did…").
- **End with a clear ask or next step** (a question, "Please test", assignment, or `ref …#note_<id>`).
- Collaborative/humble tone: "you were right to push on this", "Thanks @mike87", "will fix at source".
- Cross-reference prior notes by URL: `Done with ref https://gitlab.com/.../work_items/<n>#note_<id>`.

## Type A — Log-investigation ticket (user-reported bug, needs log diagnosis)
Detailed recipe in [[support-ticket-comment-structure]]. Shape:
1. **Version check first**: "Reviewed the logs, user is on the latest **X.Y, build NNNN**, on <os/model>" +
   the device line. Query Google Cloud logs by **device id** (`labels.d`).
2. State the conclusion (what it is / isn't).
3. Verbatim annotated **log trace** showing the failing sequence.
4. **Rule out the alternatives explicitly** ("volume button logs X first — never appears; shutter logs
   its own line — never appears; what's left is …").
5. Pull firebase event settings to confirm config when relevant
   ([[pull-lumabooth-event-firebase-json]], [[asset-url-removal-shared-event]]).
6. If unproven: "Can't reproduce but handled with `ref …`; if still happening, send the firebase
   `event_id`" — never overclaim.

## Type B — New-requirement / design-decision ticket
1. `@mention` the requester (usually `@mike87`).
2. Lay out the **options with a trade-off table** (Option A / Option B; columns like DB migration,
   data loss, resets-counter, breaks-analytics).
3. **Recommend** one with the engineering reason (consistency, no double-swap, no data loss).
4. **Ask one focused confirming question BEFORE coding** ("…remaining = 5-10 = 0, is that expected?
   Please confirm.") — don't implement a contested design until the reply lands.
5. When reversing course on feedback: "Got it, will simplify. Just want to confirm one case before
   reverting: …".

## Type C — Existing-code-change / fix-verification ticket
1. **Was it pre-existing or newly introduced?** Prove it with commit diffs + short hashes
   ("#1926 only changed *how* the asset list is built … the clone-the-stale-zip behavior is unchanged").
2. After fixing: **"Fixed at the source (commit `<hash>` on <branch>)"** + one paragraph of what was
   actually happening.
3. **Reliable repro steps** (numbered).
4. **"Tested OK:"** bullet list of the cases verified (incl. the negative/no-op cases).
5. **Flag the leftover edge case** as an explicit decision for the reviewer ("copying a pre-existing
   event still drops text — re-add copy-time regeneration for legacy events, or leave source-only?").
6. Before/After screenshots or `.mov` uploads where visual.

## Post-reply housekeeping
After a fix commit (`ref #num`), set the ticket label to `To Test` and assign back to the requester
(per the `gitlab` skill). For threaded replies use the `/discussions/<id>/notes` endpoint, not the flat
notes endpoint.
