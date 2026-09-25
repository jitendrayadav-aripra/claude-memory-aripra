---
name: issue-AG-309-consumable-product-merge
---

## NOW

- **Status (2026-09-25): DONE (analysis phase, closed at user's request).** Paused pending the
  client's list of known duplicates + Maria's priority call — user will pick this back up per
  [[mark-as-done-means-memory-not-jira]] (memory-only close, no Jira transition made).
- **Requirement:** client flagged duplicate rows in the Stock Catalogue — same physical consumable
  product listed twice with split `quantity`/transaction history. No title/description existed on
  the Jira ticket yet; this was pure requirement + impact analysis, no code/schema touched.
- **Two duplicate patterns confirmed from client screenshots:** (1) same name, one row with
  blank/mangled fields — root cause is the one-time `inventory_stock → consumable_product` data
  migration (`1781136001000-copy_consumable_data.ts`), which copied legacy free-text `sku` before
  `oem_number`/`shelf_location` existed as separate columns. (2) **different names, same SKU**
  ("Mercedes Wear Sensor" / "Brake Pad Wear Sensor", both `DWSPWS7051`) — proves name cannot be
  trusted as the identity key; SKU is more reliable but still just a free-text varchar with no
  uniqueness constraint.
- **Key schema facts (car-planet-backend, `entities/consumable/`):** `consumable_product.quantity`
  is a persisted running counter, never derived by summing `consumable_transaction`. **No real
  DB-level FK constraints exist anywhere in this module** — confirmed directly in
  `migrations/1781136000000-create_consumable_tables.ts` (explicit comments: "plain int — project
  convention", "No FK constraints exist"). `@ManyToOne`/`@JoinColumn` on the TypeORM entities is
  ORM-level only for joins, not enforced by MySQL. Three places reference a product:
  `consumable_transaction` (all history + the mechanic-request lifecycle — no separate requests
  table), `consumable_procurement` (RFQ/PO pipeline), and the looser
  `task_part_request_v2.consumable_product_id` (AG-236/237 "fulfil from Consumable Stock" flow).
  `checkDuplicateConsumableProduct` (`consumable.service.ts:1933-1960`, AG-268) matches on name
  first, SKU only as a same-name tiebreaker — misses the Wear Sensor case entirely; UI-advisory
  only, skippable, `createConsumableProduct` has no server-side dup check. No merge function/route
  exists anywhere in the module.
- **Recommended direction (not yet approved for build):** "hide + sum + read-only history tab" —
  soft-link the loser row via a new `mergedIntoProductId`-style field (don't overload `status`,
  which already means Active/Inactive/Deleted), hide it from catalogue/pickers, sum quantities into
  the survivor (show both source numbers, don't sum silently), add a read-only tab reusing the
  existing `getConsumableProduct(id)` fetch to browse the loser's frozen history. Deliberately
  **not** re-pointing FKs across the 3 tables above — avoids nearly all in-flight-state risk (open
  POs, Pending requests) since nothing about the loser's own record changes underneath it. Open
  design questions before build: exact scope of "hidden" (catalogue only vs. also RFQ picker/KPI
  aggregation), what happens if in-flight activity on the loser completes post-merge (route to
  survivor or not), one-off cleanup vs. reusable feature.
- **Next action when resumed:** get the client's confirmed duplicate list (name-only detection is
  not reliable enough to auto-generate this) and Maria's priority call; then lock the open design
  questions above before writing any code. Full 8-step requirement/impact write-up (business
  goal/flows, gap analysis, task breakdown, edge cases, stakeholder questions) lives in the
  conversation transcript this ticket was opened in, not duplicated here — re-derive only if that
  transcript isn't available.

---

## HISTORY

- 2026-09-25 — User described the client-reported duplicate-row problem with 3 screenshots (no
  Jira title/description yet), asked for a structured requirement/impact analysis (explicit
  8-step format) with a hard "ask before implementing" constraint. Ran an Explore subagent to map
  the full consumable schema/service/frontend rather than reading files directly in-session,
  delivered the 8-step analysis. User pushed on two points that changed the analysis materially:
  (1) duplicates can have **different names** with the same SKU (Mercedes/Brake Pad Wear Sensor
  screenshots) — broke the assumption that name-based matching was even a partial solution; (2)
  asked directly "do we even use FK constraints" — verified against the actual creation migration
  and confirmed no DB-level FK exists anywhere in the module (only TypeORM-level relation mapping),
  correcting my own imprecise "real FK" phrasing from earlier in the conversation. User then
  proposed the "hide + sum + read-only tab" method directly; evaluated it in depth (favorably,
  vs. my own earlier "true re-pointing merge" option) and flagged 4 remaining open decisions (see
  NOW). Drafted and corrected a client-facing status update message (fixed an inaccurate claim
  that deletion is technically blocked — it isn't, soft-delete exists; the real risk is
  disconnected history + orphaned in-flight references). User then said "mark AG-309 as done", I
  incorrectly asked via AskUserQuestion whether that meant transitioning the real Jira ticket —
  user pointed out this was already an established convention (violates
  [[prefers-conversational]] too) and flagged the extra tokens spent. **Also discovered this
  session: I had been writing memory to the harness's built-in auto-memory folder
  (`C:\Users\jiten\.claude\projects\d--aripra-projects\memory\`) instead of this store
  (`claude-memory-core-store`) for the earlier part of this conversation — [[mark-as-done-means-memory-not-jira]]
  already existed here, so the duplicate I wrote there was pure redundancy from not reading the
  right store first. Not deleted (uncertain whether the harness auto-memory folder is
  independently relied upon), but this ticket + the correct store are now the source of truth.**
