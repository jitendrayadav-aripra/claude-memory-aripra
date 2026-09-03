---
name: no-library-install-without-permission
description: never add/install a new npm package (either repo) without asking first, even when it would be the natural choice for a UI element like a chart
---

Never install or add a new dependency (npm package, any repo) on my own initiative. Always ask
first, even when a library would be the obvious/idiomatic choice for what's being built (e.g. a
charting library for a new graph).

**Why:** raised after building the AG-256/AG-260 "Flagged Parts Pending Resolution" chart — the
right call there was to build the segmented bar with plain divs (no new dependency needed), but the
user wants this as a standing rule regardless of whether a library would technically be justified.

**How to apply:** before running `npm install <pkg>` (either `car-planet-backend` or `carplanet`),
stop and ask — don't just install because it's the obviously-correct tool for the job. If an
existing library already in `package.json` covers the need, keep using that (e.g. Recharts for
actual charts) — this rule is about *adding new* dependencies, not avoiding existing ones.
