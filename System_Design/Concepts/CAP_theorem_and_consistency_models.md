# CAP Theorem & Consistency Models

## CAP in Plain English

A distributed system stores data on multiple nodes. When the network between them breaks (**Partition**), you must choose:

- **Consistency (C):** every read sees the latest write — so during a partition, nodes that can't confirm they're current must refuse to answer.
- **Availability (A):** every request gets a response — so during a partition, nodes answer with whatever they have, possibly stale.

You cannot have both *during a partition*. And **partitions are not optional** — networks fail. So CAP really says: **when (not if) a partition happens, pick C or A.**

Common misreading to avoid: "pick 2 of 3" as if P were skippable. P is forced; the choice is C-vs-A *under partition*. In normal operation you can have both.

### PACELC (the useful extension)

**If Partition: choose A or C. Else (normal operation): choose Latency or Consistency.** Even with no partition, strong consistency costs latency (coordination/quorums). E.g., DynamoDB is PA/EL; Spanner is PC/EC.

## CP vs AP Systems — Real Examples

| System | Type | Behavior under partition |
|---|---|---|
| ZooKeeper, etcd | **CP** | Minority side refuses writes (needs quorum); correct but partially unavailable |
| Spanner, CockroachDB | **CP** | Consensus per write; minority partitions stall |
| HBase, MongoDB (default) | **CP-leaning** | Failover pauses; primaries step down without quorum |
| Cassandra, DynamoDB | **AP** (tunable) | All reachable nodes accept reads/writes; conflicts reconciled later |
| DNS, most caches | **AP** | Serve stale happily |

Match to domain: bank ledger, inventory, config/leader election → CP. Shopping cart, likes, presence, feeds → AP (staleness is fine; downtime isn't).

## Consistency Models (strongest → weakest)

1. **Strict/Linearizable:** every read returns the most recent write, as if one copy existed. Cost: coordination on every operation (consensus, quorums). Needed for: locks, leader election, account balances at transaction time.
2. **Sequential:** all nodes see operations in the same order (not necessarily real-time order).
3. **Causal:** operations that are causally related (reply after message) are seen in order everywhere; concurrent unrelated ops may interleave differently. Sweet spot for social apps.
4. **Session guarantees** (client-centric, cheap and practical):
   - **Read-your-own-writes:** after I post, *I* always see my post (others may not yet).
   - **Monotonic reads:** I never see data go backwards in time between requests.
5. **Eventual:** if writes stop, replicas converge... eventually. No ordering promises meanwhile. Cheapest, most available.

### Making eventual consistency livable

- **Quorums:** N replicas, write to W, read from R; **R + W > N ⇒ reads see latest write** (e.g., N=3, W=2, R=2). Tune per operation.
- **Conflict resolution** for concurrent writes: last-write-wins (simple, loses data), version vectors (detect conflicts, let app merge), **CRDTs** (data types that merge automatically — counters, sets).
- **Read repair / anti-entropy:** replicas sync differences in background (Merkle trees).

## Interview Playbook

When asked "how consistent does this need to be?":

1. Split the system's data by requirement — most systems are mixed: payments ledger = linearizable; product views counter = eventual; user sees own profile edit = read-your-writes.
2. Name the cost: strong consistency ⇒ higher latency (quorum/consensus round trips), lower availability under partition, worse multi-region performance.
3. Name the mitigation for weak consistency: idempotency, conflict resolution strategy, session guarantees for UX.

**Follow-ups you'll get:**

**Q: Is eventual consistency ever visible to users? How do you hide it?**
A: Yes — posting a comment that vanishes on refresh (replication lag). Hide with read-your-writes: route the author's reads to the leader briefly, or reflect the write optimistically in the UI.

**Q: Why is strong consistency expensive across regions?**
A: Every write needs agreement across regions → cross-continent RTTs (~100–150ms) on the write path. AP systems write locally and reconcile async.

**Q: Where does CAP not apply?**
A: Single-node systems (no partition possible) and it says nothing about latency or normal operation — that's why PACELC exists.

**Q: How does DynamoDB let you choose?**
A: Eventually consistent reads (default, cheaper) vs strongly consistent reads (within region) vs global tables (cross-region, eventual, last-writer-wins).

---
Cross-refs: quorums & replication → `../Building_Blocks/03_REPLICATION_AND_SHARDING.md`; DBMS-side view → `../../Core_CS/DBMS/09_CAP_IN_DBMS.md`.
