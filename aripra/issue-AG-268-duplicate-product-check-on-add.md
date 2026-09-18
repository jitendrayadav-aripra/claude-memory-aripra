## NOW

**Status:** DONE (closed) 2026-09-18 — user said "mark the ticket as done" (memory-only per standing
rule, Jira status untouched). Note: re-test outcome after the `find`-vs-`findOne` bug fix was not
explicitly confirmed back in conversation before closing.

**Status (pre-close):** IMPLEMENTED (scope narrowed per user — UI-advisory check only, no DB constraint yet).
Both repos type-check clean. Not yet manually tested in the running app — see tasks/todo.md
"To verify manually" checklist.

**What shipped:** new `GET /consumable/check-duplicate-product` (service + controller + route in
car-planet-backend), wired into `ConsumablesAddEditModal.handleSubmit` (add mode only) in carplanet —
exact name+SKU match blocks with inline error, same-name-different-SKU shows the existing
`ConfirmDialog` component, no match / check failure proceeds to save unchanged. Edit mode untouched.

**Deliberately deferred (user's call, not forgotten):** entity `@Unique` decorator, migration adding
a DB unique index, and the safety-net hard-block inside `createConsumableProduct` itself — so this is
UI-advisory only for now; direct API calls or a race between two submits can still create an exact
duplicate. Also still open: the 3 pre-existing local-DB duplicate pairs ("nut"/N10272302, "tyre
repair kit"/539771440, "locking wheel nuts blade"/545772650) block that future migration until
resolved. OEM Number intentionally excluded from the check.

**2026-09-18 bug found + fixed during manual test:** user added "Oil filter" (lowercase f) with an
already-used SKU and it was NOT blocked. Root cause: `checkDuplicateConsumableProduct` used
`findOne`, which only grabs one arbitrary row among same-name matches — "Oil filter" is a generic
name shared by ~35 genuinely distinct products, each its own SKU, so the arbitrary row it compared
against had a different SKU than the true match. Confirmed case-insensitive name matching itself is
fine (collation `utf8mb4_0900_ai_ci`) — not a case bug. Fix: switched to `find` (all same-name rows)
and check whether any of them has the exact SKU. Single-function change, tsc clean.

**Next action:** wait for user to re-test in the running app (same repro: same name+SKU as an
existing row should now block/warn correctly). If they later want DB-level enforcement, resume from
the deferred items above (already scoped in tasks/todo.md history section).

**Refs (found via Explore agent):**
- Frontend modal: `carplanet/src/app/dashboard/inventory/consumables/consumables_add_edit_modal.tsx` (`ConsumablesAddEditModal`), `validate()` ~L108-131, `handleSubmit` ~L134-180, POST `/consumable/add-consumable-product`.
- Backend: `car-planet-backend/server/controllers/consumable.controller.ts:383-395` → `consumable.service.ts:1857-1919` (`createConsumableProduct`) → saves `ConsumableProduct` entity directly, no dup check.
- Entity: `car-planet-backend/server/entities/consumable/consumable-product.entity.ts` — `product` varchar NOT NULL, `sku`/`oemNumber` nullable varchar, **no unique index/constraint on any of them**.
- Pattern to reuse: `consumable-subcategory.service.ts:42-47` and `supplier.service.ts:130-140` — `findOne(...)` + `throw new StringError(...)`, excluding soft-deleted rows (`Not(ConsumableProductStatus.DELETED)`).
- No DB-QUERY-CATALOGUE.md exists in this repo; no documented uniqueness notes anywhere.

**Next action:** Waiting on user decision on: (a) case-insensitive/trim matching for name+SKU exact dup (hard block),
(b) UX for same-name-different-SKU (warn-and-confirm vs block), (c) whether to also add a DB unique index. Do not
implement until user says go ahead.

**Decisions/don'ts:**
- Do NOT implement anything until explicit go-ahead is given after the plan is shown (per [[ask-before-applying-is-separate-step]] style rule — analysis/plan requests are not permission to build).
- Follow bottom-up / minimal-impact conventions already used elsewhere in this project (backend QueryBuilder pattern, no raw SQL).

---

## HISTORY

- 2026-09-17: Ticket opened from a screenshot of the "Add New Product" dialog (Inventory > Stock > Stock Catalogue).
  Reported gap: no check for existing product by name/SKU before adding, causing duplicate stock catalogue entries.
