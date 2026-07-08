# SQL vs NoSQL (DBMS interview angle)

> **Tag: Theory/recall** — full system-design treatment in `System_Design/Building_Blocks/04_SQL_VS_NOSQL.md`; this file is the rapid-fire comparison version for DBMS rounds.

## The Core Table

| | SQL (RDBMS) | NoSQL |
|---|---|---|
| Data model | Tables, rows, fixed schema | KV / document / wide-column / graph |
| Schema | Schema-on-write (enforced) | Schema-on-read (flexible) |
| Transactions | Full ACID, multi-row | Mostly single-item; BASE otherwise |
| Joins | Native | Denormalize instead |
| Scaling | Vertical + read replicas; manual sharding | Horizontal by design |
| Consistency | Strong | Often eventual/tunable |
| Query language | Standard SQL | Per-product APIs |
| Examples | Postgres, MySQL, Oracle | Redis, MongoDB, Cassandra, DynamoDB, Neo4j |

## The Four NoSQL Families (one-liner each)

- **Key-value** (Redis, DynamoDB): hash-map-as-a-service; caching, sessions, simple entities at scale.
- **Document** (MongoDB): JSON per record, nested, indexed; entity-centric apps with variable fields.
- **Wide-column** (Cassandra, HBase): partition key + sorted rows; write-heavy time-series/feeds.
- **Graph** (Neo4j): nodes+edges; traversal-heavy queries (fraud rings, recommendations) where SQL would be 7 self-joins.

## ACID vs BASE

BASE = **B**asically **A**vailable, **S**oft state, **E**ventually consistent — the deliberate relaxation NoSQL makes to win availability/partition tolerance (CAP — see `09_CAP_IN_DBMS.md`). Not "no guarantees": e.g., DynamoDB offers per-item atomicity + optional transactions; MongoDB has multi-document transactions now — the boundary is blurring, and saying so shows currency.

## When Which (the judgment answer)

Choose SQL when: relational data, multi-row invariants (money, inventory), ad-hoc queries/reporting, unknown future access patterns — the default for most applications.
Choose NoSQL when: horizontal write scale beyond one node, simple known access patterns, flexible per-record shapes, or a specialized model (graph, cache).
The interview-safe line: **model and access patterns first, engine second; most systems use both** (Postgres for orders + Redis for sessions + Elasticsearch for search = polyglot persistence).

## Most-Asked Interview Questions

1. **SQL vs NoSQL differences?** → the core table; lead with schema + transactions + scaling.
2. **What is BASE? vs ACID?** → expansion + the CAP motivation.
3. **Types of NoSQL databases + a use case each?** → four families.
4. **Why do NoSQL DBs avoid joins?** Data spread across nodes → cross-machine joins are scatter-gather slow → denormalize into read-shaped records instead.
5. **Is MongoDB ACID?** Single-document always; multi-document transactions since 4.0 (with performance cost) — nuance scores.
6. **Can MySQL scale horizontally?** Yes with manual sharding (see System Design) — it's operational pain, not impossibility, that pushes NoSQL.
7. **Schema-on-read vs schema-on-write?** Enforcement at insert (safety, migrations) vs at query (flexibility, app must validate).
