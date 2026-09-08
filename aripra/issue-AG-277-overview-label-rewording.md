---
name: issue-AG-277-overview-label-rewording
description: AG-277 — Parts Oversight Overview label rewording and drill-down table styling fixes (Charles's UI review feedback)
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-07.** `tsc`+`next lint` clean. Pure UI-copy and styling changes — no
backend, no data, no migration. Closed by explicit instruction.

**Source:** UI review feedback from Charles, given as 3 annotated screenshots (Overview KPI cards
numbered 1-5, the "Flagged Parts Pending Resolution" card zoomed in, and Alert Checks with the "rows"
badge and a VRM circled). Title/description were empty in the real Jira ticket going in — drafted by
me per the user's explicit request, in numbered-list form matching how the user themselves listed
the changes (not the free-form `##`-headed AG-260 style used for earlier tickets — this one didn't
need it, being pure copy/style).

**What got built, in `carplanet/src/app/dashboard/leaderboard/parts-inventory/`:**
- `overview/overview_tab.tsx` — 5 KPI card labels reworded: "Received but unfitted" / "Parts arrived
  on sold cars" / "Overdue fitting (>2 days)" / "Awaiting delivery" / "Consumable stock value"
  (was: "Arrived, not used — on shelf" / "On cars that are GONE" / "Breaching 2-day rule (in-stock
  cars)" / "On order" / "Consumables on hand"). "Flagged Parts Pending Resolution" chart renamed
  "Parts Pending Return or Resale" — title, description, and all 4 bucket cards' title+description
  text reworded per the user's exact mapping. **Also updated, on my own initiative (flagged, not
  silently done):** the 4 buckets' `setDrilldown({title})` panel-header text and the
  `resolutionSegments` array's `label` fields (feeding the segmented-bar legend below the cards) —
  the user's list only named the description-line copy, but the same concept (e.g. "Ready for
  sale"→"Awaiting eBay Listing") shows up in 3 places per bucket (card label, drilldown panel title,
  legend), and updating only one would have left the other two visibly stale/inconsistent.
  Deliberately did NOT touch the separate "Ageing of arrived-but-unused parts" chart's own drilldown
  title (`Arrived, not used — ${entry.label}`) even though it echoes KPI card #1's old wording — not
  named in the request, different chart entirely, avoided scope creep.
- `alert-checks/parts_inventory_alert_table.tsx` — count badge `"{total} rows"` → `"{total} parts"`
  (covers all 3 Alert Checks variants — Late to arrive/Breach/Gone — since they share this one
  component). Checked every sibling badge first (Ready Cars: "cars", People detail: "parts",
  Consumables: "lines") — "parts" was the consistent choice, not an arbitrary pick; flagged this as
  my own interpretation of "remove the word rows" before building, user didn't object.
- VRM/registration column recoloured `text-[#1B55EC]` (blue) → `text-[#0F172B]` (the same near-black
  already used for every other primary/value text in this module) in **4 places**:
  `alert-checks/parts_inventory_alert_table.tsx`, `alert-checks/parts_inventory_ready_cars_table.tsx`,
  `people/people_person_table.tsx`, `overview/parts_inventory_drilldown_panel.tsx`.

**Related:** [[issue-AUT-3572-parts-consumables-inventory-module]] (the module this all lives in),
[[issue-AG-272-parts-inventory-task-card-redirect]] (the reason VRM's blue "link" styling became
misleading — it opens the Task Card now, not a real navigation).

**2026-09-07 follow-up (same session, after this ticket was already closed) — Consumables KPI card
click-through added.** Separate small fix in the same file, no new ticket number given — the
Consumables tile was the only Overview KPI card with no `onClick`, a leftover gap the file's own
comment flagged as deferred until the Consumables sub-tab existed (it has, since Checkpoint 4).
Added `goToConsumables()` mirroring the existing `goToAlertChecks()` (navigates to
`&sub=CONSUMABLES`), plus the same hover/cursor styling the other 4 cards already have. Also updated
the file's own top-of-file comment, which still claimed "Consumables tile has no click-through yet."
`tsc`+`next lint` clean. Documented in `NEW_PARTS_AND_STOCK_INVENTORY.md`'s changelog as its own
entry rather than folded silently into AG-277's, since it wasn't part of the original AG-277 scope.

---

## HISTORY

- 2026-09-07: User shared 3 annotated screenshots (Charles's UI review) and gave a numbered list of
  label/styling changes wanted, explicitly said description was empty in the real AG-277 and asked
  for one to be drafted. Traced every change to its exact code location before drafting anything —
  confirmed "remove the word rows" is exactly one spot (shared Alert Checks table component, matches
  the circled "135 rows" badge precisely) by checking every sibling badge in the module for its
  actual wording convention first, and confirmed "change all reg. to black" touches exactly 4 render
  locations (not more, not fewer) by grepping the whole module for the blue hex. First description
  draft was prose-based; user asked for a title plus the label mappings in numbered/bulleted form
  matching how they'd listed it — redrafted accordingly. Got explicit "yes go for it" and applied all
  changes; `tsc`+`next lint` clean. One implementation slip during the KPI-label edit (an Edit call
  accidentally left a stray `<p style={{display:"none"}}>` placeholder in the JSX) — caught
  immediately via the next Read and cleaned up before continuing, not left in the diff.
