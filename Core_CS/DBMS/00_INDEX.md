# DBMS — Index & Fast Revision

## Tiered Priority

**Tier 1 (asked constantly):**
- `05_TRANSACTIONS_ACID.md` — ACID + isolation levels; near-guaranteed
- `02_NORMALIZATION.md` — "what NF is this?" drills
- `04_INDEXING.md` — B-tree, clustered vs non-clustered, slow-query whys
- `07_JOINS.md` — types + query writing (pairs with live SQL tests)
- `03_KEYS.md` — quick-fire definitions

**Tier 2 (commonly asked):**
- `06_CONCURRENCY_CONTROL.md` — 2PL, optimistic vs pessimistic, DB deadlocks
- `08_SQL_VS_NOSQL.md` — comparison + BASE

**Tier 3 (occasional):**
- `01_ER_AND_RELATIONAL_MODEL.md` — mapping rules
- `09_CAP_IN_DBMS.md` — light touch, cross-refs System Design

## All Comparison Tables in One Scan

**Keys:** superkey ⊇ candidate (minimal) ∋ primary (chosen, NOT NULL, one) | unique/alternate (many, NULL ok) | foreign (referential integrity, NULLable) | surrogate (stable, meaningless) vs natural (meaningful, mutable — keep as UNIQUE).

**Normal forms:** 1NF atomic → 2NF no partial dep (whole key) → 3NF no transitive dep (nothing but the key) → BCNF every determinant a superkey. Anomalies: update/insert/delete.

**ER mapping:** 1:N → FK on N side · M:N → junction table (+ relationship attrs) · weak entity → owner PK in composite key.

**Isolation levels:** RU (dirty possible) → RC (no dirty; PG default) → RR (no non-repeatable; MySQL default) → Serializable (no phantoms). Anomalies: dirty / non-repeatable / phantom / lost update.

**ACID C vs CAP C:** integrity constraints vs replica agreement — not the same thing.

**Locks:** S+S ✓, S+X ✗, X+X ✗ · 2PL grow-then-shrink ⇒ serializable · strict 2PL holds X to commit. DB deadlock → detect cycle, kill victim, app retries.

**Optimistic vs pessimistic:** version-check-retry (low contention) vs lock-first (high contention); SELECT FOR UPDATE vs version column CAS.

**Joins:** INNER (matches) ⊆ LEFT (all left + NULLs) ⊆ FULL; CROSS = product; anti-join = LEFT + IS NULL. WHERE on right table turns LEFT into INNER (use ON). Engines: nested-loop (indexed inner) / hash (big unsorted) / merge (sorted).

**Clustered vs non-clustered index:** table stored in key order (one) vs separate structure with pointers/PK refs (many). Composite index: leftmost prefix rule. Covering index: query answered in-index.

**B-tree vs hash index:** ranges+sort+prefix O(log n) vs equality-only O(1).

**SQL vs NoSQL:** enforced schema + ACID + joins + vertical/manual-shard vs flexible + BASE + denormalized + horizontal. Families: KV / document / wide-column / graph.

**WHERE vs HAVING:** pre-grouping row filter vs post-aggregation group filter. **UNION vs UNION ALL:** dedup vs keep.

## Standard Drills
Normalize the ORDERS example (in file 02) · trace isolation anomalies · write: second-highest salary, dept-wise max, employees-without-orders · R+W>N quorum math.
