---
name: issue-AG-273-part-note-panel-resolution-actions
description: AG-273 — Part Returns Audit note panel's 4 resolution actions (Listed/Resold/Refitted/Scrapped) with eligibility gating, documented as a standalone ticket after the fact
metadata:
  type: project
---

## NOW

**Status: DONE (closed) 2026-09-04.** Retroactive documentation ticket, not new build work — the
actual feature was built earlier across [[issue-AG-260-resold-resolution-path]] (Listed/Resold) and
its two follow-on outcomes (AG-261 Refitted, AG-263 Scrapped, both folded into the AG-260 memory
ticket rather than getting their own files). The user asked for a description covering all of
`part_note_panel.tsx`'s resolution-action work specifically (not the whole of AG-260's broader
scope — Overview chart, filter panel, etc.), to file as its own ticket for reference, then marked
that new ticket (AG-273) done immediately once filed.

**What the description covers** (drafted to match AG-260's code-citation depth + AG-269's
structured "Notes for Designer/Developer/Tester + Acceptance Criteria" template, per explicit
request to follow both patterns):
- Four actions, each a dedicated `inventory.service.ts` function, each with its own dedicated
  date/user/note columns (never shared, `status` never overwritten):
  `markTaskPartAsListed` (:3543), `markTaskPartAsResold` (:3570, requires Listed first),
  `markTaskPartAsRefitted` (:3617, blocked server-side for Faulty origin, no Listed requirement —
  two entry points), `markTaskPartAsScrapped` (:3664, requires Listed first, any origin including
  Faulty, note REQUIRED unlike the other three's optional notes).
- Eligibility gating computed client-side in `part_note_panel.tsx`
  (`isFaultyOrigin`/`isActionZone`/`actionOptions`/`isResolved`/`hideSignOffSection`) and enforced
  server-side (client-side is UX only, not the source of truth).
- The explicit action-toggle-row UI mechanism that replaced the earlier implicit
  "infer the action from which field has content" heuristic.
- **One scope call made while drafting:** the user named only 3 actions (Listed/Refitted/"Sold") in
  their request; Scrapped was included anyway since it's built via the identical mechanism in the
  same file — flagged explicitly to the user as an addition they could trim, not silently added.
  User did not object before marking the ticket done, so Scrapped's inclusion stands as accepted.

**No code changed for this ticket** — pure documentation/ticket-filing work, mirroring
[[issue-AG-272-parts-inventory-task-card-redirect]]'s pattern (built-then-retroactively-ticketed),
except here the underlying feature was already built and closed under AG-260 well before this
ticket existed. AG-273 exists only so the note-panel-specific slice of that work has its own
Jira reference, independent of AG-260's broader Overview/filter-panel scope.

**Related:** [[issue-AG-260-resold-resolution-path]] (the actual build), [[issue-AG-272-parts-inventory-task-card-redirect]]
(same "build first, ticket-and-close after" pattern, same session cluster).

---

## HISTORY

- 2026-09-04: User asked for a ticket-description draft covering the note panel's Mark as
  Listed/Refitted/Resold work specifically, in the same pattern as AG-260 and AG-269 — fetched
  AG-269's raw Jira text (not previously in this session's context, only its memory-summarized
  form) to confirm its actual template (bold field labels: Notes for Designer/Developer/Tester +
  Acceptance Criteria — a different, more structured template than AG-260's free-form `##`-headed
  style), then combined both: AG-269's field structure, AG-260's precise file:line citation depth.
  Included Scrapped despite it not being named, flagged as an explicit addition. User filed it as
  the real AG-273, then said to mark it done — closed same message, no code touched.
