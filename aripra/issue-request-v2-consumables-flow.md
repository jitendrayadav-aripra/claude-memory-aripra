---
name: issue-request-v2-consumables-flow
---

## NOW

- **Status (2026-07-22):** Backend AND frontend both COMPLETE. `tsc --noEmit` clean (exit 0) on
  both `car-planet-backend` and `carplanet`. Migration has NOT been run — waiting for user to run
  `migration:run` themselves. No manual browser verification done yet (no dev server run this
  session) — that's the only remaining item.
- **Repos:** `car-planet-backend` + `carplanet` — both done. No branch created, no commits made
  (working tree only, per this repo's "never auto-commit" rule).
- **Full plan/analysis + full build checklist (backend AND frontend) + file-impact summary lives
  in the repo, not here:** `d:\aripra\projects\tasks\todo.md`, section "Request V2 — experimental
  parallel Task Card request flow". Read that file for detail; this ticket is just the
  resume-pointer + decisions.
- **New backend surface:** `POST /request-v2`, `POST /request-v2/paginated-list`,
  `POST /request-v2/stats`, `GET /request-v2/task/:taskId`, `PUT /request-v2/:id/approve` (body:
  `consumableProductId`, `quantity?`, `actionNotes?`), `PUT /request-v2/:id/deny`,
  `PUT /request-v2/:id/order`, `DELETE /request-v2/:id` (added mid-frontend-pass, not in the
  original plan — needed for the mechanic-side delete-while-Pending affordance).
- **New frontend surface:** "+ Request V2" button on the Task Card (`create_task_modal_new.tsx`,
  gated `TASK_ADD_PARTS` — reused, no new permission), new form modal, new Task Card row renderer,
  new "Consumable Requests V2" Inventory tab (`page.tsx`, gated `INVENTORY_CONSUMABLES_REQUEST` —
  reused), new Approve-search modal.
- **Next action:** none pending from me on the Task Card / Inventory-tab side — remaining work is
  the user running the migration and doing a manual browser pass (checklist in `tasks/todo.md`'s
  Verify section). **A second, related ticket is now active on top of this one:**
  [[issue-AG-164-request-consumables-prep-menu]] — a dedicated Prep-menu submit surface reusing
  this same `task_part_request_v2` table and `consumables_request_v2_tab.tsx`, task-less/
  vehicle-less. Read that ticket if resuming work in this area.
- **Known pre-existing unrelated diff:** `git diff` on the backend shows a 1-line stray change in
  `task.service.ts` (~line 1265, task-completion-blocking query) that predates this feature —
  traced to an earlier session's "Remove Mechanic Request transaction_type" work, not touched by
  anything in this ticket. Don't mistake it for something this feature did.
- **Feature summary:** new independent "+Request V2" button on the Task Card, alongside (not
  replacing) the existing "+Request". Creates a generic pending request (no consumable/non-
  consumable decision yet). Inventory reviews in a new "Consumable Requests V2" tab: Approve
  (search+pick a consumable, deduct stock), Deny, or Order (routes into the existing non-
  consumable `task_part`/RFQ/PO pipeline, unchanged).
- **Key decisions confirmed with user 2026-07-21 (do not re-litigate without new input):**
  1. New isolated table `task_part_request_v2` — NOT bolted onto `task_part` (has a mature RFQ/PO/
     director-lock state machine + a "don't reorder this enum" warning) or `consumable_transaction`
     (requires a product id at creation time, which V2 must not have yet).
  2. Reuse existing permission keys `TASK_ADD_PARTS` (button) and `INVENTORY_CONSUMABLES_REQUEST`
     (tab) — no new permission migration for this feature.
  3. Widen the shared `getConsumableProducts` search (`consumable.service.ts`, `customText`
     filter) to also match `cp.oemNumber` — affects Stock tab search / RFQ "Add More Consumables"
     / `UseConsumableModal` too; confirmed acceptable since additive/superset-only.
  4. On "Order": the V2 request row disappears from the Task Card's V2 list; the newly created
     real `task_part` row appears in the normal Parts list via the existing `PartItem` component.
- **Don'ts:** never touch the existing "+Request" flow or Consumable Requests tab; never reorder
  `InventoryPartsStatus`; new columns are plain `int` refs only, no `@ManyToOne`/FK (project
  convention); never run `migration:run`/`revert` myself — hand the migration file back to the user.

---

## HISTORY

- 2026-07-21 — User gave the full feature spec (NEW_CONSUMABLES.md + consumables_request_tab.tsx +
  create_task_modal_new.tsx as context) with an explicit "do not implement yet — analyse, plan,
  ask questions, wait for approval" instruction. Ran 2 parallel Explore-agent research passes
  (existing "+Request"/Add-Part-Request/Use-Consumable flow in `create_task_modal_new.tsx` +
  `ag_part_modal.tsx` + `use_consumable_modal.tsx`; and the non-consumable "Parts Request From
  Tasks" dashboard + `task_part` entity/service + `InventoryPartsStatus` lifecycle), plus read
  `NEW_CONSUMABLES.md` in full for the consumables backend catalogue. Confirmed via grep that
  `consumable_product.oemNumber` already exists (added in an earlier, unrelated ticket logged in
  `tasks/todo.md`) but isn't yet used in the search query. Wrote the full analysis + a 10-section
  plan to `tasks/todo.md`, then asked 4 clarifying questions via AskUserQuestion — all answered,
  recorded above. Implementation has not started as of this entry.
