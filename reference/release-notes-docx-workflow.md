---
name: release-notes-docx-workflow
description: Where the Parts Leakage/Oversight & Resolution module's client-facing release-notes docx files live, their exact format, and how to build the next one
metadata:
  type: reference
---

**Where the docs live:** `my-docs/projects/parts-and-consumable-inventory-4th-project/`, named
`Parts_Leakage_Oversight_Resolution_Release<N>_v1.docx`. Latest as of 2026-09-14: **Release 3**
(`Parts_Leakage_Oversight_Resolution_Release3_v1.docx`). Earlier: Release 2
(`Parts_Leakage_Oversight_Resolution_Release2_v1.docx`), Release 1 (`..._Release1_v1.docx` /
`..._Release1_v3 (2).docx`). Before trusting "latest = Release N", glob the folder to confirm no
higher-numbered file already exists.

**How they're built:** a Node script per release in the scratchpad
`C:\Users\jiten\AppData\Local\Temp\claude\docxgen\` (e.g. `build_release2_v2.js`,
`build_release3.js`), using the `docx` npm package (already installed there — `node_modules` +
`package.json` present, don't reinstall). **To build the next release**, read the most recent
release's script in full first — copy its exact helper functions (`body`, `bullet`, `h1`, `h2`,
`feature`, `progressItem`) verbatim, only changing the content — rather than reinventing the
structure. Run with `node build_release<N>.js "<absolute output path>.docx"`. After building,
**verify by extracting the docx** (`unzip` into a scratch folder, `grep -o '<w:t[^>]*>[^<]*</w:t>'
word/document.xml | sed -E 's/<[^>]*>//g'`) and read the plain text back — don't just trust that the
script ran without error; confirm every section actually rendered before calling it done.

**Exact document structure (stable across releases, don't deviate without being asked):**
- Title (bold, centered): "Parts Leakage/Oversight & Resolution Module Release Notes"
- Subtitle (italic, centered): "Release N - <2-4 word theme phrases joined by commas/&>"
- **1. Introduction** — 3 paragraphs: what the doc is for, what this release builds on / closes from
  the prior release, and a one-liner on the section format below.
- **2. Release Overview** — bulleted plain-English summary of every change, one bullet per feature,
  closing paragraph.
- **3. Features Included in this Release** — numbered `3.1`, `3.2`, ... one per feature/theme
  (multiple related tickets can combine into one feature write-up — organize by user-facing area,
  not 1:1 with ticket numbers). Each feature is exactly 3 paragraphs: **Previously:** (the gap/bug in
  plain terms, cite "flagged at the end of Release N-1" if it was a carried-over known issue), **Now:**
  (what changed), **Why it matters:** (business value; cross-reference section 5 here if an edge case
  is deliberately left open).
- **4. End-to-End Business Flow** — bullets walking the non-conforming-part lifecycle start to
  finish; update only the specific bullet(s) a new feature touches, keep the rest stable release to
  release so the flow reads consistently over time.
- **5. In Progress - Not Included in This Release** — one `h2` + paragraph per still-open item.
  **Before writing this section, check whether any prior release's "In Progress" item got delivered
  this cycle** — if so, remove it from section 5 (move the delivered part into section 3) but check
  whether only *part* of that item shipped (e.g. Release 2's combined "AI resale pricing + AI cost
  pricing" item only had the resale-pricing half delivered in Release 3 — the cost-pricing half
  stayed in section 5, reworded to drop the now-delivered half). Ends with one italic closing
  paragraph: "Note: none of the items above are in this release...".

**Content-gathering process:** everything shipped since the previous release's docx was finalized —
cross-reference [[memory-architecture]]'s `ARCHIVE.md` for closed tickets dated after the prior
release doc's own build date, not just "everything in memory," since some tickets get built and
archived across multiple release cycles.
