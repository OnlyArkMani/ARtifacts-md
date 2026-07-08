# Core CS Fundamentals — Master Index

Four subjects, each with topic files + a `00_INDEX.md` containing tiered priorities and all comparison tables merged for fast scanning.

- `OS/` — 8 topics · `Computer_Networks/` — 9 topics · `DBMS/` — 9 topics · `OOP/` — 7 topics

## Suggested Study Order (limited time)

These subjects are tested as rapid-fire Q&A — optimize for *recall density*, interleaving subjects rather than exhausting one:

**Pass 1 — Tier 1s only (2–3 days):**
1. **OOP** four pillars + overload/override (fastest wins; every Java interview opens here)
2. **DBMS** ACID + normalization + indexing + joins (pairs with SQL screening tests)
3. **OS** process vs thread + synchronization + deadlock
4. **CN** TCP vs UDP + HTTP + OSI + DNS

**Pass 2 — Tier 2s (2 days):** scheduling & page replacement traces · subnetting drills · concurrency control & SQLvsNoSQL · SOLID + patterns + code traps.

**Pass 3 — day before interview:** read only the four `00_INDEX.md` files (all tables) + redo the standard drills (page trace, subnet calc, normalization example, singleton from memory).

Rule of thumb: **comparisons are the highest-yield format** — most questions are literally the tables in the index files.

## Cross-Subject Connections (interviewers love these)

- **ACID durability = OS fsync/WAL:** the D in ACID is implemented by write-ahead logs flushed via `fsync` — DB journaling and filesystem journaling are the same idea (DBMS 05 ↔ OS 08).
- **DB deadlocks vs OS deadlocks:** same four Coffman conditions and wait-for graphs; different culture — OS mostly prevents/ignores, DBs detect-and-kill-victim + app retries (OS 05 ↔ DBMS 06).
- **TCP and the OS:** sockets are file descriptors; accept() queues, buffer sizes = flow-control windows, epoll event loops — networking is delivered *through* OS abstractions (CN 08 ↔ OS 07).
- **MVCC vs synchronization:** databases avoid read locks with versioning; the same optimistic/pessimistic split as application-level locking (DBMS 06 ↔ OS 06).
- **LRU everywhere:** page replacement (OS 04), CPU caches, Redis eviction, DB buffer pools — one algorithm, four contexts; interviewers connect them deliberately.
- **B-trees everywhere:** DB indexes (DBMS 04), filesystems (ext4 extents, NTFS) — disk-page-optimized trees are the shared answer to "big data on slow disks."
- **HTTP idempotency ↔ retry safety:** GET/PUT idempotent by contract (CN 03) = the same idempotency reasoning as distributed systems (System Design artifact, `Building_Blocks/13_IDEMPOTENCY.md`).
- **Concurrency vs parallelism (OS 07) ↔ thread-safe singletons and `volatile` (OOP 06/07):** the Java memory-model questions are OS concepts wearing Java syntax.
- **CAP/consistency:** DBMS 09 is the light version; the full story with quorums and consistency models lives in `../System_Design/Concepts/CAP_theorem_and_consistency_models.md`.

## Format Expectation

These rounds are usually: definition → comparison → "why?" → small applied problem → cross-topic connection. The files mirror that: every topic has a theory-vs-applied tag, mandatory comparison tables, and a ranked most-asked list. Practice answering each most-asked question aloud in ≤45 seconds.
