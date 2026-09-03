---
name: issue-AUT-3661-po-status-rollup
description: AUT-3661 — fix PO status stuck at "Parts Ordered" once all parts have arrived (backend rollup logic, car-planet-backend)
metadata:
  type: project
---

## NOW

**Status: DONE, 2026-09-01.** User tested own exact repro in the running app after the fix and
confirmed it works. No frontend changes were needed — no migration/schema change either.

**Root cause (3 distinct gaps in `car-planet-backend`, not just one):**
1. `PO.populateJsonFields()` (`server/entities/inventory/po.entity.ts`) only recognized 3 of the 12
   `InventoryPartsStatus` values (PO Created/Parts Ordered/Parts Arrived) when rolling up the PO's
   overall status. Once every sibling part moved past "Parts Arrived" into a post-arrival
   disposition (Used/Faulty/Returned/Not Required/Incorrect), the loop matched no branch and
   silently left `po.status` frozen at its last value.
2. `deletePOPart` (`server/services/inventory/inventory.service.ts`) — deleting a part off a PO via
   the PO detail screen (`po_section_view.tsx`) never recalculated/saved the PO's status at all.
   **This was the actual trigger in the user's real repro**: 2 parts arrived + marked Used, 3rd
   still-"Parts Ordered" part deleted from the PO screen → PO frozen at "Parts Ordered" forever.
3. `updateTaskPartStatus` (`server/services/task/task.service.ts`) — the action that moves a part to
   Used/Faulty/Returned/Not Required/Incorrect (or reverts it to Parts Arrived) also never triggered
   any PO recheck. Real gap, but not the user's actual trigger.

**Investigation method worth remembering:** the obvious/simple theory (populateJsonFields' missing
branches alone) looked right until the user manually reproduced the "fix" scenario in their own
dev environment and got the CORRECT result — directly contradicting the theory. Traced the real
code line-by-line from that contradiction (not the agent-summarized version) and found
`markTaskPartAsReceived` always guarantees the just-arrived part registers as index 5 at rollup
time, so single/bulk check-in alone can never reproduce the bug — check-in-based repros will
always look fine. The real trigger only surfaced when the user described their exact click-by-click
scenario, which led to `deletePOPart` — a completely different function than either of us had been
looking at. See [[feedback_verify_branch_before_continuing]]-style lesson: verify against the user's
own concrete counter-example before trusting a plausible-sounding root cause, especially one only
validated by a subagent's summary rather than direct code reading.

**Fix:**
- `po.entity.ts`: `populateJsonFields()` excludes `Deleted`-status parts, then treats "Arrived or
  anything past arrival" (Arrived/Returned/Not Required/Used/Faulty/Incorrect) as "this part is
  done" — not just the literal word "Arrived". Falls through to the original min-index logic
  unchanged for the pre-arrival case.
- `inventory.service.ts`: `deletePOPart` now calls `populateJsonFields()` + `PORepository.save()`
  on the PO after unlinking the deleted part, before returning it.
- `task.service.ts`: `updateTaskPartStatus` now loads the `po` relation and re-runs the same
  recompute+save after any part status change.
- Dead/no-op block inside `getPO()` (`inventory.service.ts:2317-2340`, compares `po.status` to
  itself, provably inert) — **left untouched, user's explicit call**, not because it mattered.

**Explicitly out of scope (user's call, flagged not silently dropped):** `deleteTaskPart`
(task.service.ts — the Task Card's own separate delete path, distinct from `deletePOPart`) has the
same missing-rollup gap. Not fixed this round.

**Side effects confirmed safe via a dedicated Explore-agent side-effect audit** (all 8 callers of
`populateJsonFields()`, all frontend consumers of both changed endpoints, read fully not
grep-sampled):
- `partVRMs` (PO list search + column) now drops a VRM once its only part on that PO is Deleted —
  intended, not a regression.
- `deletePOPart`'s response can now correctly reveal a "Delete PO" button in `po_section_view.tsx`
  that was effectively unreachable before (gated on `status === PO_CREATED`, which was frozen) —
  safe, the actual PO-delete backend independently re-validates no parts are ordered/arrived.
- `updateTaskPartStatus`'s response now includes a partial `po` field it previously lacked — this
  actually **fixes** a pre-existing latent bug: every status update was silently wiping the part's
  `po` info in frontend local state until next reload (frontend already expects/declares a `po`
  field; every consumer already guards with optional chaining, so the still-missing nested
  `supplier`/`poCreatedBy` sub-fields don't crash anything).

**Refs:** Plan/detail log in `tasks/todo.md` (repo root, top section as of 2026-09-01).

---

## HISTORY

- 2026-09-01: User pasted ticket description (carplanet Jira site not authorized for this session's
  Atlassian connector — user chose to paste text directly rather than grant access). Read the actual
  consumables + parts PO frontend/backend files (not a stale unrelated memory) to ground the
  explanation, per user's own redirect early in the session.
- 2026-09-01: First root-cause pass (populateJsonFields' missing branches) looked plausible from an
  Explore-agent report, but the user's own manual repro directly contradicted it — retraced the
  actual code by hand rather than trusting the summary, confirmed check-in-based repros can't
  reproduce the bug given how `markTaskPartAsReceived` guarantees index-5 at rollup time.
- 2026-09-01: User described their exact click-by-click repro (arrive+Used two parts, delete the
  3rd still-ordered part from the PO screen) — this pointed to `deletePOPart`, a function neither
  the first investigation nor the user's own test had touched. Confirmed via direct read: it never
  saves any recomputed PO status at all.
- 2026-09-01: Implemented all 3 fixes, `npx tsc --noEmit` clean. Ran a dedicated Explore-agent
  side-effect audit across every caller of the 3 changed functions before calling it done, per
  user's explicit ask to confirm no other flow was affected — found 3 intended/beneficial side
  effects, zero regressions, one pre-existing unrelated gap (`deleteTaskPart`) surfaced and
  deliberately left out of scope.
- 2026-09-01: User tested own exact repro in the running app, confirmed working, marked done.
