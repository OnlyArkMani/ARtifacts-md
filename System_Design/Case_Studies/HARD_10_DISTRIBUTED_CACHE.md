# Case Study: Distributed Cache — Redis-like (Hard)

**Building blocks used:** Consistent Hashing (core) · Caching internals · Replication · Data Partitioning · CAP tradeoffs

"Design Redis/Memcached as a service." Tests whether you understand what's *inside* the building blocks you normally just draw as a box. Overlaps with Key-Value Store — the differences: **in-memory (speed is the product), eviction is a feature, durability is optional.**

---

## 1. Basic Version (single node)

- Hash map in RAM + TTL per key + **LRU eviction** at memory cap.
- LRU in O(1): hash map + doubly-linked list (map points to list nodes; access moves node to head; evict from tail) — the classic DSA question, live inside a system.
- Concurrency: single-threaded event loop (Redis' choice — no locks, simple, one core) vs sharded locks/multi-threaded (Memcached — uses all cores). Know both.
- Expiry: lazy (check TTL on read) + periodic active sampling of expired keys (Redis does both).

## 2. Scaled Version — full interview walkthrough

### Functional requirements
get/set/delete with TTL; atomic ops (INCR, CAS); memory-bounded with eviction policy; client library or proxy access.

### Non-functional
- Sub-millisecond p99; 10M+ QPS aggregate; 1 TB+ cached data
- **Availability over consistency** — it's a cache: serving slightly stale or missing (→ DB fallback) beats blocking. Say this early; it drives every choice.
- Cluster changes (add/remove node) must not wipe the cache

### Capacity estimate
1 TB / 64 GB usable per node ≈ 16 shards; 10M QPS / ~500k QPS per node ≈ 20 nodes — memory and QPS both point to ~16–32 shards (× 2 with replicas).

### High-level design

```
 Clients (smart client lib w/ ring map)         
    │  hash(key) → shard                        
    ├───────────────┬────────────────┐          
    ▼               ▼                ▼          
┌────────┐     ┌────────┐      ┌────────┐       
│Shard 1 │     │Shard 2 │      │Shard 3 │  ...  each shard:
│primary │     │primary │      │primary │       event loop + RAM store
│  │async│     │  │     │      │  │     │       + LRU + TTL
│replica │     │replica │      │replica │       
└────────┘     └────────┘      └────────┘       
 cluster topology: gossip + config epoch (or central config svc)
```

- **Partitioning: consistent hashing** with vnodes (or Redis-Cluster-style **16384 fixed hash slots** — a lookup table of slot→node; easier controlled migration). Adding a node moves ~1/N of keys — the cache survives topology changes, which was the requirement.
- **Routing:** smart clients holding the ring/slot map (1 hop, map refresh on MOVED responses) vs a proxy tier (simple clients, +1 hop, central choke). Both valid — name the tradeoff.
- **Replication:** async primary→replica per shard; on primary death, promote replica (fast failover; may lose last writes — acceptable for a cache, say so explicitly).

### Deep dive 1: eviction & memory management
- Policy choice per use case: LRU default; LFU for stable hot sets (Redis approximates both by **sampling** N random keys instead of exact ordering — exact LRU costs memory/coordination; approximation is nearly as good — great systems-thinking detail).
- TTL wheel/sampling for expiry; memory fragmentation (slab allocation à la Memcached vs jemalloc).
- Eviction under write burst → backpressure or evict-ahead watermarks.

### Deep dive 2: consistency corners (where interviewers push)
- **Failover write loss:** async replication → promoted replica misses last writes → cache returns stale/miss → fine for cache semantics; NOT fine if users abuse cache as a store — document the contract.
- **Split brain:** old primary keeps accepting writes after partition → epoch/config numbers (fencing): nodes reject commands from stale-epoch primaries.
- **Atomic ops on one key:** trivially safe within a shard (single-threaded loop = free serialization). Multi-key ops (MGET across shards / transactions): scatter-gather; cross-shard atomicity generally *not offered* (Redis Cluster requires same-slot via hash tags `{user42}.field`) — knowing this limitation is a strong signal.

### Bottlenecks & fixes
- **Hot key** (one celebrity key = 1M QPS on one shard): replicate hot key to R shards, clients read random replica; or client-local in-process cache with tiny TTL. Detection: per-key sampling metrics.
- **Big key** (10 MB value blocking event loop): size caps, chunking, async serialization off-thread.
- **Thundering herd on expiry:** jittered TTLs, request coalescing (one rebuilds, rest wait), stale-while-revalidate.
- Slot migration while serving → keys move slot-by-slot with redirects (ASK) — live rebalancing without downtime.

### Follow-up questions
- **Why is Redis single-threaded fast?** All RAM ops, no lock contention, epoll event loop; network I/O dominates anyway; scale across cores via multiple shards per host.
- **How do clients discover topology changes?** MOVED/ASK redirects update client maps lazily; or config service push.
- **Persistence options?** Snapshot (RDB — fast recovery, loses delta) vs append-only log (AOF — durable, bigger/slower). For pure cache: none, warm from DB.
- **Cache stampede on cold start?** Warmup from snapshot or replay top-keys log; throttle DB with request coalescing.
- **Exact vs approximate LRU?** Exact needs a global list (pointer overhead + contention); sampled eviction gets ~same hit rate free. Approximation as an engineering principle.

### Tradeoffs to defend
AP semantics (it's a cache) vs durability (use a DB); smart clients (fast) vs proxy (operable); consistent hashing (smooth) vs fixed slots (controllable migration); exact (perfect) vs sampled (practical) LRU.

## 3. Advanced Version

- **Multi-region:** independent caches per region (caches shouldn't replicate cross-region — refill locally from regional DB); exception: shared expensive computations → async cross-region invalidation bus with LWW.
- **Failure:** full-cluster restart = stampede on DB → staged warmup, DB rate-limit shield, coalescing (the cache's failure mode is really a *database protection* problem).
- **Tiering:** RAM + NVMe second tier for long-tail keys (bigger effective cache, 10× cheaper per GB, slightly slower).
- **Multi-tenancy:** per-tenant memory quotas + eviction isolation, or noisy neighbors evict each other's data.
