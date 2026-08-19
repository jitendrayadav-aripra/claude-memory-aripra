---
name: commit-message-format
description: "How to write git commit messages in this user's repos — conventional-commits subject + the durable `ref #<num>` footer; preserves past+current best practice"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

The user's commit-message convention, derived from 10 years of history (lumabooth_ios, 2016→2026)
and confirmed as the form to use going forward. Goal: keep the best of past + current, don't regress.

**Why:** user asked to codify the best commit practice they've built over 3.5→current so future work
doesn't lose it (2026-06-15).

## The durable invariant (NEVER drop)
- Every commit ends with a ` ref #<num>` footer pointing at the GitLab work-item. Used in ~95%+ of
  commits every single year 2016–2026.
- NEVER use `Closes #` / `Fixes #` / `Resolves #` — zero occurrences in the entire history. Always
  `ref #<num>`, and the issue is closed manually in GitLab, not by the commit.

## Only `#`-reference the work's OWN ticket (confirmed 2026-06-16)
- The `ref #<num>` footer points at the ONE ticket the commit belongs to. Any OTHER ticket mentioned in
  the subject/body (a related, prior, or especially a CLOSED ticket) must be written as a PLAIN number
  (no `#`) — e.g. "preserve behavior from 1797", "per 1836". A `#<num>` anywhere creates a GitLab
  cross-reference and posts a "mentioned in commit" back-link on that other ticket, which is noise (and
  worse on a closed ticket — wrong association / reopen-style churn).
- Same rule for GitLab notes/replies: only the ticket you're posting on gets `#`; reference others plain.
  (We hit this on #1823 — wrote 1836/1797/1929 plain so the plan didn't back-post to those.)

## Current best form (2026+, use this)
`type(scope): imperative summary  ref #<num>`
- Example: `fix(green-screen): show both background-carousel arrows + theme them  ref #1941`
- **type** — one of: `fix`, `feat`, `chore`, `docs`, `refactor`, `revert`, `spike` (spike = exploratory
  branch work). Stick to this set.
- **scope** — kebab-case feature area: `ai-portrait`, `webcam`, `mac-screenshots`, `color-picker`,
  `snapshot`, `event-copy`, `print-limits`, `welcome-screen`, `sync`, etc.
- **summary** — imperative mood, lower-case, no trailing period.

## Guardrails (drift seen in the real history — avoid)
- Don't typo the type (`ix:` appeared 2×). 
- Don't invent types (`refine:` appeared 14× — use `refactor` or `fix`).
- Pick ONE canonical scope per feature — history has `ai-portrait` vs `ai-portraits` split. Reuse the
  existing scope spelling rather than coining a variant.

## Preserve the best of the OLD style
Pre-2026 messages were long descriptive sentences that explained ROOT CAUSE / "couldn't replicate but
per the logs…". Keep that depth — don't lose explanation to a terse subject. For any non-trivial fix,
put the conventional subject on line 1, blank line, then a BODY explaining the why / root cause.
(Mirrors the user's support-ticket comment structure.)

## Body formatting (confirmed 2026-06-16)
- **Bullet points over long prose** — when the body covers more than one point, use a bulleted list,
  not one long run-on paragraph. Short single-point bodies can stay a sentence.
- **Method names in single quotes** — wrap any code symbol (method/function/selector) in single quotes,
  e.g. `'imageTaken:'`, `'didReceiveLiveViewImage:'`, `'flipCapturedImageIfNeededAtPath:'`. Keeps them
  legible in plain-text git log (do NOT use backticks in commit bodies).

## Related conventions
- [[commit-discipline]] — the behaviour around committing: confirm-before-commit, no Co-Authored-By/Claude
  footer (body + `ref #num` only), no internal Part-numbering, and a revert/removal commit cites the prior
  commit's short hash (`revert(scope): …  ref #num` or git-native `Revert "…"`).
