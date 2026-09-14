---
name: issue-AG-292-kpi-card-hover-details
description: AG-292 — official exact-wording hover tooltips for the Overview tab's 5 KPI cards, superseding AG-291's own draft wording
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-14.** All 5 `KpiInfoTooltip`
`title` strings in `overview_tab.tsx` replaced verbatim with the wording below (structure/component
unchanged from AG-291's build). Confirms my guess —

**2026-09-14 same-day follow-up — dropped the misleading "pre-order" count from the "Awaiting
delivery" card's caption.** User's own observation (asked for opinion first, per
[[feedback_opinion_request_is_not_a_go_ahead]] — answered, then got explicit separate go-ahead):
the caption read "{count} parts · {lateOrNoEta} late or no ETA · {preOrder} pre-order" as one
continuous breakdown, implying pre-order was a sub-count of the headline total when it's actually a
wholly separate, earlier pipeline stage not included in `stats.onOrder.count`/`.value` at all.
Removed the "· N pre-order" fragment from the caption entirely (kept "late or no ETA", which IS a
real subset of the total). Also updated this card's tooltip text (deviating from AG-292's literal
Jira wording, per the user's own explicit instruction to "change the details shown on info btn hover
if required") — the old tooltip said "Shows how many ... aren't a placed order yet," which became
inaccurate once that number was removed from the visible card; reworded to explain pre-order stages
exist and are excluded, without claiming the card still shows a count for them.

**Further same-day simplification** — user pointed out the explicit "aren't included" exclusion
clause was itself redundant: the card's own name, "Awaiting delivery," already implies something was
ordered and is now pending arrival, which reasonably excludes Pending/RFQ stages without needing to
say so. Agreed and simplified to: "Parts ordered (PO Created or Parts Ordered) that haven't arrived
yet, with a breakdown of how many are late or missing a delivery date." `tsc`+`next lint` clean.
this is the official, exact-wording follow-up to [[issue-AG-291-overview-kpi-tooltips-hover-color]]'s
just-built tooltips: same 5 KPI cards, same info-icon pattern, but Jira gives precise required text
for each, differing from what I wrote for AG-291 on my own judgment.

**Gate check (developer notes explicitly warn about this): tooltips #2 ("Parts arrived on sold
cars") and #3 ("Overdue fitting") describe AG-286/AG-287's fixed behaviour, not to be shipped until
those land.** Both are already closed this session (AG-286 2026-09-10, bounced+re-closed 2026-09-14;
AG-287 2026-09-11, bounced+re-closed 2026-09-14) — confirmed live in code (`EXCLUDE_POST_HANDOVER_AFTERSALES`
and `OVERDUE_FITTING_IN_STOCK_STATUSES` both already applied in `getPartsInventoryPartsStats`). Gate
is satisfied — safe to ship all 5 immediately.

**Exact required wording** (verbatim from Jira, to replace AG-291's own wording):
1. Received but unfitted: "Parts that arrived but aren't fitted yet, on any vehicle. This includes
   cars already sold, refunded, or cancelled. See 'Parts arrived on sold cars' for that breakdown."
2. Parts arrived on sold cars: "Parts that arrived but aren't fitted, on cars that are Sold,
   Refunded, or Cancelled. Excludes after-sales repair parts, since those are still going onto that
   same car."
3. Overdue fitting (>2 days): "Parts that arrived more than 2 days ago and still aren't fitted, on
   cars currently in stock. This covers Reservation, Deposit, Awaiting Payment, Trade Stock,
   Courtesy Car, and Company Car. Excludes cars that have sold or left the business."
4. Awaiting delivery: "Parts ordered, or still being requested or quoted, that haven't arrived yet.
   Shows how many are late or missing a delivery date, and how many aren't a placed order yet (still
   pending, RFQ created, or RFQ sent)."
5. Consumable stock value: "Total value of consumables in stock, using each product's most recent
   price from a purchase order or, if none exists, a manual stock adjustment. Also flags lines
   critically low on stock and lines with no recorded price, so the real total may be higher than
   shown."

**Related:** [[issue-AG-291-overview-kpi-tooltips-hover-color]] (superseded tooltip wording),
[[issue-AG-286-parts-arrived-sold-cars]], [[issue-AG-287-overdue-fitting-instock-statuses]] (gate
dependencies, both already satisfied), [[issue-AG-277-overview-label-rewording]] (prior label pass
on these same 5 cards, kept separate per the ticket's own note).

---

## HISTORY

- 2026-09-14: Ticket opened by user request, memory file created immediately per
  [[create-ticket-file-immediately-on-open]], before any analysis.
