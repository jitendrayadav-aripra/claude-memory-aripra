---
name: issue-AG-297-remove-legacy-waste-chart
description: AG-297 — remove the hidden legacy "Waste run-rate — written-off parts by month" chart (behind SHOW_LEGACY_WASTE_CHART flag) and all its dead frontend+backend code
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-16.** Code work fully complete and verified; retroactive Jira ticket
(user created AG-297 after the work was already done, matching the [[issue-AG-293-resale-recovery-retroactive-doc]]
pattern). `editJiraIssue` write kept failing on an infrastructure error (see below) across 4 attempts —
user pasted the drafted title/description into Jira manually instead.

**What was removed** — the Overview tab's old "Waste run-rate — written-off parts by month" stacked
bar chart (Not Required/Faulty/Incorrect written-off value pivoted by month), which was replaced by
the "Parts Pending Return or Resale" resolution widget ([[issue-AG-285-scrapped-value-total]] /
[[issue-AG-295-parts-recovered-metric]] built on top of that same widget) but deliberately left in the
code behind a `SHOW_LEGACY_WASTE_CHART = false` flag as a fallback, per AG-256/AG-260's own code
comments. User confirmed it's no longer needed.

- `overview_tab.tsx` — removed the `SHOW_LEGACY_WASTE_CHART` flag, the `wasteData` derived variable,
  the entire ~55-line hidden chart JSX block, and the conditional wrapper around the "Parts Pending
  Return or Resale" widget (now renders unconditionally). Removed the now-unused `Legend` import from
  `recharts`. Updated the top-of-file comment.
- `parts_inventory_drilldown_panel.tsx` / `types/inventory.ts` — removed the `"WASTE_MONTH"` drilldown
  bucket option, its `month`/`wasteStatus` params, and the `PartsInventoryWasteMonth` type +
  `wasteByMonth` field.
- `inventory.service.ts` (backend) — removed `getPartsWasteByMonth()` entirely, its call site in
  `getPartsInventoryOverviewStats`, and the `"WASTE_MONTH"` case in `getPartsInventoryDrilldown`'s
  bucket switch.

Confirmed via full repo-wide grep (`WASTE_MONTH|wasteByMonth|getPartsWasteByMonth|
SHOW_LEGACY_WASTE_CHART|PartsInventoryWasteMonth|wasteData|wasteStatus`) across both `carplanet/src`
and `car-planet-backend/server` — zero matches remain. `tsc --noEmit` clean both repos, `next lint`
clean on all 3 changed frontend files. No migration (no schema change). No behaviour change to the
live "Parts Pending Return or Resale" widget.

**Jira write blocked — infrastructure issue, not a payload problem.** 3 `editJiraIssue` attempts on
2026-09-16 all failed:
1. Full title+description (backtick-formatted `tsc`/`next lint`) → generic `"Error POSTing to
   endpoint:"`.
2. Same payload, backticks removed → identical generic error.
3. Minimal isolation test (`{"summary": "..."}` only) → specific error: `"upstream connect error or
   disconnect/reset before headers... Cannot assign requested address|remote address:
   10.255.0.12:8080"` — this is the Jira MCP server's own connection to its upstream failing, not a
   content/schema issue on this end. A `getJiraIssue` read on AG-297 in between succeeded fine
   (ticket exists, summary is a placeholder, status Backlog) — reads work, only the write path is
   down. Retried a 4th time after reporting to user — still the same connection error.

**Drafted title/description (ready to paste in manually if retries keep failing):**

Title: `Remove unused legacy "Waste run-rate" chart and its dead code`

Description: Why (superseded by the Parts Pending Return or Resale widget, kept as a hidden fallback,
no longer needed) + What was removed (the 3 file groups above) + confirmation of the zero-reference
grep + Acceptance Criteria (fully removed, no behaviour change to the live widget, tsc+lint clean both
repos).

---

## HISTORY

- 2026-09-16: User asked to check memory+code for the hidden legacy waste chart, confirmed it's
  no longer required, asked for removal plan + explanation before applying. Investigated full
  footprint (3 frontend files + 1 backend file, no other consumers) via grep, explained plan, got
  "yes go ahead", implemented full removal, verified tsc+lint+grep clean, gave commit messages
  proactively. User then said they'd created Jira ticket AG-297 for it and asked to add
  title/description + mark done — this ticket's memory file created retroactively at that point
  (code work already complete), per [[create-ticket-file-immediately-on-open]] applied as-soon-as-
  known since the ticket number didn't exist earlier. Jira write is failing on an infrastructure
  error unrelated to payload content.
