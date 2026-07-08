# Transactions & ACID

> **Tag: Theory/recall + scenario questions** — ACID with the bank-transfer example is near-guaranteed; isolation levels are the depth probe.

## Concept

A **transaction** = a group of operations that must behave as one atomic unit: `BEGIN … COMMIT` (all applied) or `ROLLBACK` (none). Canonical example: transfer ₹500 A→B = debit A + credit B — a crash between the two must not lose or invent money.

## ACID

- **Atomicity** — all or nothing. Mechanism: undo log / rollback journal.
- **Consistency** — every commit moves the DB from one valid state to another (constraints, FKs, invariants hold). Partly the app's responsibility.
- **Isolation** — concurrent transactions don't see each other's intermediate states; result equivalent to *some* serial order (at the strictest level). Mechanism: locks or MVCC.
- **Durability** — once committed, survives crashes. Mechanism: **write-ahead log (WAL)** flushed (fsync) before ack — connect to OS file-systems file: durability is literally fsync.

## Read Anomalies (learn as stories)

- **Dirty read:** T2 reads T1's *uncommitted* write; T1 rolls back → T2 used data that never existed.
- **Non-repeatable read:** T1 reads a row twice; T2 commits an update in between → different values.
- **Phantom read:** T1 runs a *range* query twice; T2 inserts a matching row in between → new "phantom" rows.
- **Lost update:** two read-modify-writes interleave; one overwrites the other (both read balance 100, both write 100+x).

## Isolation Levels (the table to memorize)

| Level | Dirty read | Non-repeatable | Phantom | How |
|---|---|---|---|---|
| READ UNCOMMITTED | possible | possible | possible | ~no read isolation |
| READ COMMITTED | ✗ | possible | possible | read only committed (default: Postgres, Oracle) |
| REPEATABLE READ | ✗ | ✗ | possible* | snapshot per txn (default: MySQL/InnoDB; *InnoDB largely prevents phantoms via gap locks) |
| SERIALIZABLE | ✗ | ✗ | ✗ | full serial equivalence (predicate locks / SSI) — slowest |

Tradeoff: isolation ↑ = concurrency ↓ (more locking/aborts). Real systems mostly run READ COMMITTED or REPEATABLE READ and handle the gaps in app logic (`SELECT … FOR UPDATE`, optimistic version checks).

**MVCC (how Postgres/MySQL actually do it):** writers create new row *versions* instead of overwriting; readers see a consistent snapshot by transaction ID → **readers never block writers, writers never block readers**. Cost: old versions pile up (vacuum/purge).

## Scenario Answers (rapid-fire prep)

- *Crash after debit, before credit?* Atomicity: WAL has no commit record → recovery rolls back the debit.
- *Two simultaneous ₹100 withdrawals from ₹150 balance?* Isolation: row lock or CAS (`UPDATE … WHERE balance >= 100`) makes one fail — the check and write must be atomic.
- *Committed but power dies?* Durability: commit ack only after WAL fsync → replay on restart.
- *Ticket booking double-sell?* `SELECT … FOR UPDATE` (pessimistic) or version-column retry (optimistic) — cross-ref `06_CONCURRENCY_CONTROL.md`.

## Most-Asked Interview Questions

1. **Explain ACID with an example.** → bank transfer, one line per letter, name the mechanism (undo log, constraints, locks/MVCC, WAL).
2. **Isolation levels + what each prevents?** → the table; know your DB's default.
3. **Dirty vs non-repeatable vs phantom?** → same-row uncommitted / same-row committed-change / range-new-rows.
4. **What is MVCC?** Versioned rows + snapshots; readers don't block writers; the reason Postgres feels fast under mixed load.
5. **How is durability actually implemented?** WAL + fsync before ack; batch/group commit for throughput.
6. **What's a lost update and how do you prevent it?** → interleaved read-modify-write; row locks, atomic `UPDATE SET x = x - ?`, or optimistic versioning.
7. **ACID vs BASE?** BASE (Basically Available, Soft state, Eventually consistent) = the distributed/NoSQL relaxation — bridge to `08_SQL_VS_NOSQL.md` and CAP.
