---
name: issue-AG-294-refitted-destination-badge-swap
description: AG-294 — on the ORIGIN Task Card of a Refitted part (not the destination), show the origin status as the primary badge and move "Refitted" resolution detail to the hover tooltip instead; destination card unchanged
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-14.** Title/description written to Jira via `editJiraIssue`.

**Confirmed current behavior against live code** (backend `markTaskPartAsRefitted`,
`inventory.service.ts:3694-3782`; frontend `task_parts_status_badge.tsx`):
- Both the origin `taskPart` AND the new destination `refitEntry` get `resolutionStatus = "Refitted"`
  set explicitly (lines 3731 and 3772) — that's why both Task Cards currently show the same
  "Refitted" pill as the primary badge; there's no per-row distinction today.
- The only field that actually distinguishes which row is which: `refittedFromTaskPartId` is set
  ONLY on the destination `refitEntry` (line 3776, pointing back to the origin's id) — never on the
  origin row. On the frontend this surfaces as `refittedFromVehicleVrm` being present only for the
  destination card (batch-fetched, per AG-289).
- `refitEntry.status` (line 3771) is already copied from the origin's `status` (e.g. "Incorrect"/
  "Not Required") — so the data needed for the new requirement already exists on the destination
  row, this is a pure frontend rendering change.

**Plan:** in `task_parts_status_badge.tsx`, only the destination side's render path changes:
- Add `isRefittedDestination = !!refittedFromVehicleVrm` (true only for the destination row).
- Line 132's `if (isRefitted)` early-return (the "Refitted" pill) becomes
  `if (isRefitted && !isRefittedDestination)` — so only the ORIGIN card still shows the "Refitted"
  pill as primary; the destination card falls through to the normal status-badge branches below,
  which already render `status` ("Incorrect" → orange, "Not Required" → gray fallback, both already
  styled/handled — confirmed, no new styling needed).
- Tooltip's resolutionStatus line (111) currently shows "Originally: {status}" for any `isRefitted`
  row — on the destination row this should flip to "Resolved: Refitted" instead (i.e. the normal,
  non-special line), since the origin status is now the visible primary badge and repeating it in
  the tooltip would be redundant. Condition becomes
  `isRefitted && !isRefittedDestination ? "Originally: ${status}" : "Resolved: ${resolutionStatus}"`.
- Everything else — `refittedToLine`/`refittedFromLine`, the origin card, the info icon's visibility
  condition, `recoverablePrice` — untouched, per the ticket's own "don't change any other details."

No backend change needed at all — purely a frontend conditional-render fix.

**Built 2026-09-14, `tsc`+`next lint` clean.** Exactly the 3 changes planned, all in
`task_parts_status_badge.tsx`:
1. New `isRefittedDestination = !!refittedFromVehicleVrm` flag (true only on the destination row).
2. Primary-pill early return: `if (isRefitted)` → `if (isRefitted && !isRefittedDestination)` — only
   the origin card still shows the green "Refitted" pill; destination falls through to the normal
   status branches, which already handle "Incorrect" (orange) and "Not Required" (gray fallback)
   with no new styling needed.
3. Tooltip's resolution line: `isRefitted ? "Originally: ${status}" : "Resolved: ${resolutionStatus}"`
   → `isRefitted && !isRefittedDestination ? ... : ...` — destination card now reads "Resolved:
   Refitted" instead of "Originally: {status}" (redundant now that status is the visible badge).
- Origin card, `refittedToLine`/`refittedFromLine`, info-icon visibility, `recoverablePrice` —
  untouched, per the ticket's explicit "don't change any other details."

Title/description written into Jira via `editJiraIssue` (the drafted text verbatim).

**2026-09-14 — correction: had the origin/destination direction backwards.** User caught it
immediately after the Jira write: "you have done the opposite of what I requested." Correct
direction, per the user's own precise restatement ("Refitted" at the location where the part is
being refitted; origin status at the location it's being sourced from):
- **Destination card** (where the part is being refitted TO) keeps the "Refitted" pill exactly as
  it already behaved pre-ticket — **no change there**, contrary to the first (wrong) build.
- **Origin card** (where the part is being sourced FROM) is the one that actually changes: shows
  origin status as primary badge; hover flips to "Resolved: Refitted" + "Refitted to: ...".

Root cause of the mix-up: the original chat description used "where the part is being refitted"
almost interchangeably for both sides ("task card from where the part is being refitted" vs "other
task card where the part is being refitted"), which didn't disambiguate — only the correction's
sharper phrasing ("sourced from" vs "refitted [to]") pinned it down. Restated the corrected
understanding back to the user before re-implementing, rather than silently flipping it a second
time — user confirmed, then re-implementation done.

**Re-fix, `tsc`+`next lint` clean:** flipped both conditions from `!isRefittedDestination` to
`isRefittedDestination` — the "Refitted" pill early-return (line ~145) and the tooltip's
"Originally: {status}" vs "Resolved: {resolutionStatus}" branch — no other logic touched.
`isRefittedDestination`'s own definition (`!!refittedFromVehicleVrm`) was already correct, only its
usages were backwards.

**Jira title/description rewritten via `editJiraIssue`** to match the corrected direction (origin
card is now the one described as changing, destination explicitly called out as unchanged).

**Related:** [[issue-AG-261-refit-vehicle-task-attachment]] (built the Refitted badge + to/from
tooltip lines this ticket is modifying).

---

## HISTORY

- 2026-09-14: Ticket opened by user request ("New ticket AG-294"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]], before any analysis.
