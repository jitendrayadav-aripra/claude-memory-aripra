---
name: issue-AG-274-task-card-status-lock-after-resolution
description: AG-274 — Task Card must not allow status edits on parts that already have a resolution outcome (Listed/Resold/Refitted/Scrapped)
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-05.** `tsc` clean both repos, `next lint` clean on all touched
frontend files. User confirmed "it is working fine" before closing. Closed by explicit instruction.

**The bug:** once a non-conforming part is resolved (Listed/Resold/Refitted/Scrapped) via
[[issue-AG-260-resold-resolution-path]]'s Part Returns Audit flow, its origin `status`
(Faulty/Incorrect/Not Required) is supposed to stay permanent — that's the entire reason
`resolution_status` is a separate column. But the Task Card's own status-edit path
(`updateTaskPartStatus`, `task.service.ts:2790`, reached via `PUT /task/part-status/:id` from both
the current `task-v2` UI and the legacy `task/` tab) had zero awareness of `resolutionStatus` — a
part marked Not Required then Resold could still be changed to Used from the Task Card, producing a
contradictory record.

**Scope decisions confirmed by the user:**
1. **Lock applies from the moment ANY `resolutionStatus` is set, including `"Listed"`** — not just
   the three terminal outcomes. User's literal phrasing was "once the resolution status is set,"
   confirmed explicitly when asked to disambiguate Listed-vs-terminal-only.
2. **`deleteTask`'s separate bug deferred to its own ticket, not this one.** Found during
   investigation: `task.service.ts:1929` — `parts.filter((p) => (p.status =
   InventoryPartsStatus.DELETED))` uses `=` instead of a comparison, silently overwriting every
   part's status to Deleted as a side effect of the filter predicate. Same root concern (a resolved
   part's status getting clobbered) but a different trigger (task deletion, not the status dropdown)
   — user said "we will handle the deleteTask related fix later," explicitly out of scope here.
3. **New mid-conversation requirement — show the resolution status "on info":** added an info
   icon + tooltip to the Task Card's existing status badge, not just a modal-level lock, so the
   resolution outcome is visible on the part row itself without opening the status modal.

**What got built:**
- Backend: `updateTaskPartStatus` (`task.service.ts:2790`) throws a `StringError` up front if
  `part.resolutionStatus` is truthy, before any status mutation happens.
- Frontend: `resolutionStatus` added to the frontend `TaskPart` type (`carplanet/src/app/types/task.ts`)
  — the field was already present on the wire (the Task Card's `getTasksList` loads the full
  `TaskPart` entity via `relations: ["parts"]`, no explicit select), just never modelled or read by
  any Task Card component before this.
  `TaskPartsStatusBadge` (`stock-details/task-v2/components/task_parts_status_badge.tsx`) gained an
  optional `resolutionStatus` prop — renders an `InfoOutlinedIcon` + MUI `Tooltip` ("Resolved:
  {status}") next to the badge in both its render branches (the plain-fallback pill used for
  Faulty/Incorrect/Not Required/Used, and the icon-pill used for Arrived/RFQ/Ordered, added there too
  for consistency even though it's practically unreachable for those pre-resolution statuses).
  `AGPartStatusNoteModal` (`.../modals/ag_part_status_note_modal.tsx`) gained a `resolutionStatus`
  prop — when set, the status `<select>` is replaced with an explanatory message
  ("This part has already been marked as {X} — its status can no longer be changed"); the comment
  textarea/thread stays fully functional regardless, since only the `status` field itself is locked,
  not commenting.
  Call site: `create_task_modal_new.tsx:583` (badge) and `:4350` (modal) both now pass
  `part.resolutionStatus` / `statusModalPart.resolutionStatus` through.
- **Legacy Task Card tab (`stock-details/task/`) deliberately NOT touched** — it calls the exact same
  `PUT /task/part-status/:id` endpoint, so the backend guard protects it too; it just surfaces the
  rejection as a generic error toast instead of the friendlier hidden-dropdown treatment. Consistent
  with that tab being phased out in favour of `task-v2` (mirrors the Stock V2 legacy-tab precedent
  elsewhere in this project).

**Ticket description was drafted before any code was written**, per the user's explicit
"explain what you understand → does it make sense → how will you achieve it → ask permission"
sequencing (same pattern as every other ticket this session) — combined AG-260's precise
file:line-citation depth with AG-269's structured "Notes for Designer/Developer/Tester + Acceptance
Criteria" template, since the user asked for the same pattern as both. User then said "yes go ahead
... consider this as ticket AG-274" — title/description were user-supplied as empty going in, filled
by this draft.

**Correction after the ticket was already filed:** the original draft's "Notes for Tester: Not
Applicable" was wrong — copied loosely from AG-269's pattern (a backend-only price-lookup fix with
no real click-through surface) without checking whether it actually applied here. AG-274 has a very
concrete manual test surface (resolve a part via each of the 4 outcomes, confirm the Task Card locks
status + shows the info icon + tooltip, confirm commenting still works, confirm an unresolved part is
unaffected, confirm the legacy tab still rejects server-side even without the friendlier UI). User
caught this by asking directly ("is there nothing to test by tester") rather than accepting the N/A
at face value — **lesson: don't default "Notes for Tester" to N/A just because a sibling ticket in
the same batch had it; check whether THIS ticket has an actual manual-test surface before writing
that field.** Corrected text handed back to the user to swap into the real Jira ticket.

**Related:** [[issue-AG-260-resold-resolution-path]] (the feature whose data integrity this
protects). Documented in
`my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md`
(new "Related ticket — AG-274" section + changelog entry, 2026-09-05).

---

## HISTORY

- 2026-09-05: User raised the gap directly with a concrete example (Not Required → Resold → someone
  changes status to Used from the Task Card = wrong entry). Investigated via an Explore agent to
  find every Task Card code path that writes `taskPart.status` (found `updateTaskPartStatus` as the
  sole live entry point from both Task Card UIs; `addPart`/`deletePart`/`deleteTask` in
  `task.service.ts` also touch `status` but only on creation/pre-arrival paths, except `deleteTask`'s
  separate bug flagged above) and whether `resolutionStatus` was even visible to the frontend (it
  wasn't modelled in `types/task.ts` despite being present on the wire). Presented full analysis +
  the Listed-vs-terminal-only fork + the `deleteTask` finding, drafted a title + AG-260/AG-269-style
  description, and asked permission per the user's explicit instruction not to apply anything first.
  User answered both open questions (lock from Listed onward; defer `deleteTask`) in the same message
  as a new requirement (show resolution status via an info icon), then gave final permission ("yes go
  ahead ... consider this as ticket AG-274"). Built backend guard + 3 frontend pieces, `tsc`/`next
  lint` clean both repos, updated the living project doc.
