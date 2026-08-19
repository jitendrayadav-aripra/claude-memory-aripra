---
name: issue-AG-198-hide-stock-supplier-rfq
---

## NOW

- **Status (2026-08-19): DONE.** Fix applied and `tsc --noEmit` verified clean (0 errors,
  carplanet). No action pending on me — only a manual browser check is left, and that's on the
  user.
- **Requirement:** the RFQ "Add Supplier" picker (parts-request module) listed a pseudo-supplier
  named "STOCK." Now that parts can be fulfilled directly from stock instead of via RFQ, that entry
  is redundant in this picker. The `supplier` DB row must NOT be deleted (referenced by historical
  `part_supplier`/`rfq_supplier`/PO rows) — just hidden from this one list.
- **Fix:** `carplanet/src/app/dashboard/inventory/dashboard/supplier_modal.tsx` — client-side
  filter `supplier.company !== "STOCK"` applied right after the `GET /inventory/suppliers`
  response, before `setSuppliers`. No backend change.
- **Why scoped this way:** confirmed via grep that `SupplierModal` (this file) is imported ONLY by
  `rfq_modal.tsx` (parts-request RFQ "Add Supplier"). The admin Suppliers tab
  (`suppliers_tab.tsx`), the global `InventoryContext`, and the consumables RFQ picker
  (`consumable_supplier_modal.tsx` — a separate component) all hit the same
  `GET /inventory/suppliers` endpoint but through different code, so none of them are affected —
  STOCK stays fully visible/manageable everywhere except this picker.
- **Why no schema change:** `Supplier` entity has no `type`/`isActive`/`isSystem` column — STOCK is
  identified purely by the `company` string, same pattern as this codebase's existing
  `getSystemUser` (`userName: "System"` match in `user.service.ts`). The team has since moved to a
  type/flag column for a similar case (`getTradeDealUser`, ticket AUT-3097, because name-matching
  was called out as fragile) — considered that here but judged it over-engineering for a purely
  cosmetic, zero-business-logic list filter. Worth reconsidering only if STOCK needs to be
  identified elsewhere in code later.
- **Verified:** `tsc --noEmit` clean project-wide. Did NOT get a visual browser click-through this
  session — no browser-automation tool was available; told the user explicitly and handed them
  manual verification steps (open RFQ → Add Supplier → confirm STOCK absent; check Suppliers admin
  tab → confirm STOCK still there).
- **Open follow-on (not started, not requested yet):** `consumable_supplier_modal.tsx` (consumables
  RFQ picker) has the identical STOCK-in-list situation but is a separate component and was
  explicitly out of scope for this pass. Only touch it if asked.

---

## HISTORY

- 2026-08-19 — User gave the sole context as a folder path
  (`carplanet/src/app/dashboard/inventory/dashboard/parts-request`) plus a plain-English
  description — no Jira ticket pulled this time. Ran a background Explore agent to trace the RFQ
  add-supplier flow end to end: frontend `supplier_modal.tsx` → `GET /inventory/suppliers` →
  backend `supplierService.getAllSupplier()` (plain TypeORM `find()`, only filter is
  `id: MoreThan(0)`) → `Supplier` entity (no distinguishing flag column). Confirmed via grep that
  `SupplierModal` is imported only by `rfq_modal.tsx`. Presented two options (client-side filter vs.
  a backend `?excludeStock=true` query param) plus the in-repo precedent for name-string vs.
  type-flag identification of special records — user approved the client-side filter (Option A).
  Applied the one-line filter; backend (`:5555`) and frontend (`:3000`) dev servers were already
  running as the team's own instances, so none were started fresh. `tsc --noEmit` ran clean. No
  browser-automation tool was available to click through the modal, so visual verification was
  handed to the user with exact steps.
