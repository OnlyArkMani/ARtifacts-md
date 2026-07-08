# Concurrency Control & Locking

> **Tag: Theory + scenario** — 2PL, shared/exclusive locks, optimistic vs pessimistic, and DB deadlocks; follows directly from ACID isolation.

## Concept

Many transactions run at once; concurrency control makes the outcome equivalent to some serial order (**serializability**) — or a chosen weaker level — while maximizing parallelism.

## Lock-Based (Pessimistic)

- **Shared (S) lock** — for reads; many holders OK. **Exclusive (X) lock** — for writes; sole holder.

| Request↓ Held→ | S | X |
|---|---|---|
| **S** | ✅ | ❌ |
| **X** | ❌ | ❌ |

- **Two-Phase Locking (2PL):** growing phase (acquire only) then shrinking phase (release only) — guarantees serializability. Problem: cascading aborts → **Strict 2PL** (hold X locks till commit) is what's used.
- **Granularity:** row < page < table; finer = more concurrency, more lock-management overhead. **Intention locks** (IS/IX) at table level advertise row-level activity so table locks can check compatibility fast.
- **Gap/next-key locks** (InnoDB REPEATABLE READ): lock the *gaps* between index entries → blocks phantom inserts.

## Deadlocks in Databases

T1 locks row A, wants B; T2 locks B, wants A. DBs **detect** (wait-for graph cycle) and **abort a victim** (cheapest rollback), returning a deadlock error — **apps must retry**. Prevention in app code: touch rows/tables in a consistent order, keep transactions short. (Cross-ref OS `05_DEADLOCK.md` — same theory, different recovery culture: DBs kill-and-retry rather than prevent.)

Timestamp-based prevention schemes (theory syllabus): **wait-die** (older waits, younger dies) and **wound-wait** (older wounds younger) — know the names.

## Optimistic Concurrency Control (OCC)

Don't lock — proceed, then **validate at commit**: if another transaction touched your read/write set, abort and retry. Implementation you'll actually use: **version column**:

```sql
UPDATE account SET balance = ?, version = version + 1
WHERE id = ? AND version = ?;   -- 0 rows updated ⇒ conflict ⇒ retry
```

| | Pessimistic (locks) | Optimistic (validate) |
|---|---|---|
| Best when | High contention | Low contention |
| Cost | Waiting, deadlocks | Aborted work on conflict |
| Reads block? | Can | Never |
| Example | SELECT FOR UPDATE | version-column CAS, MVCC-SSI |

**MVCC recap** (from ACID file): snapshot reads without locks + write versioning — most modern DBs are MVCC + locks for writes (hybrid). Serializable via SSI (Postgres) aborts dangerous patterns instead of locking predicates.

## Most-Asked Interview Questions

1. **Shared vs exclusive locks?** → compatibility matrix; reads share, writes exclude.
2. **What is 2PL and why does it guarantee serializability?** No lock acquired after any release ⇒ transactions order by their "lock point"; strict 2PL also fixes cascading aborts.
3. **Optimistic vs pessimistic — when each?** Contention decides: hot rows → lock; rare conflicts → version-check and retry (cheaper, no deadlocks).
4. **How do databases handle deadlock?** Detect cycle → abort victim → app retries; prevent by ordering + short transactions.
5. **What does SELECT … FOR UPDATE do?** Takes X locks on read → later update can't hit a lost-update race; the pessimistic booking answer.
6. **Why can MySQL block an insert into a "gap"?** Next-key locking preventing phantoms at REPEATABLE READ.
7. **Lock escalation?** Many row locks converted to a table lock to save memory — sudden concurrency cliff; keep transactions small.
8. **How would you implement a counter hit by 10k writers/s?** Don't fight over one row: shard the counter / batch in app / use atomic increments — bridges to System Design hot-key patterns.
