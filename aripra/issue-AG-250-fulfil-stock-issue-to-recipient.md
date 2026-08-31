---
name: issue-AG-250-fulfil-stock-issue-to-recipient
---

## NOW

- **Status (2026-08-25): DONE — closed in Jira** (transition "Close" → status Closed,
  statusCategory Done). Comment posted on AG-250 documenting the fix. No action pending on me.
- **Ticket:** `[Parts] Fulfil-from-stock: add "Issue to" recipient selection (currently defaults
  to task creator)` — reported by Ahmad, R4 testing, 2026-08-24. Fulfilling a part from stock
  always showed "issued to" as whoever created the task (e.g. Bhavin) — the flow never asked who
  to actually issue it to.
- **Root cause:** `check_consumable_stock_modal.tsx`'s direct "Fulfil from Stock" call sent only
  `{ consumableProductId }` — no recipient field existed anywhere in that path.
  `requestedByUserId` (correctly `task_part.created_by`, the original requester) was the only
  person-like field available, so anything reading "who was this issued to" fell back to it —
  not a bug in that field itself, just a missing separate recipient.
- **Fix:**
  - `check_consumable_stock_modal.tsx` — added a required Team → User picker (reusing
    `DIRECT_ISSUE_ALLOWED_TEAMS`/`FieldRow`/`selectStyles` already exported from
    `consumable_direct_issue_modal.tsx`), shown for the direct-fulfil path; selected technician
    sent as `directIssuedTo`.
  - `inventory.service.ts` `fulfilTaskPartFromConsumableStock` — accepts `directIssuedTo`, passes
    it to `consumableService.issueConsumableRequest`, which sets it on the transaction.
    `actionByUserId`/`issuedByUserId` (approver/issuer) stay the current fulfilling user, unchanged.
  - `consumables_stock_drawer.tsx` — the transaction history's "Issued To" column previously only
    read `issuedToUser` for `entryMethod === "Direct Issue"` rows, always falling back to
    `requestedByUser` for Request-origin rows regardless of whether a real recipient had been
    captured. Now prefers `issuedToUser` (from `directIssuedTo`) whenever set, falling back to
    `requestedByUser` only when no distinct collector was ever recorded. This is what actually
    surfaces the fix in the product's stock history — the backend already resolved
    `issuedToUser` correctly, the frontend column was the last mile that ignored it.
  - `fulfilTaskPartFromConsumableStockValidation` — `directIssuedTo` optional at the schema layer
    (required by the UI), same convention as the existing Direct Issue modal.
- **Shipped in the same session as a broader feature:** an approval-gate system for consumable
  products (`requires_approval` flag → separate Approve → Issue flow with its own recipient
  picker in the Consumables Requests tab's Task Requests toggle). AG-250's fix reuses the same
  `directIssuedTo` plumbing but is independently applicable — see
  `carplanet/tasks/todo.md` at the projects root for the full feature writeup if that broader
  context is ever needed.
- **Verified:** `npx tsc --noEmit` clean on both `carplanet` and `car-planet-backend`. NOT
  manually clicked through in the running app this session — flagged to the user as worth a
  quick R4 pass to confirm the recipient shows correctly end-to-end.

---

## HISTORY

- 2026-08-25 — Fetched AG-250 from Jira (`aripra.atlassian.net`, cloudId
  `761f8a59-91ff-493b-ab27-a38827186c27`) to confirm scope against work already done in this
  session (the Team/User picker had just been added to `check_consumable_stock_modal.tsx` for the
  direct-fulfil path per a separate user request, plus the "Issued To" column bug in
  `consumables_stock_drawer.tsx` had just been fixed a couple of turns earlier). Confirmed the
  ticket's description matched exactly what was already implemented. Posted a comment on AG-250
  summarizing the fix (frontend/backend file list, what changed, what stayed the same) and
  transitioned it via the "Close" transition (id 13) — the only status-category-Done transition
  available on this workflow from Backlog (no separate "Done" status, just Backlog → Selected for
  Development → Close).
