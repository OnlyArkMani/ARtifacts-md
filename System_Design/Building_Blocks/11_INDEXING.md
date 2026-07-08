# Database Indexing

## 1. What Problem It Solves

Without an index, finding a row means scanning the entire table — O(n), disastrous at millions of rows. An index is a sorted auxiliary structure that lets the database jump to matching rows in O(log n), like a book's index vs reading every page.

## 2. How It Works

### B-tree / B+tree (the default)

- Balanced tree with high fanout (hundreds of children per node) → a billion rows in 3–4 levels → 3–4 disk reads worst case.
- **B+tree specifics:** all values in leaves, leaves linked in sorted order → efficient **range scans** (`WHERE age BETWEEN 20 AND 30`, `ORDER BY`).
- Supports: equality, ranges, prefix matches (`LIKE 'abc%'`), sorting.

### Hash index

- Hash table: key → row location. O(1) equality lookups only. **No ranges, no ordering, no prefixes.** (Memory stores like Redis use this internally.)

### Other index concepts worth naming

- **Composite index** on (a, b, c): usable for filters on `a`, `a,b`, `a,b,c` — the **leftmost prefix rule**. Order columns by: equality filters first, then range/sort columns.
- **Covering index:** index contains all columns the query needs → answer straight from the index, never touching the table ("index-only scan").
- **Clustered index:** the table *is* stored in index order (InnoDB primary key). Secondary indexes point to the primary key. Only one clustered index per table.
- **Inverted index:** term → documents containing it. The basis of full-text search (see `15_SEARCH_INVERTED_INDEX.md`).
- **LSM trees (Cassandra/RocksDB/LevelDB):** writes buffered in memory (memtable), flushed as sorted immutable files (SSTables), merged by compaction. Much faster writes than B-trees (sequential I/O), slightly slower reads (check multiple files, mitigated by Bloom filters). This is *the* NoSQL write-optimization story.

## 3. Tradeoffs

**Pros:** reads go from O(n) to O(log n)/O(1); enables sorts and ranges without scanning.

**Cons:** **every index slows every write** (each INSERT/UPDATE must update all indexes), storage overhead, the query planner might not use the index you expect (functions on columns, low selectivity, type mismatches).

**When NOT to index:** write-heavy tables where read patterns don't justify it; low-cardinality columns alone (e.g., `gender` — index eliminates too little); tiny tables; columns only appearing in `SELECT`, never in filters/joins/sorts.

## 4. Diagram

```
B+tree (fanout shown as 3 for readability; real ~hundreds):

                [ 40 | 80 ]                ← root (in memory)
             /      |       \
      [10|25]    [55|70]   [90|95]         ← internal
      /  |  \    ...            \
 [5,8][10..][25..]  ...        [95,99]     ← leaves (actual rows/ptrs)
   └──▶└──▶└──▶ linked list ──▶└──▶        ← makes range scans cheap

Query: WHERE id BETWEEN 25 AND 60 → descend to leaf 25, walk right.
```

## 5. Common Interview Follow-ups

**Q: Why not index every column?**
A: Writes: every index must be updated on every write, plus storage and planner confusion. Index what your queries filter/join/sort on — driven by actual query patterns.

**Q: B-tree vs hash index?**
A: Hash: O(1) but equality only. B-tree: O(log n) but ranges, ordering, prefixes. Databases default to B-trees because real queries need ranges and sorts.

**Q: You have an index on (last_name, first_name). Which queries use it?**
A: `WHERE last_name = X` ✅; `WHERE last_name = X AND first_name = Y` ✅; `WHERE first_name = Y` alone ❌ (leftmost prefix rule).

**Q: Why are writes fast in Cassandra?**
A: LSM trees — writes are appends to an in-memory memtable + commit log (sequential I/O), flushed to immutable SSTables later. No in-place page updates like B-trees.

**Q: Query is slow despite an index — why might that be?**
A: Planner skipped it: function wrapping the column (`WHERE UPPER(name)=...`), leading wildcard `LIKE '%x'`, low selectivity (returns most of table — scan is cheaper), stale statistics, or type coercion. Check with `EXPLAIN`.

---
**Used in case studies:** URL Shortener (key lookup), News Feed, Job Scheduler (due-time index), plus DBMS cross-ref: `../../Core_CS/DBMS/04_INDEXING.md`.
