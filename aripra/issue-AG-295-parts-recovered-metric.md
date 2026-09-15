---
name: issue-AG-295-parts-recovered-metric
description: AG-295 — new "Parts Recovered" card (Resold + Refitted sub-metrics) on the Overview "Parts Pending Return or Resale" widget, above Scrapped; click-through to a parts table
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-15.** Title/description written to Jira via `editJiraIssue`.

**Requirement, as given:**
- Add a new "Parts Recovered" card to the Overview "Parts Pending Return or Resale" widget, showing
  money actually recovered via Refitted + Resold outcomes (currently no such metric exists).
- Under it, two sub-metric cards: **Resold** (value = the resale price the user enters at resale
  time) and **Refitted** (value = actual invoice price, falling back to PO unit price).
- Clicking either sub-metric opens a parts table with "the related important columns."
- Scrapped metric unchanged (it's a loss, not a recovery) — don't touch it.
- New card positioned **above** Scrapped in the widget's card order.

Analysis in progress — need to verify current `getPartsResolutionBuckets` (doesn't currently return
Resold/Refitted at all, only waitingToBeReturned/readyForSale/alreadyOnSale/unknown/scrapped) and
find where Resold's user-entered resale price is actually stored.

**Related:** [[issue-AG-260-resold-resolution-path]] (built the Resold outcome itself),
[[issue-AG-261-refit-vehicle-task-attachment]] (built the Refitted outcome),
[[issue-AG-285-scrapped-value-total]] (added Scrapped as its own bucket in this same widget — closest
precedent for "a resolution outcome with no existing bucket gets a new unconstrained query"),
[[issue-AG-294-refitted-destination-badge-swap]] (same session, found the `refittedFromTaskPartId`
double-row mechanic this ticket also had to guard against).

**Built 2026-09-14, `tsc`+`next lint` clean both repos.** No migration — all fields already existed.
- **`getPartsResolutionBuckets`** — 2 new separate queries (same bypass pattern as Scrapped, since
  Resold/Refitted are terminal statuses excluded from the main query): `resold`
  (`COUNT`+`SUM(resalePrice)`), `refitted` (`COUNT`+`SUM(COALESCE(invoiceUnitPrice, poUnitPrice))`
  **with `refittedFromTaskPartId IS NULL`** — without this guard every refit would be double-counted,
  since `markTaskPartAsRefitted` sets the same `resolutionStatus = "Refitted"` on both the origin row
  AND a second row it creates on the destination task; caught by re-reading that function rather than
  assuming resolutionStatus alone identifies a unique row).
- **`getPartsInventoryDrilldown`** — 2 new `resolutionStage` bypass cases (`RESOLD`/`REFITTED`,
  same `refittedFromTaskPartId IS NULL` guard on REFITTED). New joins `resoldByUser`/`refittedByUser`
  (both plain int columns, same precedent as the existing `listedByUser` join). New `valueExpr`
  variable overriding the generic `taskPart.poUnitPrice` "value" select — `resalePrice` for RESOLD,
  `COALESCE(invoiceUnitPrice, poUnitPrice)` for REFITTED — applied consistently to the row select,
  `totalValueRaw`, `sortMap.value`, and the default sort fallback, so the drilldown's total and every
  row's £ column always match the money actually recovered, not the original PO price (a deliberate
  decision, confirmed with the user first: every other bucket already follows "the row's £ column is
  whatever that bucket's own aggregate sums").
- **Frontend** — new "Parts Recovered" nested-card wrapper in `overview_tab.tsx` (same 2-column
  pattern as "Parts for Resale"), inserted between Return Window Unknown and Scrapped, green styling
  (`#ECFDF5`/`#047857`/`#A7F3D0`, matching the existing "Refitted" pill elsewhere). 2 new
  `extraColumns` branches in `parts_inventory_drilldown_panel.tsx` (Resold Date/By; Refitted Date/By
  — no separate price column needed since the £ column itself already shows the right value).
- **Deliberately NOT changed**: the widget's proportional bar/legend (`resolutionSegments`) — decided
  with the user that bundling 2 more "recovered" segments into a bar currently scoped to "pending +
  one loss figure" would blend 3 different stories rather than clarify; the new card stands alone as
  a positive counterpoint instead. Scrapped itself also untouched, per the ticket's explicit note.

**2026-09-15 — found + fixed a real pre-existing bug while verifying against live data.** User
compared this ticket's new "Refitted: 3 parts" metric against the Part Returns Audit's own "Refitted"
filter facet, which showed 5 — spotted the Audit page visibly listing both the origin part AND the
synthetic destination-task entry as two separate rows for the same refit. Verified directly against
the local DB (not guessed): `resolution_status = 'Refitted'` = 5 rows total, of which 3 are origin
(`refitted_from_task_part_id IS NULL`, the real distinct refits — matches this ticket's own metric
exactly) and 2 are destination rows. Root cause confirmed in `buildNonConformingBaseQuery`
(`inventory.service.ts:618`, the shared base query for BOTH the Audit's main list AND its filter
facet counts) — its `refittedCount` facet (and the main list itself) filtered only on
`resolutionStatus = 'Refitted'` with no guard against the destination row, unlike this ticket's own
query which already had the guard from the start. Same root cause as AG-294 (both rows share
`resolutionStatus = "Refitted"`), just a different, older piece of code nobody had touched yet.
**Fix**: one unconditional `.andWhere("taskPart.refittedFromTaskPartId IS NULL")` added to
`buildNonConformingBaseQuery` itself — since it's the single shared base, this fixes the duplicate
list row AND the inflated facet count AND any other breakdown built on it (mechanic, parts team,
etc.) in one place. Re-verified against the DB post-fix: facet count now correctly returns 3,
matching AG-295's own metric exactly. `tsc` clean. No frontend change, no migration.

Title/description written into Jira via `editJiraIssue` — covers both the main feature and the
Audit bug fix in one ticket (not split out separately, since it was found during this ticket's own
verification).

**2026-09-15 — first fix silently didn't work; found the real cause.** User reported the Audit page
still showed 5 Refitted parts after the fix. Root cause: my `.andWhere("refittedFromTaskPartId IS
NULL")` was added right after `buildNonConformingBaseQuery`'s initial joins — but the function calls
`qb.where(new Brackets(...))` **later**, and TypeORM's `.where()` replaces the entire WHERE clause
rather than appending to it (only `.andWhere()`/`.orWhere()` append). So the guard was silently
discarded the moment that later `.where()` ran; it never actually took effect despite compiling and
looking correct. **Fix**: moved the same `.andWhere(...)` line to directly after the
`.where(Brackets(...))` block closes instead of before it. Verified no other call site of
`buildNonConformingBaseQuery` (11 total) calls `.where()` again afterward — they all only use
`.andWhere()`/`.select()`/etc. — so this placement is safe everywhere the function is used. `tsc`
clean. Flagged to the user that a backend restart may be needed to see it live, if hot-reload isn't
picking up the change automatically — outside what could be verified from here.

---

## HISTORY

- 2026-09-14: Ticket opened by user request ("new ticket: AG-295"), memory file created immediately
  per [[create-ticket-file-immediately-on-open]], before any analysis.
