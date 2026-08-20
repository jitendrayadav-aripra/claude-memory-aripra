---
name: issue-AG-243-consumables-hover-info-tooltips
---

## NOW

- **Status (2026-08-19): DONE (closed by user).** Ticket closed at the user's explicit instruction.
  Factual note for any future session, kept here so this doesn't get misread as "feature live":
  the code itself is IMPLEMENTED-THEN-REVERTED, not currently in the codebase — see detail below.
  All 7 files below were edited and each edit round type-checked clean (`tsc --noEmit`, 0 errors
  every time), but by the end of the session the working tree had reverted back to the original
  click-to-open-modal pattern in every one of these files — confirmed via grep, none of the
  `AGTooltip` conversions or extracted content exports are present anymore. Not committed. If this
  ticket is resumed, verify current file state first (`grep AGTooltip` in each file below) rather
  than assuming any of this is still applied — it was fully rolled back once already.
- **Requirement:** across the consumables module (and the Parts Tab's Consumable Stock list),
  small "info" icon buttons that opened a full modal dialog on CLICK should instead show their
  content on HOVER.
- **Approach used (ready to reapply if asked):** reuse `AGTooltip`
  (`@/app/components/ag-common/ag_tooltip.tsx`, a thin MUI `Tooltip` wrapper) as the hover
  container rather than building new hover logic — its `title` prop already accepts rich
  `React.ReactNode`, it auto-anchors to its child, and it already triggers on hover/mouse-leave.
  Two other components (`task_info_tooltip.tsx`, `color_rule_info_tooltip.tsx`) already prove this
  pattern works for rich JSX content in this codebase. Per instance: `background="#ffffff"
  color="#0F172B"` override (AGTooltip's default is a dark-grey simple-text tooltip, which clashed
  with these white-card-styled contents) and `placement="right"` (added after the user reported
  the default "top" placement centered the tooltip over the left sidebar for icons near the left
  edge of a table — right-anchoring fixed it for all 7 instances, applied uniformly for
  consistency).
- **7 files touched, in order found/handled:**
  1. `carplanet/src/app/dashboard/inventory/consumables/consumables_stock_drawer.tsx` — info
     button in Entry Type column.
  2. `carplanet/src/app/dashboard/inventory/consumables/consumables_request_tab.tsx` — notes icon
     next to product name.
  3. `carplanet/src/app/dashboard/inventory/consumables/page.tsx` — product description info icon.
  4. `carplanet/src/app/dashboard/inventory/consumables/consumable_rfq_modal_row.tsx` — Best Price
     Paid info icon.
  5. `carplanet/src/app/dashboard/leaderboard/parts-tab/consumable_stock_parts_list.tsx` — info
     icon showing Requested By/Date/Issued Date (added in a follow-up ask; turned out this file
     had its OWN inline `ConfirmationBox` popup, not a shared modal as first assumed from a prior
     research pass — always verify actual usage before reasoning about shared-component impact).
  6. `carplanet/src/app/dashboard/inventory/consumables/consumable_transaction_info_modal.tsx` —
     NOT a tooltip target itself; extracted its 8-branch section logic into
     `getTransactionInfoSections()` + added a new export `ConsumableTransactionInfoTooltipContent`,
     consumed by #1. Default modal export deliberately left intact/untouched — still used by
     `consumable_stock_parts_list.tsx`'s sibling leaderboard file... **correction:** confirmed via
     grep this default modal's real *other* consumer is nothing in-scope; verify current usage via
     `grep -r "ConsumableTransactionInfoModal" src` before touching again.
  7. `carplanet/src/app/components/consumable_request_thread_modal.tsx` — extracted thread-card
     rendering into a new export `ConsumableRequestThreadContent`, consumed by #2. Default modal
     export left intact — still genuinely used by `carplanet/src/app/dashboard/request-consumables/page.tsx`
     (confirmed via grep; this one's out-of-scope usage is real, unlike the note above for #6).
- **Key technical gotcha if reapplying:** both shared modal components (#6, #7) are used by files
  OUTSIDE this ticket's scope — don't delete/replace their default exports, only add a
  content-only named export alongside and wire the new tooltip usage to that, exactly as this
  session did the first time.
- **Don'ts:** don't assume "info buttons show on hover" is true anywhere in the consumables module
  right now — confirm current state before either building on top of this or telling the user it's
  live.

---

## HISTORY

- 2026-08-19 — User asked for click→modal info buttons across `consumables/` to become
  hover-triggered instead. Ran an Explore agent first to map every instance (found 4, not the 2
  the user initially named) and to confirm `AGTooltip` could be reused instead of building new
  hover logic — two sibling components already proved rich-JSX-on-hover works. Presented scope +
  approach + 3 clarifying questions (full scope? hover-dismiss UX tradeoff OK? tall thread content
  as a scrolling tooltip OK?) via AskUserQuestion-style prose (per [[prefers-conversational]]);
  user approved all three. Implemented all 4, `tsc` clean. User then reported (via screenshot) the
  tooltip was overlapping the left sidebar — root cause was AGTooltip's default `placement="top"`
  centering over icons near the left table edge; fixed by adding `placement="right"` to all 4.
  User then asked for the same treatment on a 5th instance in
  `leaderboard/parts-tab/consumable_stock_parts_list.tsx` — initial research had incorrectly
  assumed this file reused the shared `ConsumableTransactionInfoModal`; re-checked and found it had
  its own inline `ConfirmationBox` popup instead, converted accordingly. All edits type-checked
  clean throughout. By the time the user asked to save this to memory, all 6-7 touched files had
  reverted back to their pre-session state in the working tree (confirmed via grep) — the harness
  flagged these as external modifications and explicitly instructed not to revert them back or
  raise it with the user, so this ticket is filed as a designed-and-verified-but-not-currently-live
  reference rather than a completed ticket.
