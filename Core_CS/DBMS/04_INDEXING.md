# Indexing (B-tree, Hash, and When to Use)

> **Tag: Theory + applied ("why is this query slow?")** — overlaps System Design `Building_Blocks/11_INDEXING.md`; this file is the DBMS-interview angle.

## Concept

Index = auxiliary sorted/hashed structure mapping column values → row locations, turning O(n) scans into O(log n)/O(1) lookups. The DBMS picks whether to use it via the query planner (`EXPLAIN` shows the decision).

## B-tree / B+tree (default everywhere)

- High-fanout balanced tree; leaves hold values (B+), linked left→right → **range scans and ORDER BY** are cheap.
- Height 3–4 for billions of rows ⇒ 3–4 page reads.
- Supports: `=`, `<`, `BETWEEN`, prefix `LIKE 'ab%'`, sorting. Not: leading-wildcard `LIKE '%ab'`, arbitrary functions.

```
            [ 40 | 80 ]
          /     |      \
    [10|25]  [55|70]  [90|95]
    leaves: [5,8]→[10,20]→[25,30]→… (linked for ranges)
```

## Hash Index

Bucket by hash(key) → O(1) exact-match. **No ranges, no ordering.** Postgres `USING hash`; MySQL MEMORY tables; everything in-memory KV (Redis).

| | B-tree | Hash |
|---|---|---|
| Equality | O(log n) | O(1) |
| Range / sort / prefix | ✅ | ❌ |
| Default? | Yes | Niche |

## Clustered vs Non-Clustered (top question)

- **Clustered:** the table's rows are physically stored in index order — the index *is* the table (InnoDB PK, SQL Server clustered). One per table. Range scans on the cluster key are optimal.
- **Non-clustered (secondary):** separate structure; leaves hold pointers — row IDs (heap) or **the PK value** (InnoDB → secondary lookups do index → PK → row: two B-tree descents; why fat PKs hurt every secondary index).

## Composite, Covering, Partial

- **Composite (a,b,c):** serves filters on `a` / `a,b` / `a,b,c` — **leftmost prefix rule**. Order: equality columns first, then the range/sort column.
- **Covering:** index contains everything the query touches → index-only scan, never hits the table. `INDEX(user_id, created_at, amount)` covers `SELECT amount WHERE user_id=? ORDER BY created_at`.
- **Partial/filtered:** index only rows matching a predicate (`WHERE deleted = false`) — smaller, faster.

## When Indexes Don't Help / Hurt

- **Every write updates every index** — write-heavy tables want few indexes.
- Low selectivity (`gender`): planner rightly ignores it — scanning is cheaper than millions of pointer hops.
- Functions on the column (`WHERE YEAR(date)=2024`) defeat it → rewrite as range or create a function/expression index.
- Stale statistics, implicit type casts → planner surprises. Debug with `EXPLAIN ANALYZE`.

## Most-Asked Interview Questions

1. **How does a B+tree index work? Why B-tree not BST/hash?** Disk-page-sized high-fanout nodes minimize I/O (3 levels vs 30 for BST); hash can't do ranges.
2. **Clustered vs non-clustered?** Physical order + is-the-table vs separate + pointer/PK indirection; one vs many.
3. **This query is slow despite an index — why?** Leading wildcard, function on column, low selectivity, wrong composite order (leftmost rule), stale stats, type mismatch.
4. **Index on (last_name, first_name): which queries can use it?** last_name alone ✓, both ✓, first_name alone ✗.
5. **Why not index everything?** Write amplification + storage + planner confusion.
6. **What's a covering index?** Query answered entirely from the index — no table access; the go-to optimization for hot read queries.
7. **How do indexes interact with joins?** FK columns should be indexed — join = repeated lookup of the join key; missing FK indexes are the #1 real-world slow-join cause.
8. **B-tree vs LSM?** (bridges to System Design/NoSQL) In-place page updates, read-optimized vs append + compaction, write-optimized (Cassandra/RocksDB).
