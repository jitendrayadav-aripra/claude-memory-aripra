---
name: issue-AG-308-ebay-price-search-final-implementation
description: AG-308 — sub-task of AG-260, documents the ACTUAL final implementation of eBay-based part pricing (daily queue + external API + recoverable-price display), since AG-299's own Jira description only covers the original external-export-API scope
metadata:
  type: project
---

## NOW

**Status: DONE (for now), 2026-09-25 — memory-only close, Jira status untouched per
[[mark-as-done-means-memory-not-jira]].** Created by the user directly in Jira (empty, no
title/description) specifically because [[issue-AG-299-external-parts-price-api|AG-299]]'s own
description doesn't match what was actually finally built — AG-299 only describes the original
external bulk/single-part export API; the daily-scheduled queue, result callback, pricing rule, and
price-display work that came later were never folded into AG-299's own Jira text.

**What this ticket documents** (title + description written via `editJiraIssue`, sub-task shape so
no four-section Story structure needed, per `create-ag-jira` skill §5): the same final feature
already fully detailed in AG-299's own memory file and in
`my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md`'s
"Related ticket — AG-299" section — not duplicating the full build history here, just noting that
AG-308's Jira-facing description now accurately reflects the shipped feature (external API, daily
queue, result callback, pricing rule, 3-surface price display) where AG-299's does not.

**Same parent as AG-299** — AG-260 (Story, "Parts Inventory — Add a 'Resold' resolution path for
non-conforming parts"). Assignee: Jitendra Yadav (unchanged, explicitly re-passed per
`create-ag-jira` skill §8 to avoid an accidental reset on the description edit). Status left at
"Backlog" — never touched, per the standing memory-only-close rule.

**Full technical detail** (architecture pivot history, the operator-precedence bug, staging
cron-gating, the partner's response-wrapper mismatch still pending) is in
[[issue-AG-299-external-parts-price-api]] — AG-308 is the accurately-worded Jira record of that same
work, not a separate build.

---

## HISTORY

- 2026-09-25: User created AG-308 directly in Jira (title/description empty) as a sub-task of
  AG-260, specifically because AG-299's own Jira description had drifted from what was actually
  built. Drafted a summary + direct description (sub-task shape, no four-section headers) covering
  the external API, daily queue, result callback, pricing rule, and the 3 places the price now shows
  — confirmed with the user before writing, then applied via `editJiraIssue`. Verified parent (AG-260)
  and assignee (Jitendra Yadav) both survived the edit. Marked DONE in memory same day, per explicit
  instruction — Jira status left untouched.
