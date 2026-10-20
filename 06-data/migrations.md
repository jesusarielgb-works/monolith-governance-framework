# 06 — Migrations

> [!NOTE] INSTRUCTIONS
> One version history for the whole database, even though modules own
> separate schemas. Fill in the rollback policy before the second migration
> ships. Delete this block once a rollback has been rehearsed once.

## One sequence for the whole application

There is one database, so there is one ordered changelog — never a
changelog per module. A changeset touching `catalog`'s schema and one
touching `billing`'s schema still apply in the same numbered sequence,
under `resources/db/migration/` (see
[`module-structure.md`](../05-architecture/module-structure.md)).

| Rule | Detail |
|---|---|
| Tool | Liquibase |
| Sequence | One changelog, one increasing version number, for the entire database |
| Changeset scope | One logical change per changeset; never edit a changeset already run anywhere |
| Module tag | The owning module is the segment after the sequence number in the changeset id, e.g. `2026-01-15-003-billing-add-invoice-lines` ([liquibase-migration-standards](https://github.com/jesusarielgb-works/liquibase-migration-standards/blob/v1.1.0/docs/01-naming-conventions.md)) — for traceability only; it does not split the sequence |

## Rollback policy

| Change type | Rule |
|---|---|
| Additive (new nullable column, new table, new index) | Ships with an automatic or trivial rollback |
| Destructive (drop column, drop table, rename) | Two phases: deploy N adds the replacement and stops using the old shape in code; deploy N+1 drops it [CASE-005] |
| Any changeset | Must define a working `rollback`, verified in CI before merge — a changeset with no rollback is incomplete |

## Mechanics

Changeset syntax, file layout and rollback authoring for Liquibase are not
repeated here — see
[liquibase-migration-standards](https://github.com/jesusarielgb-works/liquibase-migration-standards)
for the full standard. This document is the monolith-level policy that
standard's changesets must satisfy: one sequence, additive by default,
two-phase for anything destructive.

---

**Evidence:** [CASE-005](https://github.com/jesusarielgb-works/engineering-casebook/blob/v1.0.0/records/cases/CASE-005-stripe-online-migrations-2017.md) — Stripe moved hundreds of millions of records by adding the new shape first and dropping the old one last.

**Related:** [`./database-conventions.md`](./database-conventions.md) · [`../05-architecture/module-structure.md`](../05-architecture/module-structure.md)
