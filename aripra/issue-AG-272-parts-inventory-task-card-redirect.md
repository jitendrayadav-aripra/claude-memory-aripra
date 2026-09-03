---
name: issue-AG-272-parts-inventory-task-card-redirect
description: AG-272 — Parts Inventory (Parts Oversight) row click opens the Task Card instead of navigating to vehicle Stock Details
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-03** — built and closed same day it was raised. `tsc` clean both
repos, `next lint` clean on all touched frontend files. Raised internally (not client-sourced) after
noticing [[issue-AG-260-resold-resolution-path]]'s Part Returns Audit tab already had a better
pattern than what Parts Inventory was doing.

**What it replaced:** four row-click sites across
`carplanet/src/app/dashboard/leaderboard/parts-inventory/` navigated to
`/dashboard/stock-details?vid=...` on row click — landing on the vehicle's general overview, not
the specific task the user clicked through on. Mirrored the existing Task Card pattern instead
(`non_conforming_tab.tsx`'s `handleTaskIconClick`, also already used by
`leaderboard/parts-tab/parts-tab.tsx`'s `handleRowClick`): fetch task + vehicle, populate
`VehicleContext`, open the shared `CreateTaskModalNew` modal via `createPortal` — no page navigation
at all, in-place overlay.

**Backend** (`car-planet-backend/server/services/inventory/inventory.service.ts`) — added `task.id`
(aliased `taskId`) to the row output of:
- `getPartsInventoryLatePipelineList`, `getPartsInventoryBreachList`, `getPeopleAttributionParts` —
  all three already joined `taskPart.task` (for the technician column), so this was just an added
  `.addSelect("task.id", "taskId")`, no new join.
- `getPartsInventoryGoneList` — did NOT already join `task` (no technician column there) — added
  `.leftJoin("taskPart.task", "task")` + the select.
- `getPartsInventoryDrilldown` (Overview tab's drill-down panel) — added to its select, its inline
  return-type object, and its row-mapping.
- Shared `mapAlertRow` helper got `taskId: r.taskId ?? null` once, covering the four alert-shaped
  functions above in a single edit.

**Frontend types** — `PartsInventoryAlertRow` (in `parts_inventory.api.ts`, shared by 4 tables) and
the local `DrilldownRow` type (in `overview/parts_inventory_drilldown_panel.tsx`) both got
`taskId: number | null`.

**New shared hook** — `parts-inventory/use_task_card_modal.tsx` (`useTaskCardModal()`), factoring out
the fetch-task-and-open-modal sequence into one implementation instead of copy-pasting it into the
three consuming files. Returns `{ openTaskCard(taskId, vehicleId), isLoadingTask, taskCardModal }` —
consumers call `openTaskCard(...)` from their row-click handler and render `{taskCardModal}` once in
their JSX.

**Three tables rewired** to call `openTaskCard(row.original.taskId, row.original.vid)` instead of
their old `openVehicle(vid)` → `router.push(...)`, with the now-unused `router`/`routerDispatch`
imports removed:
- `alert-checks/parts_inventory_alert_table.tsx` (covers all 3 alert variants: Late/Breach/Gone —
  it's one shared component)
- `people/people_person_table.tsx`
- `overview/parts_inventory_drilldown_panel.tsx` (plain `<tr onClick>`, not MRT — same fix, different
  wiring point)

**Scope exception — deliberately excluded:** `alert-checks/parts_inventory_ready_cars_table.tsx`
("Ready Cars" table) keeps its old row-click → vehicle Stock Details behaviour. Its rows are
grouped/aggregated per vehicle (`COUNT(taskPart.id)` as `parts`), not one row per task/part — there
is no single task a click there could open a card for. Confirmed with the user before implementing,
not an oversight.

**No migration** — this ticket added a `SELECT`/`addSelect` column to existing queries and one new
`leftJoin`, nothing schema-level. Nothing to run.

**Related:** [[issue-AG-260-resold-resolution-path]] (the ticket whose Part Returns Audit tab
supplied the reference pattern — non_conforming_tab.tsx). Description text for this ticket (used
when the user created the real AG-272 in Jira) was drafted to match AG-260's own Jira ticket body
style (`## Why` / `## What to build` / `## Scope exception` / `## Note` headed sections) at the
user's explicit request.

---

## HISTORY

- 2026-09-03: User pointed at `carplanet/src/app/dashboard/leaderboard/parts-inventory` and asked
  for row-click to open the Task Card instead of navigating to vehicle Stock Details, "as we already
  did in case of the parts returns audit tab." Explored both the current behaviour (4 row-click
  sites navigating to `/dashboard/stock-details?vid=...`) and the reference pattern (confirmed it's
  a modal, `CreateTaskModalNew` via `createPortal`, not an actual page route) via two Explore-agent
  investigations — the second one specifically checked whether `task.id` was already available in
  each backend query (it wasn't; but 3 of 4 relevant query builders already joined `taskPart.task`
  for an unrelated reason, making the addition cheap; the 4th, Gone List, needed a new join; Ready
  Cars was flagged as structurally incompatible — grouped-by-vehicle, no single task per row).
  Presented the analysis + plan (including the "factor into one shared hook rather than duplicate
  3×" call) and got explicit permission ("ok go with that") including confirmation that Ready Cars
  should keep its existing vehicle-click behaviour. Built backend (5 query edits) + frontend (2 type
  edits + new shared hook + 3 files rewired), `tsc`/`next lint` clean both repos. User then asked for
  a Jira-ticket-style description "in the same format as AG-260" to file as a real ticket — fetched
  AG-260's actual raw Jira description (not just its memory summary) to match its heading style
  exactly (`## Why` / `## What to build` / `## Business rule` / `## Open questions` / `## Note`),
  drafted a proportionally-sized equivalent (dropped headings that didn't apply — no open questions,
  no business rule, added a `## Scope exception` heading instead for the Ready Cars carve-out). User
  filed it as the real AG-272, then said to mark AG-260 done, fold this work into AG-272 in memory,
  and mark AG-272 done too — both closed same day as raised, unusually fast turnaround.
