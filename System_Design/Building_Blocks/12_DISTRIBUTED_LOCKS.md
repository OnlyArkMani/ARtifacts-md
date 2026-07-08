# Distributed Locks

## 1. What Problem It Solves

In one process, a mutex stops two threads doing conflicting work. But when the "threads" are different servers — two workers grabbing the same job, two requests booking the last seat — you need a lock that lives *outside* any single machine: a distributed lock. It ensures at most one node performs a critical action at a time.

## 2. How It Works

### Redis-based (simple, common)

- Acquire: `SET lock:order:42 <my_token> NX PX 30000` — set only if not exists, with a 30s TTL.
- The **TTL is essential**: if the holder crashes, the lock self-releases instead of deadlocking the system.
- Release: delete the key **only if it still holds your token** (atomic Lua script) — otherwise you might delete someone else's lock after your TTL expired.
- **The fundamental flaw:** holder pauses (GC, network) past TTL → lock expires → another node acquires → **two holders**. Mitigation: **fencing tokens** — the lock service issues a monotonically increasing number; the protected resource rejects operations with stale tokens.
- **Redlock** (multi-Redis-node quorum) attempts stronger guarantees; famously debated (Kleppmann) — fine to mention, don't over-claim its safety.

### Consensus-based (ZooKeeper/etcd — safer)

- Locks built on a consensus (Raft/ZAB) cluster. ZooKeeper: create an **ephemeral sequential node**; lowest sequence holds the lock; ephemeral = auto-released when the client's session dies. Each client watches only its predecessor (no herd effect).
- Stronger correctness, higher latency and operational weight.

### Often better: avoid the lock

- **DB atomic ops:** `UPDATE seats SET holder=? WHERE id=? AND holder IS NULL` — the row lock does the job.
- **Optimistic concurrency:** version column; update fails if version changed → retry.
- **Partition ownership:** route all work for key X to one worker (Kafka partition) — mutual exclusion by design, no lock at all.

## 3. Tradeoffs

**Pros:** correctness for cross-node critical sections; prevents duplicate side effects (double-charge, double-send).

**Cons:** lock service = SPOF/bottleneck; TTL dilemma (too short → false expiry, too long → slow recovery); pauses can break mutual exclusion without fencing; adds latency to every protected operation.

**When NOT to use:** when idempotency can absorb duplicates more cheaply (see `13_IDEMPOTENCY.md`); when a DB transaction/atomic update suffices; for efficiency-only dedup where occasional double-work is harmless (just accept it).

## 4. Diagram

```
Worker A ──SET lock NX PX──▶ ┌───────┐  ✅ acquired (token 33)
Worker B ──SET lock NX PX──▶ │ Redis │  ❌ exists → wait/retry
                             └───────┘
Danger:                            Fencing fix:
A acquires (t=33) → GC pause       Storage rejects writes with
TTL expires → B acquires (t=34)    token < 34, so A's late write
A wakes, still thinks it owns! ⚠   with 33 is refused ✅
```

## 5. Common Interview Follow-ups

**Q: What happens if a lock holder crashes?**
A: TTL (Redis) or ephemeral session node (ZooKeeper) releases the lock automatically. Without expiry you get a permanent deadlock; with it you get the false-expiry risk — hence fencing tokens.

**Q: Explain fencing tokens.**
A: Lock service hands out increasing numbers with each grant. Downstream resources record the highest seen and reject lower ones. Even if two nodes think they hold the lock, only the newer one's writes are accepted.

**Q: Redis lock vs ZooKeeper lock?**
A: Redis: fast, simple, weaker guarantees (async replication can lose a lock on failover). ZooKeeper/etcd: consensus-backed, correct under partitions, heavier. Choose by cost of a safety violation — efficiency use → Redis; correctness-critical → consensus (or redesign to avoid locking).

**Q: How would you prevent double-booking a seat without a distributed lock?**
A: Conditional write in the database: atomic compare-and-set on the seat row, or unique constraint on (seat, showtime). Push mutual exclusion into the datastore that already has ACID.

**Q: Lock contention is killing throughput — what now?**
A: Shrink critical sections, shard the lock (per-resource keys, not one global lock), or restructure to partition ownership so exclusion is free.

---
**Used in case studies:** Job Scheduler (leader election, no-duplicate-runs), Ride-Sharing (driver assignment), Payment System.
