---
name: issue-AG-257-alert-checks-tab
description: AG-257 — Parts Inventory Alert Checks tab (original build); resumed 2026-09-11 for a column-audit follow-up on Ready Cars + Late-to-arrive tables
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-14.** Built 2026-09-11 (no prior memory file existed for this ticket
before then — not previously tracked in this store; original build predates this memory store's use
on this project). `tsc` clean both repos + `next lint` clean. No migration.

**Built, same audit source as AG-284/286/287's follow-ups**, resumed together with them. Fetched
full Jira text (chat message was an abbreviated summary) — confirmed against current code before
building:
- **Ready Cars** ("Nothing fitted") — added "Oldest Arrival" date, paired with the existing
  "Longest wait" (`maxDays`) column, same underlying `partArrivedDate`. `getPartsInventoryReadyCars`
  is a `GROUP BY vehicle.id` aggregate query, so this needed `MIN(taskPart.partArrivedDate)` (not a
  raw column select) — formatted London-time server-side (`moment.tz(LONDON_TIME_ZONE).format(DEFAULT_DATE_FORMAT)`,
  matching this file's own `*DateText` convention) rather than sent as a raw Date.
- **Late to arrive** — Technician turned out to be **frontend-only**: confirmed
  `getPartsInventoryLatePipelineList` already joins `task.assignedToUser` and selects it via
  `ALERT_ROW_SELECT`/`mapAlertRow`, just never rendered — added one column, no backend change.
  "Days Overdue" (`DATEDIFF(NOW(), taskPart.partETADate)`) added **alongside** the existing ETA
  column, not replacing it — the ticket's own text pointed at AG-284's date+days-figure pairing as
  the model for this, resolving what would otherwise have been an ambiguous "instead of" reading.
  Non-overdue/no-ETA rows show "—".
- **Follow-up same day** — user asked to rename the breach/gone tables' "Days" column (days since
  arrival) to "Days Since Arrival", matching AG-284's Overview drilldown naming for the same value.
  Trivial header-string + column-width change (90→140, to fit the longer label). `tsc`+`next lint`
  clean.

**Related:** [[issue-AG-278-people-alert-checks-tab-styling]] (restyled this same tab's UI into a
tab bar), [[issue-AG-272-parts-inventory-task-card-redirect]] (row-click behavior in these same
tables), [[issue-AG-284-drilldown-return-window-columns]], [[issue-AG-286-parts-arrived-sold-cars]],
[[issue-AG-287-overdue-fitting-instock-statuses]] (sibling column-audit tickets, same day).

---

## HISTORY

- 2026-09-11: User resumed 3 tickets at once (AG-286, AG-287, AG-257) with new column-addition
  requirements from the same audit source as AG-284's follow-up. AG-257 had no existing memory file
  — created immediately per [[create-ticket-file-immediately-on-open]], before any analysis.
