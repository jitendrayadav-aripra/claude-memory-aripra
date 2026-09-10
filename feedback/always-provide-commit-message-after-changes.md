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

**How to apply:** keep it short (see [[short-commit-messages]]) — one line, `type: summary (TICKET-ID)`
format, no long bullet-point bodies unless the user asks for more detail. Give it right after
confirming `tsc`/lint are clean, as part of the same turn — not a separate follow-up.
