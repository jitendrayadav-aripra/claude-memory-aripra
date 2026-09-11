---
name: always-update-living-doc
description: proactively update NEW_PARTS_AND_STOCK_INVENTORY.md with every relevant change, don't wait to be asked
metadata:
  type: feedback
---

Whenever a ticket in the Parts Oversight / Non-Conforming / Part Returns Audit domain is built
(and again when it's closed), proactively update
`my-docs/projects/parts-and-consumable-inventory-4th-project/NEW_PARTS_AND_STOCK_INVENTORY.md` —
its own "Related ticket — AG-XXX" section + a changelog entry — as part of that same close-out,
without waiting for a separate "update the doc" instruction.

**Why:** explicit instruction — the user doesn't want the doc's currency to depend on remembering
to ask every time; it should just always be kept in sync as a default step, the same way commit
messages and the ticket's own memory file now are (see [[always-provide-commit-message-after-changes]],
[[create-ticket-file-immediately-on-open]]).

**How to apply:** treat "does this ticket touch Parts Oversight, Alert Checks, People, Consumables,
Overview, or Part Returns Audit / non-conforming-parts resolution?" as a mandatory checklist item
at close-out (same moment the ticket's own memory file gets marked DONE) — if yes, add/update its
"Related ticket" section and changelog entry in the same turn. Scope boundary stays as already
established: skip it for tickets that only touch a general/shared component outside this domain
(e.g. AG-282's Task Card badge work was correctly left out — not part of this module).
