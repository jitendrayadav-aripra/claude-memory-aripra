---
name: always-provide-commit-message-after-changes
description: proactively give a commit message after every code change, don't wait to be asked
metadata:
  type: feedback
---

After any code change (backend or frontend, however small), proactively provide a short, meaningful
commit message for the affected repo(s) — don't wait for the user to separately ask "provide the
commit message."

**Why:** explicit instruction, given after repeatedly being asked for commit messages one-by-one this
session — the user wants this as the default closing step of every change, not an extra round trip.
**Missed again right after saving this rule** (AG-285's build summary skipped it entirely, user had
to ask again) — saving the memory isn't enough on its own; it has to actually be checked against
before ending a turn that changed code.

**Missed a third time on AUT-3639 (2026-09-29/30):** four code-changing turns in a row ended with no
commit message (rename + created_by, removed_by, Sales Diary "+X" badge, and the batch-2 price hold /
cap / counts). The user had to ask. The failure mode: a long multi-file turn ends with a docs and
summary write-up, and the commit message falls off the end. Fix: write the commit message(s)
**first** in the final response, before the summary, not last.

**How to apply:** keep it short (see [[short-commit-messages]]) — one line, `type: summary (TICKET-ID)`
format, no long bullet-point bodies unless the user asks for more detail. Give it right after
confirming `tsc`/lint are clean, as part of the same turn — not a separate follow-up. Treat "did I
just edit backend or frontend code in this turn?" as a mandatory checklist item before sending the
final response — if yes, the commit message(s) go in that same response, every time, no exceptions
for small/single-line changes.
