---
name: issue-AG-261-refit-vehicle-task-attachment
description: AG-261 — attach a real destination vehicle + task to a Refitted part, visible on both the original and destination Task Cards
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-10.** `tsc` clean both repos, `next lint` clean on every changed
frontend file. Closed by explicit instruction. This is AG-261's own deferred scope — Refitted
itself (status+note action, no destination) was built earlier under
[[issue-AG-260-resold-resolution-path]]'s 2026-09-03 pass; this ticket closes the "no vehicle/task
attachment yet" gap noted there, then extended further mid-build into making that attachment
visible on both sides.

**Built in two rounds:**

**Round 1 — store the destination (2026-09-09):**
- Migration `1788958159905-add-refit-vehicle-task-to-task-part.ts` — `refitted_vehicle_id` /
  `refitted_task_id` (`INT NULL`, no FK) on `task_part`, matching `refitted_by`'s plain-int
  convention.
- `markTaskPartAsRefitted` requires and validates both: destination vehicle exists and differs from
  the part's own current vehicle; destination task exists and belongs to that vehicle.
- Frontend (`part_note_panel.tsx`, Refitted action fields): real vehicle search
  (`/vehicle/search-vehicle/:search`, same endpoint `ag_vehicle_search.tsx`/`select_vehicle_modal.tsx`
  use) → click to select → task dropdown populated from `/task/:vehicleId` (the real Task Card
  endpoint). **Both required**, neither free text — client-selected only, independently
  re-validated server-side regardless.

**Round 2 — make it visible on both Task Cards (2026-09-09/10), after the user asked for it:**
- Migration `1789023120746-add-refit-from-task-part-to-task-part.ts` — `refitted_from_task_part_id`
  (`INT NULL`, no FK) on `task_part`, set only on the new destination-task entry, pointing back to
  the original part.
- `markTaskPartAsRefitted` now also creates a **second `TaskPart` row on the destination task**.
  Two real design decisions, both raised and confirmed by the user before building:
  - **No price fields, no paired `InventoryStock` row** — the money was already spent once, recorded
    on the original row only; duplicating it would double-count "money invested in parts" (user
    caught this risk before I'd even finished proposing the design).
  - **`status` is COPIED from the original part (Not Required/Incorrect), not set to "Parts
    Arrived"** — user's own correction of my first design. I'd proposed "Parts Arrived" reasoning it
    needed *a* real status; user pointed out the part already exists and isn't a fresh delivery
    event, so giving it "Parts Arrived" would wrongly pull it into the task-completion gate
    (`task.service.ts:1260` — Parts Arrived isn't in the allowed-to-complete set) and the
    Overview/Alert-Checks "unfitted parts" KPIs. Copying the origin status sidesteps both — Not
    Required/Incorrect are already completion-allowed and never counted as "unfitted" — making the
    entry purely informational, exactly as intended. `resolutionStatus: "Refitted"` set from
    creation on this new row too (so it's also excluded from every "unresolved flagged parts"
    bucket via the existing `TERMINAL_RESOLUTION_STATUSES` exclusion).
  - Verified before building: `calculateTaskSummary`/`calculateTaskPartSummary` (fired on part
    creation) is pure count/cache recomputation, no notifications/socket emits, already handles
    "Parts Arrived" as a normal case — confirmed no side effects. Original task's own completion is
    unaffected (its row's status never changes).
- `getTasks` (`task.service.ts`) batch-resolves display text (vehicle VRM + task category name) for
  both directions in 2 extra queries total, same one-query-for-the-whole-card pattern as the
  existing `hasNotes` fetch — not a query per part.
- `TaskPartsStatusBadge` — shows a dedicated emerald "Refitted" pill as the primary label **only**
  when `resolutionStatus === "Refitted"` (Resold/Scrapped/Listed keep showing origin status as
  before, unchanged). Tooltip gained "Refitted to: {VRM} · {category} (#taskId)" on the original
  part and "Refitted from: {VRM} · {category} (#taskId)" on the new destination entry.
- **Follow-up fix (2026-09-10):** user caught via screenshot that the tooltip's "Resolved: Refitted"
  line was now redundant (the pill already says "Refitted") and asked why the origin status wasn't
  shown anywhere any more. Agreed — swapped that one line to **"Originally: {status}"**, scoped only
  to the Refitted case (matches existing "Originally: {status}" wording already used for the same
  concept in Part Returns Audit's own `non_conforming_tab.tsx`). Resold/Scrapped/Listed's tooltip
  line is untouched — their badge still shows origin status as the primary label, so "Resolved:
  {resolutionStatus}" there is still non-redundant.

**Branch note (important if resuming near this area):** built entirely on
`AKASH/CR-AG-258-peopletab-split90D-and-nodatecolumn` in both repos — does **not** include AG-270/
AG-271's "AI recoverable resale price" work (that's on a sibling branch,
`AKASH/Feature-AG-271-AI-recoverable-resaleprice`, not merged — client hasn't confirmed those
changes yet, deliberately not merged per the user). If `TaskPartsStatusBadge`/`types/task.ts` look
like they're missing `invoiceUnitPrice`/`computeSuggestedResalePrice`/recoverable-price logic later,
that's why — it's not lost, it's just on the other branch.

**Not built / explicitly deferred:** nothing further flagged — the original "narrow scope" boundary
(not wiring the attachment into Overview drilldowns/Alert Checks tables) still stands, unchanged.

**Related:** [[issue-AG-260-resold-resolution-path]] (where Refitted itself was first built, and
where this attachment gap was originally noted and deferred).

---

## HISTORY

- 2026-09-09: User asked to resume AG-261 — no dedicated memory file existed for it (folded into
  AG-260's file as a deferred sub-item). Read AG-260's file, summarized what was live vs deferred.
- 2026-09-09: User confirmed the vehicle/task attachment as new ticket AG-261, required not
  optional. Explained the plan (reuse `/vehicle/search-vehicle/:search` + `/task/:vehicleId`,
  plain-int columns matching `refittedBy`) and got permission. User asked several verification
  questions (columns, table, not-free-text confirmation, dropdown vs other) — answered each before
  implementing. Built Round 1. `tsc`/`next lint` clean.
- 2026-09-09: User asked directly whether AG-261 was tracked as active — it wasn't (oversight, no
  file existed yet). Created this file, added to `MEMORY.md`'s active index.
- 2026-09-09/10: User asked for the entry to also show on the destination Task Card with status
  "Refitted", and for the original part to show the new vehicle/task details. Explained the plan
  (new TaskPart row, badge label swap). User asked two sharp verification questions before
  approving: (1) confirm the part isn't duplicated financially — caught that copying price fields
  onto the new row would double-count money invested; answered by confirming no price fields, no
  paired InventoryStock row. (2) confirm task completion / other Task Card processes aren't broken —
  investigated the real completion gate (`task.service.ts:1260`) and `calculateTaskSummary`/
  `calculateTaskPartSummary` before answering; original design (status "Parts Arrived") would have
  blocked destination task completion and polluted unfitted-parts KPIs. User then directly
  questioned why "Parts Arrived" was needed at all, since the part already exists — correctly
  identified that copying the origin status instead avoids both problems. Revised design agreed,
  Round 2 built. Along the way discovered the current branch doesn't include AG-270/AG-271's work
  (different branch, not merged — client hasn't confirmed those changes) — flagged to the user
  rather than silently proceeding; user confirmed to keep building on the current branch, they'd
  switch branches themselves. `tsc`/`next lint` clean both repos.
- 2026-09-10: User asked for migration timestamps (both, decoded), then commit messages — gave
  overly long ones first, user corrected ("short and meaningful, not such big ones") — saved as
  [[short-commit-messages]]. User then flagged via screenshot that the tooltip's "Resolved:
  Refitted" line was redundant and the origin status was no longer visible anywhere — asked for my
  opinion (not a go-ahead). Recommended "Originally: {status}" matching existing Part Returns Audit
  wording, scoped to Refitted only. User approved, built, verified. User then said to always
  proactively provide a commit message after changes without being asked — saved as
  [[always-provide-commit-message-after-changes]]. User then said to mark AG-261 as done.
