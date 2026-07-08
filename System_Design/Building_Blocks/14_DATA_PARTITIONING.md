# Data Partitioning Strategies

## 1. What Problem It Solves

When data or traffic outgrows one machine, you must split it. *How* you split determines everything downstream: which queries are fast, which shards run hot, and how painful rebalancing is. (Sharding = partitioning across machines; this file goes deeper on the strategy choices than `03_REPLICATION_AND_SHARDING.md`.)

## 2. How It Works

### Horizontal partitioning (sharding) — split rows

| Strategy | How | Best for | Weakness |
|---|---|---|---|
| **Hash** | `partition = hash(key) mod N` (or consistent hashing / hash slots) | Uniform load, point lookups | Range queries hit all partitions; key adjacency destroyed |
| **Range** | Key ranges per partition (A–F, G–M, …; or time ranges) | Range scans, time-series queries | Hot partitions (sequential keys pile onto one partition) |
| **Directory/lookup** | Explicit key→partition map service | Full control, easy targeted moves | Extra hop, directory must be HA and consistent |
| **Geo** | Partition by user region | Data residency laws, latency locality | Cross-region queries; uneven region sizes |

**Hybrid (very common):** compound key — hash on a partition key, range-sort within it. DynamoDB/Cassandra: `(partition_key, sort_key)` → `hash(user_id)` picks the node, messages sorted by `timestamp` within. Gets you distribution AND per-entity range scans.

### Vertical partitioning — split columns/tables
Hot, small columns (username, avatar) separate from cold, wide ones (bio, settings blob); or whole tables split by domain (the microservices database-per-service move). Improves cache density and isolates load.

### The hot partition problem
Skewed access (celebrity user, today's date as key) overwhelms one partition. Fixes: **key salting** (append random suffix to spread writes: `key#1..key#10`, fan-in on read), split the hot range, dedicated cache for hot entities, or choose a higher-cardinality key.

### Rebalancing
- Fixed many partitions up front (e.g., 1024 vnodes), move whole partitions between nodes as needed (Kafka/Cassandra style) — the practical default.
- Dynamic splitting when a partition grows too big (HBase, DynamoDB adaptive).
- Never `mod N` with data at rest — adding a node reshuffles everything.

## 3. Tradeoffs

**Pros:** unbounded storage/write scaling; failure blast radius shrinks to one partition; parallel query execution.

**Cons:** cross-partition queries/joins/transactions become scatter-gather or impossible; secondary indexes get hard (local index per partition → query all partitions; global index → the index itself must be partitioned and kept consistent); operational complexity.

**When NOT to partition:** while one primary + replicas + caching still holds — premature sharding is expensive to undo. Also avoid partitioning small dimension datasets that could just be replicated everywhere.

## 4. Diagram

```
Hash:  hash(user_id) ──▶ P0 | P1 | P2 | P3     (even, no ranges)

Range: [A–F]▶P0  [G–M]▶P1  [N–S]▶P2  [T–Z]▶P3  (ranges OK, hot risk)

Compound (DynamoDB-style):
  hash(user_42) ─▶ P2:  [ user_42 | msg@t1 ]
                        [ user_42 | msg@t2 ]  ← sorted within partition
                        [ user_42 | msg@t3 ]    → "last 50 msgs" = 1 partition read
```

## 5. Common Interview Follow-ups

**Q: How do you choose a partition key?**
A: Three tests: (1) high cardinality and even spread, (2) most queries include it (single-partition reads), (3) writes don't concentrate on one value. E.g., chat: `conversation_id` beats `sender_id` — all messages of a conversation co-locate.

**Q: Timestamp as partition key — good idea?**
A: Usually terrible for writes: all current writes hit "now"'s partition. Fine for range reads. Compromise: hash something else for distribution, keep time as the sort key; or salt time buckets.

**Q: How do secondary indexes work on partitioned data?**
A: Local (per-partition) index — cheap writes, but a query by that attribute must fan out to all partitions. Global index — single-partition query, but the index is itself partitioned and updated async (DynamoDB GSI: eventually consistent). State the tradeoff.

**Q: What happens operationally when you split/move a partition?**
A: Copy data to the new owner while serving traffic, dual-write or replicate the delta, atomically flip routing metadata, then delete the old copy. Systems with fixed vnodes just move whole vnodes — that's why we pre-split.

**Q: How would you find a hot partition and fix it live?**
A: Per-partition metrics (QPS, latency, size); once found: add caching for its hot keys, salt its key space, or manually split that range.

---
**Used in case studies:** every Scaled version; the compound-key trick is central to Chat and News Feed; salting appears in YouTube (view counters).
