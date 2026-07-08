# Consistent Hashing

## 1. What Problem It Solves

You're spreading keys across N servers with `hash(key) % N`. Now add one server: N→N+1, and **almost every key maps to a different server**. For a cache that means a total cache wipe; for a database, a massive data migration. Consistent hashing makes cluster membership changes cheap: adding/removing a node only moves ~1/N of the keys.

## 2. How It Works

- Imagine the hash space (0 to 2³²−1) bent into a **ring**.
- Each **server** is hashed onto the ring. Each **key** is hashed onto the ring too.
- A key belongs to the **first server clockwise** from its position.
- Remove server B → only B's keys slide to the next server clockwise. Everyone else is untouched. Add a server → it takes over just the arc between it and its predecessor.

**Problem:** with few servers, arcs are uneven — one server may own half the ring. And when a server dies, its entire load dumps onto ONE neighbor.

**Fix — virtual nodes (vnodes):** map each physical server to many points on the ring (e.g., 100–1000 tokens each). Load evens out statistically, and a dead server's load spreads across *many* successors instead of one. Heterogeneous hardware: give bigger machines more vnodes.

**Replication on the ring:** store each key on the next R distinct servers clockwise (Dynamo/Cassandra style) — replication falls naturally out of the same structure.

## 3. Tradeoffs

**Pros:** minimal data movement on membership change, decentralized (any node can compute placement), natural fit for replication, no central directory needed.

**Cons:** more complex than modulo, load is only *statistically* even (vnodes mitigate), range queries scatter (hash destroys key order), still doesn't fix a single *hot key*.

**When NOT to use:** fixed, never-changing server count (modulo is fine); when you need range-partitioned data (use range sharding); small systems where a config-file lookup table is simpler.

## 4. Diagram

```
                    0/2³²
              ┌───────●───────┐
         S3 ●─┘               └─● S1
            │      RING          │
      keyX ─┼─▶ (clockwise       │
            │    → S1 owns it)   │
         S2 ●─┐               ┌──● (vnode of S3)
              └───────●───────┘
                      S1(vnode)

Add S4 between S2 and S3: only keys in that arc move to S4.
Modulo hashing would have remapped ~ all keys.
```

## 5. Common Interview Follow-ups

**Q: Why virtual nodes — what exactly do they fix?**
A: Two things: (1) uneven arc sizes with few nodes → vnodes average out placement; (2) failure load concentration — with vnodes, a dead node's keys redistribute across the whole cluster, not one neighbor.

**Q: Where is consistent hashing used in real systems?**
A: Cassandra/DynamoDB partitioning, Memcached client libraries (ketama), Redis Cluster (variant: 16384 hash slots), load balancers with sticky routing, CDN cache assignment.

**Q: How many keys move when a node joins a cluster of N?**
A: ~K/N of K total keys — only the arc the new node takes over. Versus nearly all K with modulo.

**Q: Consistent hashing vs a central directory (lookup table)?**
A: Directory gives perfect control and easy rebalancing but is a SPOF/extra hop and must be kept consistent. Consistent hashing is computed locally by anyone with the member list. Redis Cluster's hash slots are a hybrid: fixed 16384 slots, a small movable mapping of slots→nodes.

**Q: Does it solve hot keys?**
A: No — one extremely hot key still lands on one node. Handle separately: replicate the hot key across nodes and randomize reads, or add a local cache layer in front.

---
**Used in case studies:** Distributed Cache (core), Key-Value Store (core), Rate Limiter (shard counters), Web Crawler (URL → fetcher assignment).
