---
name: answer-vs-edit-scope
description: "When asked to ANSWER a question or DRAFT a reply, stay read-only — don't start editing code unless explicitly asked"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 029f26a0-f640-43df-958a-57216fd0ecf2
---

When the task is to **answer a question, explain, or draft a GitHub reply**, do NOT start changing code —
investigate read-only, surface what I find, and ASK before editing.

**Why:** On #1948 (2026-06-19) Mike asked two questions; while answering Q2 I discovered the editor shares the
load path and immediately started wiring an editor gate across 4 files. User stopped me — "who said to make
changes, i just need the answer of those questions" — and I had to revert it all. The investigation was correct
and useful; making the code change was out of scope and wasted work.

**How to apply:** "answer Mike", "draft a reply", "explain how X works", "what does this do" = read-only. It's
fine (good, even) to *find* a bug or needed change while answering — but report it and ask "want me to fix it?"
rather than editing. Code edits happen only when the user asks for a change. Distinct from
[[check-recent-commit-before-tweaking]] (which is about HOW to start an edit once asked).

**Same rule on RESUME (`/rt #N` / "back to #N").** Resuming = load the NOW block and BRIEF me (phase, branch
+ SHAs, next action), then STOP and ask. Do NOT continue developing just because last session was mid-dev — I
usually resume to ask a *different* question about the ticket, not to pick up the previous task. Added 2026-06-25
after I jumped straight back into coding on a resume. Enforced in MEMORY.md SPEC rule 6 + the `/rt` command.
