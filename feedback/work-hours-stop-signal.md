---
name: work-hours-stop-signal
description: "User bills hourly and habitually overworks (8+h, very punctual — see commit history); wants help knowing when a day's work covers a full ~8h so they can STOP and protect personal life. Give an honest 'that's a full day' signal; never pad hours. Hours DERIVED on demand from worklog-daily (no separate file)."
metadata:
  node_type: memory
  type: feedback
---

The user bills **hourly** and is highly punctual/diligent — years of 8+h days (visible in commit history). With
AI the per-hour output rose, but the long-standing habit to *keep working* persists, and they've said they don't
want it eating their personal life.

**How to apply:**
- Hours are a DERIVED VIEW of [[worklog-daily]] — `/hours` estimates them on demand (per-ticket split + overhead);
  there is NO separate hours file (dropped 2026-06-26 to avoid duplicate state — the worklog is the single source
  of truth, same as `/changes`). The user records the number in their own Excel. **Never pad, never suggest
  make-work to hit a number** — the answer to "should I do 2 more hours to justify the bill?" is always NO.
  Bill/track the time actually worked.
- **Proactively give a clear "that's a full day — good place to stop" signal** once the day's logged work
  reasonably covers ~8h. Frame stopping as legitimate and healthy, not slacking — higher AI output means a full
  day's value fits in a day, not that the day should get longer.
- If asked "is it enough hours / should I keep going," answer honestly against the logged work; affirm logging off.
- Integrity check only: confirm the billed time was genuinely worked (not idle). If part was downtime, bill what
  was actually worked — but don't manufacture the gap either way.

**Why:** sustainable work-life balance + honest hourly billing. The user raised this 2026-06-26 ("can't stop
working, it's habit… don't want to mess up personal life"). Surfaced via the `/hours` command.
Related: [[daily-worklog]], [[session-end-backup-memory]].
