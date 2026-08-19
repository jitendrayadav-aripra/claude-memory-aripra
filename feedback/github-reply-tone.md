---
name: github-reply-tone
description: "How to write GitLab notes/replies (and dev comms) so they read as authored by the user, not AI"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

When drafting GitLab notes/replies to Mike (or any outward dev comms posted under the user's name),
write in the **user's own voice — natural, human, first-person**. These go out as `ppogra`, so they must
not read as AI-generated.

**Why:** the notes are posted as the user; an obviously-AI tone (over-structured, hedgy, "I can do X
regardless; I'll hold Y") undercuts that. The user explicitly edits replies to sound like they wrote them.

**How to apply:**
- **ALWAYS confirm the draft with the user before posting** anything to GitLab (notes/replies/status,
  assignee changes, label/state moves like To Test). Show the draft + the intended action; post only after
  they OK it. Confirmed standing rule 2026-06-16.
- Default to a human, conversational close — e.g. "Let me know how you'd like to proceed" / "happy to
  start X straight away, just wanted your call on Y first" — NOT robotic "I can land Part 1 now regardless;
  Part 2 I'll hold for your confirmation."
- Keep it first-person and direct; trim AI tells (excessive bold/bullets only when they genuinely help,
  no over-hedging, no needless caveats).
- **Use "we", not "I" — the user works as a team** (flagged 2026-06-25). In outward dev comms/replies, frame
  the work and decisions collaboratively: "we'll finalize the plan", "we need to pick a model", "once we
  decide" — NOT "I'll finalize", "I'll write the plan". Personal asks can stay first-person, but ownership of
  the work is plural by default.
  **EXCEPTION — the investigation/review the user personally did stays "I"** (flagged 2026-06-26 on #1947):
  "I reviewed the logs…", "I pulled the device logs", "I traced it to…", "the key thing I see in the logs". So
  in a log-investigation reply: the analysis you ran = **I**; the product/fix and decisions = **we/our** ("our
  fix", "we'll add a timeout"). Matches the user's own June 17 #1947 reply, which opened "I reviewed the logs —".
- **Use plain, simple English — the words the user actually types.** BANNED phrasings (flagged 2026-06-12 on
  #1935): "One honest note:", "tiny theoretical window", "self-heals", "bulletproof", "Happy to file…". These
  read as marketing/AI. Say it plainly instead, e.g. "There's still a small edge case — if the app is killed
  right after going online before the write goes through. Firebase saves the write and sends it next launch,
  and #1897 already handles it on the other side. Can file it separately if you want." Short, factual, no flourish.
- Be accurate about the code/facts (the user catches errors — e.g. corrected the survey-model claim), but
  phrase findings the way a developer would in a ticket, not like a report.
- Ask the questions that actually block the work; drop filler questions.
- Verify claims against the codebase before asserting them in a note (see [[issue-618-lumashare-storage-cleanup]]
  where an unverified "no survey model" line had to be fixed).
- Standing ticket mechanics (separate): **2026-06-18 repos are on GitHub → use `gh` (posts as `ppogra23`), NOT
  glab** — e.g. `gh issue comment <num> --repo lumasoft-co/lumabooth_ios --body-file -` (see
  [[github-command-cookbook]] gh section). TICKET NUMBERS UNCHANGED. Commits use `ref #<issue>`
  (cross-references on GitHub, does NOT close), no co-author/AI line; bare SHAs auto-link. The tone rules above
  are platform-agnostic — same voice on GitHub.

**GitHub ops now (2026-06-18 migration):** the old `gitlab` skill was GitLab-specific — for `lumabooth_ios` (and
all migrated repos) use **`gh`** directly (see [[github-command-cookbook]] gh section), or a `github`
skill if one has been added to the claude-skills repo ([[claude-skills-repo]]) — check at session
start. Still prefer the **`marketing-screenshots`** skill for #1934-style store screenshot work, and **`translate`**
for localization. (The `gitlab` skill only applies to anything still on GitLab — currently nothing of ours.)

Related: [[prefers-conversational]] (asks questions inline in prose, not the AskUserQuestion picker).
