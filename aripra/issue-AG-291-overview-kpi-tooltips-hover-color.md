---
name: issue-AG-291-overview-kpi-tooltips-hover-color
description: AG-291 — Parts Oversight row hover color must be #F8FAFF, matching Stock Analysis (Jira rescoped 2026-09-14 to hover-color only; the KPI tooltip work built alongside it now lives under AG-292)
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-14.** Previously closed same day, bounced back same day for the
chart hover-color follow-up below, re-closed same day. Jira title/description rescoped to
hover-color only.

**2026-09-14 further follow-up — extend hover color to the Overview tab's "Ageing of arrived-but-
unused parts" bar chart too**, per a screenshot showing the chart's default Recharts gray hover
highlight looked inconsistent next to the now-`#F8FAFF` table rows. Built: added
`cursor={{ fill: "#F8FAFF" }}` to the chart's `RechartsTooltip` (Recharts' `Tooltip.cursor` prop
controls the highlighted rectangle drawn behind the hovered bar — distinct from `Bar`'s own `cursor`
prop, which is just the CSS mouse-pointer style and was already `"pointer"`). Only applied to the
currently-visible ageing chart — the legacy stacked waste-by-month chart (`SHOW_LEGACY_WASTE_CHART`
flag, kept intact but not rendered) was left untouched since it isn't live and wasn't asked about.
`tsc`+`next lint` clean.
Originally opened with 2 requirements in one ticket; once [[issue-AG-292-kpi-card-hover-details]]
was raised as the official, separately-tracked ticket for the KPI tooltip work (with its own exact
required wording), the user asked to narrow AG-291's actual Jira summary/description back down to
just the hover-color fix, so the two tickets don't overlap in scope. Written into Jira via
`editJiraIssue`:
- **New title:** "UI fix: Row hover color should be #F8FAFF, matching Stock Analysis"
- **New description:** explains the inconsistency + the `hover:false`+`!important` fix, with an AC
  of `#F8FAFF` consistently across every Parts Oversight table.

The KPI-tooltip code changes described below were still built as part of this ticket's actual work
(before the rescoping) — left in this file's history as an accurate record of what happened, but
the *ticket itself* (title/description/AC) no longer claims that scope; AG-292 is now the ticket of
record for it.

**Original 2 requirements, given verbatim in chat 2026-09-14 (before Jira had any description):**
1. Row hover color must be `#F8FAFF` — already implemented as a standalone (non-ticketed) fix just
   before this ticket was opened, across all 6 Parts Oversight tables (Alert Checks ×2, People ×2,
   Overview drilldown, Consumables), matching Stock List v2's `hover:false` + `!important` pattern.
2. The 5 Overview tab KPI cards need to show what each one includes/excludes in its calculation, so
   users understand the basis for the number shown — not yet analyzed against current code.

**Built 2026-09-14, `tsc`+`next lint` clean.** No backend change — all 5 numbers already existed,
this only explains them. Reused the exact info-icon pattern already established in
`task_parts_status_badge.tsx` (AG-270/274/282): MUI `Tooltip` + `InfoOutlinedIcon`
(`fontSize: 14, color: "#9CA3AF"`), added as a local `KpiInfoTooltip` helper in `overview_tab.tsx`,
with `stopPropagation` on the icon so hovering/tapping it doesn't trigger the card's own
click-through navigation. Had to rename recharts' own `Tooltip` import to `RechartsTooltip` to avoid
a naming collision with MUI's `Tooltip` in the same file (2 existing chart usages updated, no
behavior change).

Tooltip text per card, each verified against the actual backend query (not guessed) before writing:
- **Received but unfitted** — no vehicle-status filter at all; includes sold/gone cars too, not just
  in-stock — called out explicitly since it's easy to assume otherwise.
- **Parts arrived on sold cars** — excludes genuine post-handover Aftersales Request parts (AG-286).
- **Overdue fitting (>2 days)** — the 7-status "physically in stock" list (AG-287), arrived >2 days.
- **Awaiting delivery** — PO Created/Parts Ordered only; the caption's "pre-order" count is
  informational and NOT included in the headline value (confirmed via `getPartsInventoryPartsStats`
  — `onOrderValue` and `preOrderCount` are separate SUMs over different status lists).
- **Consumable stock value** — quantity × last resolvable price (AG-269's Invoice > PO > manual
  adjustment priority); no-price lines excluded from the total, counted separately — matches the
  "≥" prefix already shown.

**2026-09-14 — superseded by [[issue-AG-292-kpi-card-hover-details]].** A follow-up ticket gave
exact, official Jira wording for all 5 tooltips (differing in places from what was written here on
my own judgment) — all 5 `title` strings replaced verbatim, component/structure unchanged. This
ticket's own analysis above (the include/exclude facts) is still accurate and was the basis AG-292's
wording matched against; only the literal tooltip text changed.

**Related:** [[issue-AG-277-overview-label-rewording]] (same 5 KPI cards, prior copy pass),
[[issue-AG-270-ai-resale-price-research]] (established a tooltip pattern on a related card).

---

## HISTORY

- 2026-09-14: Ticket opened by user request ("new ticket is AG-291"), memory file created
  immediately per [[create-ticket-file-immediately-on-open]], before any analysis. No Jira
  description exists yet — title/description to be drafted and proposed to the user.
