# Database Replication & Sharding

## 1. What Problem It Solves

A single database has three limits: it can die (availability), it can't serve unlimited reads (read throughput), and it can't store/write unlimited data (capacity/write throughput). **Replication** (copies of the same data) fixes the first two. **Sharding** (splitting data across machines) fixes the third. They solve different problems and are usually combined.

## 2. How It Works

### Replication

- **Leader–follower (primary–replica):** all writes go to the leader; followers replicate the write log and serve reads. Most common setup.
  - **Async replication:** leader confirms write immediately, followers catch up. Fast, but a follower can serve stale reads (**replication lag**), and a leader crash can lose the last few writes.
  - **Sync replication:** leader waits for follower(s) to confirm. No loss, but slower writes and a stalled follower stalls writes. Compromise: semi-sync (wait for 1 of N).
- **Failover:** leader dies → a follower is promoted. Hard parts: detecting real failure (vs network blip), avoiding two leaders (**split brain**), losing unreplicated writes.
- **Multi-leader:** multiple nodes accept writes (multi-region). Requires conflict resolution (last-write-wins, CRDTs, app-level merge).
- **Leaderless (Dynamo-style):** write to N replicas, succeed on W acks, read from R; if W + R > N you read at least one fresh copy (quorum).

### Sharding (horizontal partitioning)

Split rows across machines by a **shard key**:

- **Hash-based:** `shard = hash(key) % N`. Even distribution, but range queries scatter to all shards, and changing N reshuffles everything → use **consistent hashing** (see `07_CONSISTENT_HASHING.md`).
- **Range-based:** users A–M on shard 1, N–Z on shard 2. Great for range scans; risk of hot shards (e.g., time-ordered keys hammer the newest shard).
- **Directory-based:** a lookup service maps key → shard. Flexible, but the directory is a SPOF and extra hop.

**Choosing a shard key** is the whole game: it should distribute load evenly AND match your dominant query pattern (queries hitting one shard are cheap; cross-shard queries/joins/transactions are expensive and painful).

## 3. Tradeoffs

**Replication — pros:** read scaling, high availability, geographic locality. **Cons:** replication lag (read-your-own-writes problems), failover complexity, storage cost ×N.

**Sharding — pros:** write and storage scaling beyond one machine. **Cons:** cross-shard joins/transactions are hard or impossible, resharding is operationally painful, hot shards, application complexity.

**When NOT to shard:** if you're not out of vertical scaling headroom + read replicas + caching. Sharding is a last resort — interviews reward saying "I'd exhaust replication and caching first."

## 4. Diagram

```
Replication (reads scale):          Sharding (writes/storage scale):

      writes                          user_id % 3
        │                                 │
        ▼                        ┌────────┼────────┐
    ┌────────┐                   ▼        ▼        ▼
    │ Leader │               ┌───────┐┌───────┐┌───────┐
    └───┬────┘               │Shard 0││Shard 1││Shard 2│
   async│repl                │ A–H   ││ I–P   ││ Q–Z   │
   ┌────┴─────┐              └───────┘└───────┘└───────┘
   ▼          ▼               each shard is itself replicated:
┌─────────┐┌─────────┐        Shard 0 = leader + 2 followers
│Follower1││Follower2│ ◀reads
└─────────┘└─────────┘
```

## 5. Common Interview Follow-ups

**Q: A user posts a comment then refreshes and doesn't see it. Why?**
A: Replication lag — write hit the leader, read hit a stale follower. Fixes: read-your-own-writes (route that user's reads to leader briefly), session stickiness, or version tokens the read must catch up to.

**Q: How do you pick a shard key for a social app?**
A: `user_id` — most queries are per-user, so they hit one shard. Celebrity users create hot shards → mitigate with caching, or splitting hot users' data further. A bad key: timestamp (all writes hit the newest shard).

**Q: What happens when you add a shard?**
A: With naive modulo hashing, nearly all keys remap → massive data movement. Use consistent hashing (only ~1/N keys move) or pre-split into many virtual shards/partitions and move whole partitions.

**Q: How do you do a transaction across shards?**
A: Avoid it by design if possible (choose shard key so related data co-locates). Otherwise: two-phase commit (slow, blocking) or sagas (sequence of local transactions with compensating rollbacks) — accept eventual consistency.

**Q: Sync or async replication?**
A: Async by default for latency; sync/semi-sync when losing acknowledged writes is unacceptable (payments). State the tradeoff: durability vs write latency/availability.

---
**Used in case studies:** every Scaled version; central to Key-Value Store, Payment System, Distributed Cache.
