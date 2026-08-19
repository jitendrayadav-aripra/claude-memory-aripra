---
name: support-ticket-comment-structure
description: Required structure for GitLab comments on log-investigation / support bug tickets
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 85508222-fb04-4af4-b07c-2b4d62f2d874
---

For support / "investigate from logs" tickets (e.g. #1943), the GitLab comment MUST follow this
structure, written in the user's own plain voice (posted as ppogra — see [[github-reply-tone]]):

1. **Open with the version check** — literally start: "I have reviewed the logs and the user is on the
   latest version (X.Y, build NNNN)." ALWAYS confirm the app version/build from the logs first; if it's
   5.2 that's the latest. Anchor every log review on version.
2. **Log lines** — paste the actual relevant log lines as evidence (real timestamps, real text — never
   paraphrased; pull them with `gcloud logging read` scoped by `labels.u` AND `labels.d` device id).
3. **Possible cases** — list the candidate causes, each backed by its own log lines.
4. **More logs** under each case as needed.
5. **If the cause isn't pinned down** — close with: still looking into it / will try to replicate, and
   list the things to confirm with the user.

**Why:** this is the user's standard triage format; it leads with the version (rules out "not on latest
code"), keeps the finding log-driven (not robotic/AI), and is honest when the root cause isn't yet proven.

**How to apply:**
- Use the device id in every log query (`labels.d="<short id>"`), not just the email — events collide.
- Cross-check the log against the CODE before asserting a cause (a misleading log label like LumaBooth's
  "Volume button or camera button clicked" — which is just a shared default-case line for nil/tag-0/tag-(-1)
  senders incl. a single-mode screen tap — can send triage the wrong way; verify the distinguishing logs).
- **Cross-map the user's verbatim log lines → emitting code via [[log-signal-map]]** (the same map
  used for local test logs; a real user's cloud logs are the same LBLogger textPayloads). This pinpoints the
  screen/file/method fast AND sharpens the reply ("this line comes from X.m doing Y"). Append newly-traced lines.
- Don't over-claim: if not proven, say "looking into it / trying to replicate" and list confirm-with-user Qs.
