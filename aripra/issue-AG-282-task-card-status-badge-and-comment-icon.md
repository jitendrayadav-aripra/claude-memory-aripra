---
name: issue-AG-282-task-card-status-badge-and-comment-icon
description: AG-282 — Task Card part-status badge colors, comment icon "has notes" indicator, and first-flagged-date info tooltip
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-08.** `tsc`+`next lint` clean both repos. Closed by explicit
instruction. Not part of the Parts Oversight module — this is a general Task Card component
(`stock-details/task-v2/`) used across the whole app, so **not documented in
`NEW_PARTS_AND_STOCK_INVENTORY.md`** (out of that doc's scope, unlike AG-260/269/270/272/273/274/277/278).

**Three pieces of work, all in `TaskPartsStatusBadge`
(`carplanet/src/app/dashboard/stock-details/task-v2/components/task_parts_status_badge.tsx`) and its
call site in `create_task_modal_new.tsx`:**

1. **Badge colors for Used/Faulty/Incorrect/Returned** — these 4 statuses previously all rendered as
   the same flat gray pill (the component's fallback branch had zero per-status color logic beyond
   Arrived/RFQ/Ordered). Added an explicit color map matching colors already established elsewhere
   in the app: Used `#EDF6F3`/`#DEEEE9`/`#047857`, Faulty `#FAEFEF`/`#F6E1E1`/`#B91C1C`, Incorrect
   `#FEF8EE`/`#FEF2DF`/`#FE9A00`, Returned `#FFEBF9`/`#FFD6F2`/`#FF00B7`. All other statuses
   (Not Required, Deleted, Pending, the existing Arrived/RFQ/Ordered pills) untouched.

2. **Comment icon "has notes" indicator — the one that went badly wrong before landing correctly.**
   Wanted: the "Update Status / Comment" icon (grey by default) turns blue once a part has at least
   one comment in its thread. Real story, worth reading in full if this area is touched again:
   - Backend: added a batch query (`note` table, `type = TASK_PART_NOTES`, `extra_id IN (...)`) to
     compute `hasNotes` per part, one query per Task Card load not one per part — **first landed in
     the WRONG function** (`getTasksList`, reached via `GET /task/v2/:vehicleId`). The Task Card
     actually loads via `GET /task/:vehicleId` → a different, near-identically-shaped function called
     `getTasks`. Two lookalike functions in `task.service.ts`, easy to confuse — I assumed the `v2`
     endpoint based on a naming pattern from an earlier ticket's notes, without verifying against the
     actual live request. Every fix attempt silently did nothing because the code was never executed.
     **Caught by the user pasting their actual dev-server terminal log** showing `GET /task/:vehicleId`
     firing with no trace of my `[TEMP DEBUG]` line — that one log line was the actual proof, not
     further code-reading. Fixed by moving the batch-fetch into the real `getTasks`, reverting
     `getTasksList` back to untouched.
   - Frontend: `hasNotes` added to `TaskPart` type. Two more bugs found and fixed along the way, both
     in `create_task_modal_new.tsx`: (a) adding a plain comment (no status change) never notified the
     parent component at all — `AGPartStatusNoteModal` only refreshed its own internal thread, so the
     row's `hasNotes` stayed stale until a full reload; fixed with a new `onNoteAdded` callback fired
     after a plain-comment save, wired to a new `handlePartNoteAdded` parent handler that flips
     `hasNotes: true` locally. (b) a status-change action replaced the whole part object with the raw
     `PUT /task/part-status/:id` response, which never computed `hasNotes` — silently wiping the flag
     back to blank even on parts that already had comments; fixed by merging `hasNotes: true` directly
     onto that replacement (this modal always requires a comment alongside any status change, so the
     assumption is safe).
   - Migration `1788868589350-add-index-note-type-extra-id.ts` — adds an index on `note(type,
     extra_id)`, since that pair had none; benefits this new query AND the pre-existing
     `getTaskPartNotes`/non-conforming `latestComment` batch-fetch that already filter on the same
     columns. **Not run** — standard hand-back-to-user rule, same as every migration this session.

3. **Info icon — first flagged date.** The existing AG-274 info icon (originally only shown for
   resolved parts, tooltip "Resolved: X") now shows whenever a part has ever been flagged at all
   (`firstFlaggedDate`/`firstFlaggedDateText`, real write-once columns already on `TaskPart`, no
   backend change needed — just never surfaced to the frontend before). Tooltip always shows "Flagged:
   {date}" (run through `formatTableDate()`, not shown raw — matches the established `*DateText`
   convention), with "Resolved: {status}" appended only when a resolution exists. **Deliberately no
   "flagged by" user** — asked the user first; confirmed the schema has no such field (only
   `partActionedBy`, which gets overwritten on every subsequent terminal status change, so pairing it
   with the write-once first-flagged date would misattribute on any part touched more than once) —
   user chose date-only rather than a misleading actor.

**Related:** [[issue-AG-274-task-card-status-lock-after-resolution]] (the original info-icon
mechanism this extends).

---

## HISTORY

- 2026-09-08: User opened with the badge-color mismatch (screenshots comparing Task Card badges vs
  Part Returns Audit's), gave exact hex values matching what was already used in
  `non_conforming_tab.tsx`'s `STATUS_BADGE_STYLE`. Built directly (no separate investigation needed,
  values were exact matches to known-existing colors), `tsc`/lint clean, gave title+description+commit
  message.
- 2026-09-08: User asked for the comment icon to turn blue when a part has existing comments,
  confirming first that this wouldn't affect Task Card load speed (answered: bundled into the
  existing single fetch, one batched query not per-part, small per-vehicle dataset — no new frontend
  request at all). Built the `hasNotes` batch-fetch + icon swap + new blue SVG asset + a migration for
  an index on `note(type, extra_id)`. User tested — icon only worked for brand-new comments added from
  the Task Card itself, never for pre-existing ones or ones added from Part Returns Audit. Multiple
  rounds of "did you restart the backend" diagnosis (correctly ruled out once the user confirmed a
  full close-and-restart of both frontend and backend). Added a `[TEMP DEBUG]` log per the project's
  own debugging convention (measure first) since code-reading alone couldn't find the bug — user
  pasted their actual terminal output, which showed the real request was `GET /task/:vehicleId`, not
  `/task/v2/:vehicleId` as assumed, and the debug line never appeared at all — proving the fetched
  function (`getTasksList`) wasn't the one actually running. Root cause found: two near-identical
  functions in `task.service.ts` (`getTasks` vs `getTasksList`), wrong one edited from the start.
  Fixed by moving the logic to the real one and reverting the wrong one cleanly. Along the way, also
  found and fixed two more real bugs in the parent's local-state handling (plain-comment path never
  notified the parent at all; status-change path silently wiped `hasNotes` via a raw response
  replace) — both surfaced by re-tracing the exact prop/callback chain rather than more guessing.
- 2026-09-08: User asked to also show first-flagged-date (and originally "the user") on the same info
  icon, alongside the resolved status if present. Checked the entity first rather than assuming a
  "flagged by" field existed — confirmed it doesn't (`firstFlaggedDate`/`firstFlaggedDateText` are
  timestamp-only, write-once). Flagged the mismatch risk of substituting `partActionedBy` (overwritten
  on every subsequent status change, not write-once like the date) rather than silently picking one;
  user chose date-only. Built cleanly, no bugs this round — confirmed via full read-back before
  reporting done, given how much back-and-forth the previous item required. Ticket then closed by
  explicit instruction, no further code changes.
