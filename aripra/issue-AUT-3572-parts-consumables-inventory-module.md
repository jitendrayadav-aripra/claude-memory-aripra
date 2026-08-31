---
name: issue-AUT-3572-parts-consumables-inventory-module
description: AUT-3572 — turn Taz's ad-hoc HTML parts/consumables inventory prototype into a real AutoGrid module (Overview/Alert checks/People/Consumables tabs)
metadata:
  type: project
---

## NOW

**Status: DONE (for now), 2026-08-31.** Further work on this area is now tracked as separate
sub-tickets under AUT-3572, not here — this ticket will be marked done again once all of those
sub-tickets are complete. Re-open this file (`*/issue-AUT-3572-*.md`) if a sub-ticket needs this
NOW block's context (chip order, mechOwner sparsity, permission state) rather than re-deriving it.

All 4 sub-tabs built and working, and now feature-complete — People tab has all 3
pivots in chip order PO created by / Workshop manager / Technician (reordered 2026-08-31), Alert
Checks "nothing fitted" table's Workshop Manager column resolves for real. `LB_PARTS_OVERSIGHT_ACCESS`
migration is run; granting it to teams is an explicit manual admin action (Admin → Teams →
Leaderboard Access → "Parts Oversight"), owned by the user, not tracked here as outstanding.

**Known non-bug caveat:** the Workshop manager pivot shows almost entirely "(not recorded)" in
practice — verified against the live DB 2026-08-31: only 2/28,591 vehicles have `mech_owner` set at
all, 0/963 within the People tab's own dataset. Query/join is correct; underlying data is just sparse.
Not something to "fix" in code — flag to client if it comes up again. No other known open items
remain unless the user surfaces something new.

**Branch:** both repos on `AKASH/Feature-part-and-consumables-inventory-dashboard-2` (pre-existing,
not created by me).

**Refs:**
- Analysis doc: `my-docs/projects/parts-and-consumable-inventory-4th-project/initial-project-and-requirement-analysis.md`
- Living build log (updated after every change): `my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md`
- Plan/checkpoint notes: `tasks/todo.md` (repo root)
- Frontend: **moved 2026-08-26** from Inventory dashboard to Leaderboard section, and **visible label
  renamed to "Parts Oversight"** (better reflects on-order + unfitted + wasted parts, not just stock;
  internal identifiers deliberately left as `PARTS_INVENTORY`/`parts-inventory`) — now
  `carplanet/src/app/dashboard/leaderboard/parts-inventory/` (`parts_inventory_tab.tsx` sub-nav,
  `overview/`, `alert-checks/`), positioned immediately AFTER the "Parts" tab (corrected 2026-08-26 —
  was briefly before it). Reached at
  `/dashboard/leaderboard?tab=PARTS_INVENTORY&sub=OVERVIEW|ALERT_CHECKS`.
- Backend: `car-planet-backend/server/services/inventory/inventory.service.ts` — `getPartsInventoryOverviewStats`,
  `getPartsInventoryDrilldown`, `getPartsInventoryLatePipelineList`, `getPartsInventoryBreachList`,
  `getPartsInventoryGoneList`, `getPartsInventoryReadyCars`, `getConsumablesCategorySummary`,
  `getConsumablesStockList`. Routed under `/inventory/parts-inventory-*` (backend routes unchanged by
  the frontend relocation). Consumables endpoints share a new `resolveLastPriceByProductId()` helper
  (extracted from the Overview tile's inline logic) — one implementation of "which price for a
  consumable," reused 3 places now.
- Permission: **created** `LB_PARTS_OVERSIGHT_ACCESS` — migration
  `1787727229931-add-leaderboard-parts-oversight-permission.ts` (category `LEADERBOARD_TABS`,
  `related_permission: 'MD_LEADERBOARD'`, `permission_order: 7`, one above `LB_PARTS_ACCESS`'s 6 —
  so a user granted both still defaults to Parts, matching its now-first visual position, per
  `user.service.ts`'s lowest-order-wins logic). **Migration run by the user 2026-08-26** (timestamp
  decodes to 06:53:49 UTC / 07:53:49 BST). Permission row now exists but isn't granted to any team
  yet — that's why the tab is still invisible even to admin; grant it via the
  `teams_leaderboard_access.tsx` checkbox per-team. Also wired: `Permission` type
  field, team-admin checkbox (`admin/teams/teams_leaderboard_access.tsx`), and the sidebar
  default-landing-tab branch (`dashboard/sidebar/side_bar_functions.ts`) — found all three by grepping
  every place `LB_PARTS_ACCESS` appears, not just copying the migration. `leaderboard-tab.tsx` now
  gates on the real key directly (temporary `LB_PARTS_ACCESS` stand-in is gone).

**Next action:** All 4 checkpoints built, all 3 open questions resolved. Team-permission grant is
explicitly manual/out-of-scope (closed 2026-08-26). Nothing planned unless the user directs otherwise.

**Open questions — all resolved (see analysis doc's dated log for full detail):**
1. Workshop manager attribution — **resolved 2026-08-26**: it's `Vehicle.mechOwner` (the earlier "no
   data source" finding was a search-term miss — searched `workshop_manager`, the real field is
   `mechOwner`, found via `ops-daily-assignment.service.ts`'s `MECH_OWNER_CATEGORIES`, already
   surfaced elsewhere in the app as "Mech Owner"). People tab's 3rd pivot and the Alert Checks
   "nothing fitted" table's Workshop Manager column both now use it for real.
2. Price field — resolved as `poUnitPrice` for Overview/Alert Checks (client-confirmed for this build).
3. Vehicle status enum — resolved as hardcoded known set (`In Stock`/`Sold`/`Refunded`/`Cancelled`), no
   formal enum needed for now.

**Decisions:** Reporting/dashboard layer over existing data, not a new inventory model — extends
existing `server/routes/inventory/` stack per [[reuse-existing-code]]. KPI cards on Overview are
whole-card navigation links into Alert Checks (not an inline detail panel) — only the 2 Overview
charts keep an inline drill-down panel, since their buckets don't map onto the 4 Alert Checks tables.
Each Alert Checks table gets its own bounded, scrollbar-hidden scroll container (not page-level scroll)
since 4 lists are stacked on one screen.

**Constraints:** Never run `migration:run`/`migration:revert` myself — see [[feedback_migration_commands]].
Match `dashboard/leaderboard/parts-tab/parts-tab.tsx` / `parts_list.tsx` styling and table pattern exactly
(hex colors, not design tokens; explicit `size` + matching `align` on every MRT column) — this is the
concrete reference the user wants mirrored, not the token-based system elsewhere in the app. See
[[feedback_no_silent_scope_narrowing]] — do not defer/narrow agreed scope without surfacing it first.

---

## HISTORY

- 2026-08-21: Ticket opened. Jira fetch attempted, blocked (site not granted). User pasted description/requirements manually. Read prototype HTML in chunks (too large for one Read) to extract actual field mapping and business rules rather than assume from the ticket text alone — confirmed `mgr` (workshop manager) field exists in the *prototype's mock data* but does NOT exist in the real schema (prototype invented it for illustration).
- 2026-08-21: Explore agent confirmed data-source mapping — see analysis doc for full detail per requirement (parts pipeline, disposition statuses, vehicle status, staff attribution x3, consumables stock, reusable existing endpoints).
- 2026-08-21: Client confirmed price field (`poUnitPrice`) and consumables valuation approach (last procurement price). Built Checkpoint 1 (Overview tab) — first pass used design tokens + deferred all click-through, which was wrong on both counts (see [[feedback_no_silent_scope_narrowing]]); corrected to match `parts-tab.tsx`'s actual card styling and added drill-down. Built Checkpoint 2 (Alert Checks tab, 4 tables) after user clarified KPI cards should navigate there instead of opening an inline panel. Found and fixed two more bugs from user review: MRT column/header misalignment (missing explicit `size` + mismatched `align`) and a runaway infinite-scroll fetch loop (sentinel sat outside its bounded scroll container) — both now fixed, tables have their own bounded scrollbar-hidden scroll + correct alignment. Full detail in `NEW_PARTS_AND_STOCK_INVENTORY.md`'s changelog.
- 2026-08-26: User discarded the `INVENTORY_PARTS_INVENTORY` permission migration (never run anywhere, so safe to drop) and gave a new requirement: relocate the whole module from the Inventory dashboard to the Leaderboard section, positioned before "Parts". Moved the `parts-inventory/` folder, re-registered the tab in `leaderboard-tab.tsx` (that section defines tabs per-file, not via a shared types enum like Inventory does), repointed all internal navigation to `/dashboard/leaderboard?...`, and fully removed the old tab from `inventory_tab.tsx` (not just disabled). Explicitly told not to create the real permission migration yet — temporarily gated on `LB_PARTS_ACCESS` as a stand-in. `tsc`/`eslint` clean on both repos after the move.
- 2026-08-26: Renamed tab to "Parts Oversight", created + ran `LB_PARTS_OVERSIGHT_ACCESS` permission (repositioned after Parts per user correction, `permission_order` flipped to keep Parts as default), built Checkpoint 3 (People tab) and Checkpoint 4 (Consumables tab) — all 4 sub-tabs now built. Caught and fixed a real bug along the way: 3 Alert Checks tables' "PO by" column was sourced from `taskPart.createdBy` (the original requester) instead of the correct `PO.poCreatedBy` — same class of bug as [[feedback_distinct_actor_timestamp_fields]], now reinforced there with this recurrence. Then the user overturned the earlier "workshop manager: confirmed gap" finding — pointed to `ops-daily-assignment.service.ts`'s `MECH_OWNER_CATEGORIES`, revealing the real field is `Vehicle.mechOwner` (a search-term miss on my part, not an actual schema gap; already surfaced elsewhere in the app as "Mech Owner"). Added the 3rd People pivot and filled in the previously-hardcoded-blank Workshop Manager column on the Alert Checks "nothing fitted" table. All 3 of the ticket's original open questions are now resolved — module is feature-complete pending only the manual team-permission grant.
- 2026-08-31: User reported two People tab issues after the mechOwner work: (1) chip order was PO created by / Technician / Workshop manager instead of the intended PO created by / Workshop manager / Technician — fixed by reordering `ROLE_CHIPS`; this briefly broke the build (a `list_keys.ts` query-key factory still typed `role` as `"poBy" | "technician"` only, not widened when `mechOwner` was added) — fixed by widening it to match. (2) Workshop manager pivot showing only "(not recorded)" — investigated via a direct read-only DB query rather than re-reading the code, confirmed it's real sparse data (2/28,591 vehicles have `mech_owner` set overall, 0/963 in this dataset), not a query bug. No code fix needed for (2); documented as a known caveat instead.
- 2026-08-31: User separately reported the fix still failing on the remote `bitnami@ip-172-31-47-251` deploy server after pushing — traced via git log: confirmed the fix (`412114992`) was genuinely committed and merged into `beta3.0` locally, and `beta3.0` was up to date with `origin/beta3.0`. Initially suspected a `beta` vs `beta3.0` branch split (checked `origin/beta` and found it missing the entire Parts Oversight feature, not just this fix) but user clarified the actual merge target for this work is `beta3.0`, not `beta` — so pointed the user at remote-side checks instead (confirm branch/HEAD commit + re-check the file, consider stale dev-server build cache) since I have no SSH access to that server. Not yet confirmed resolved.
- 2026-08-31: User marked AUT-3572 **DONE (for now)** — moved from MEMORY.md's ACTIVE list to ARCHIVE.md. Remaining/future work on this module will come in as separate sub-tickets under AUT-3572; this ticket gets marked done again only once all those sub-tickets are complete. This ticket file (`issue-AUT-3572-parts-consumables-inventory-module.md`) stays as the canonical context (chip order, mechOwner sparsity finding, permission state) for whoever/whatever picks up those sub-tickets.
