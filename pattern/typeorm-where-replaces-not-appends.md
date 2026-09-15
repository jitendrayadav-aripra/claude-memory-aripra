---
name: typeorm-where-replaces-not-appends
description: TypeORM QueryBuilder's .where() replaces the entire WHERE clause rather than appending — a later .where() silently discards any earlier .andWhere()/.orWhere() calls
metadata:
  type: pattern
---

**TypeORM's `QueryBuilder.where(...)` REPLACES the whole WHERE clause** — it does not AND onto
whatever conditions already exist. Only `.andWhere(...)` / `.orWhere(...)` append to the existing
clause. If a function builds up conditions with `.andWhere(...)` and then later calls `.where(...)`
again (e.g. to wrap a fresh set of OR-branches in a `Brackets` group), that later `.where()` **silently
discards every earlier condition** — no error, no warning, the query just runs with fewer WHERE
clauses than the code implies.

**Why to apply:** before adding a new `.andWhere(...)` guard to an existing query-builder function in
this codebase, read the WHOLE function first and check whether it calls `.where(...)` (not
`.andWhere`) anywhere **after** the point you're inserting at. If it does, the guard must go **after**
that `.where(...)` call, not before — otherwise it compiles clean, passes `tsc`, looks correct on
read-through, and does nothing at runtime. This is exactly how AG-295's first attempt at a fix failed
silently: a guard added right after the initial joins in `buildNonConformingBaseQuery`
(`inventory.service.ts`) was wiped out by a `qb.where(new Brackets(...))` call ~70 lines later in the
same function, and the bug looked fixed in code review but wasn't fixed in the running app.

**How to apply:** when adding a condition to any `buildXQuery`-style shared query-builder function in
this file (there are several — `buildNonConformingBaseQuery`, the various `getPartsInventory*`
functions' inline `buildQuery`), grep the function for every `.where(` call first. If there's more
than one, your new `.andWhere(...)` must come after the LAST one, or be added inside whichever
`Brackets` callback is appropriate if it needs to interact with the OR-logic rather than just AND
onto the whole thing. Don't trust that "it's near the top with the joins" is a safe place to add a
condition — verify against the function's actual control flow.
