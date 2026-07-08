# Case Study: Distributed Key-Value Store (Easy → deceptively deep)

**Building blocks used:** Consistent Hashing · Replication · Data Partitioning · CAP/Consistency Models · Indexing (LSM trees)

This is "design mini-DynamoDB." Easy to start, unbounded depth — a favorite because it tests distributed fundamentals directly.

---

## 1. Basic Version (single server)

API: `put(key, value)`, `get(key)`, `delete(key)`.

- In-memory hash map → fast but lost on crash.
- Durability: **append-only log** (WAL) — every write appended before ack; on restart, replay log to rebuild the map.
- Data > RAM: **LSM-tree design** — writes go to memtable (sorted in-memory) + WAL; memtable flushes to immutable sorted files (**SSTables**); background **compaction** merges files; reads check memtable → SSTables newest-first, with **Bloom filters** to skip files that can't contain the key. (This is LevelDB/RocksDB.)

## 2. Scaled Version — full interview walkthrough

### Functional requirements
get/put/delete; values up to ~1MB; optionally TTL and atomic compare-and-set.

### Non-functional
- Scale beyond one node (say 100 TB, 1M QPS mixed)
- Highly available under node failure and network partition — choose **AP with tunable consistency** (Dynamo-style; state this choice explicitly)
- Low latency: p99 < 10ms

### Capacity estimate
100 TB / ~2 TB usable per node ≈ 50 nodes × 3 replication = 150 nodes; 1M QPS / 150 ≈ 7k QPS/node — comfortable. Numbers justify partitioning + replication immediately.

### High-level design

```
 Client ──▶ any node (coordinator via consistent-hash ring)
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
   ┌─────────┐      ┌─────────┐      ┌─────────┐
   │ Node A  │      │ Node B  │      │ Node C  │   key K stored on
   │ (K✓)    │      │ (K✓)    │      │ (K✓)    │   next N=3 nodes
   └─────────┘      └─────────┘      └─────────┘   clockwise on ring
   write: W=2 acks → success      read: R=2, newest version wins
   membership via gossip; each node runs an LSM engine locally
```

- **Partitioning:** consistent hashing with virtual nodes (see `../Building_Blocks/07_CONSISTENT_HASHING.md`).
- **Replication:** each key on N=3 successive ring nodes.
- **Quorums:** W + R > N ⇒ read overlaps latest write. Tune: W=3,R=1 (fast reads), W=1,R=3 (fast writes), 2/2 balanced.
- **Membership/failure detection:** gossip protocol; no central master (or, alternative design: a control plane à la etcd — mention both).

### Deep dive 1: consistency & conflict resolution
Async replication + concurrent writes ⇒ conflicting versions. Options: **last-write-wins** (simple; silently drops one write; needs sane clocks) vs **version vectors** (detect true conflicts, return both to client to merge — Dynamo's choice) vs CRDTs for specific types. Anti-entropy: **read repair** (fix stale replicas during reads) + background **Merkle-tree sync**. **Hinted handoff:** if a replica is down, a neighbor holds its writes and hands them over on recovery — keeps writes available during transient failures.

### Deep dive 2: the storage engine (if pushed)
LSM as in Basic version. Why LSM over B-tree here: write-heavy workloads → sequential appends beat random page updates; compaction cost is the price (write amplification, background I/O spikes — mention tuning/leveling).

### Bottlenecks & fixes
- Hot key → key salting/replication of hot keys, client-side caching.
- Node join/leave data movement → vnodes keep transfers small and parallel.
- Compaction storms → rate-limit compaction, off-peak scheduling.
- Large values → store blob in object storage, KV keeps pointer.

### Follow-up questions
- **How does a client know which node has the key?** Smart client with ring metadata (1 hop) vs any-node-as-coordinator (2 hops, simpler clients).
- **What if two writes conflict during a partition?** Version vectors detect; app merges (shopping-cart union) or LWW if acceptable — state data-loss implications.
- **How do you delete in an LSM?** Tombstones — a delete marker; the value truly disappears at compaction. (Also why deleted data can "resurrect" if tombstones expire too early.)
- **CP variant?** Use consensus (Raft) per partition — writes go through a leader with majority ack (etcd/Spanner style). Costs availability under partition + latency.
- **How do you add TTL?** Expiry timestamp per record, filtered on read, purged at compaction.

### Tradeoffs to defend
AP vs CP (chosen AP: shopping-cart-style use cases; would choose Raft-based CP for config/coordination data); quorum tuning; LWW vs vectors (simplicity vs correctness); gossip (no SPOF) vs control plane (simpler reasoning).

## 3. Advanced Version

- **Multi-region:** each region a full replica set; async cross-region replication; conflicts resolved by vectors/LWW — or per-key home region for stronger semantics.
- **Failure drills:** partition between coordinators → both sides accept writes (AP) → reconcile on heal; test tombstone + handoff interactions.
- **Range queries?** Hash partitioning can't; would need an order-preserving layer or secondary indexes (local: fan-out reads; global: async-updated index partitioned by indexed value).
- **Real systems to name:** DynamoDB (managed, this design + control plane), Cassandra (this design + SQL-ish CQL), Redis Cluster (in-memory, hash slots), etcd (CP alternative).
