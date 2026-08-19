---
name: prefers-conversational
description: User prefers plain conversational questions over the multiple-choice AskUserQuestion tool
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

User consistently rejects the `AskUserQuestion` multiple-choice tool (rejected it
~4 times in one session), choosing "let me clarify" each time, then answers in their
own words.

**Why:** they want to add their own context/nuance, not pick from pre-set options —
they often have information that reframes the choice (e.g. "I connected the M50",
"the M badge is clipping").

**How to apply:** ask follow-up questions inline in plain prose with a clear
recommendation; do NOT use AskUserQuestion for routine next-step/approach decisions.
Reserve structured prompts for genuinely binary, fully-specified forks if ever.
