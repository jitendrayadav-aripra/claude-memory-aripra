---
name: dont-default-ticket-fields-to-na
description: When drafting a ticket description in this project's Notes for Designer/Developer/Tester template, don't copy "Not Applicable" from a sibling ticket without checking whether THIS ticket actually has that surface
metadata:
  type: feedback
---

Don't default a template field (especially "Notes for Tester") to "Not Applicable" just because a
sibling ticket drafted in the same session/batch had it that way. Check whether *this specific*
ticket has a real surface for that field before writing N/A.

**Why:** Drafted AG-274's description by loosely reusing AG-269's structure (bold-labeled
Designer/Developer/Tester/Acceptance-Criteria template, fetched from AG-269's real Jira text).
AG-269 legitimately has "Notes for Tester: Not Applicable" — it's a backend-only price-lookup fix
with no click-through UI to test. AG-274 is NOT that kind of ticket — it has a very concrete manual
test path (resolve a part via each of 4 outcomes, confirm the Task Card locks the status dropdown +
shows an info icon/tooltip, confirm commenting still works, confirm an unresolved part is
unaffected, confirm the legacy tab still rejects server-side). Copying the N/A across without
re-checking meant the user had to catch it themselves ("is there nothing to test by tester") instead
of it being right the first time.

**How to apply:** Before writing "Not Applicable" in any ticket-description field (Designer/
Developer/Tester notes, or similar boilerplate sections), ask: does *this* ticket have real content
for this field, independent of what a template or sibling ticket had? For "Notes for Tester"
specifically — if the change has any UI a person could click through, or any behavior a person could
manually verify (not just a code-review-only backend logic change), write the actual manual test
steps instead of N/A.
