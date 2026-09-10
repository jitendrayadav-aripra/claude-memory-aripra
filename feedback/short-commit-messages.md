---
name: short-commit-messages
description: keep commit messages short and meaningful, not long multi-bullet bodies
metadata:
  type: feedback
---

Commit messages should be short and meaningful — a single concise line (`type: summary (TICKET-ID)`),
not a multi-paragraph body with bullet points explaining every detail of the change.

**Why:** explicit correction after giving verbose multi-bullet commit messages by default — the user
wants commit history scannable, not a re-statement of the whole diff.

**How to apply:** default to one line. Only add a short body (1-2 lines max) if the change genuinely
needs a "why" that isn't obvious from the summary line alone. Works together with
[[always-provide-commit-message-after-changes]] — give it short, and give it proactively.
