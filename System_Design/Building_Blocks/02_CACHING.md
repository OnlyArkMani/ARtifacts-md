# Caching

## 1. What Problem It Solves

Databases and computation are slow and expensive; memory is fast. A cache keeps frequently-read data close (in RAM) so most reads never touch the database. It's the single highest-leverage tool for read-heavy systems — a 90% cache hit rate cuts database load by 10×.

## 2. How It Works

Cache sits between the app and the source of truth. On read: check cache → hit (return, ~1ms) or miss (fetch from DB, store in cache, return).

### Write strategies

| Strategy | How it works | Consistency | Write speed | Risk |
|---|---|---|---|---|
| **Cache-aside** (lazy) | App reads cache; on miss, reads DB and populates cache. Writes go to DB, cache entry invalidated/updated. | Can serve stale data briefly | Fast (DB only) | Stale reads; thundering herd on miss |
| **Write-through** | Writes go to cache, cache writes synchronously to DB | Strong (cache = DB) | Slower (two writes) | Write latency; caches cold data |
| **Write-back** (write-behind) | Writes go to cache only; flushed to DB async in batches | Cache ahead of DB | Fastest | **Data loss if cache dies before flush** |

Cache-aside is the default answer in interviews; write-back only when write throughput matters more than durability (e.g., counters, likes).

### Eviction policies (cache is full — who leaves?)

- **LRU** (Least Recently Used) — evict the oldest-touched entry. Default choice; matches most access patterns.
- **LFU** (Least Frequently Used) — evict the least-accessed. Better for stable hot sets; slow to adapt to shifting patterns.
- **FIFO** — evict oldest inserted. Simple, rarely optimal.
- **TTL** — entries expire after a fixed time. Bounds staleness; combine with LRU.

### Classic problems & fixes

- **Thundering herd / cache stampede:** hot key expires, thousands of requests hit DB simultaneously. Fix: per-key lock so only one request rebuilds; serve stale while revalidating; jittered TTLs.
- **Hot key:** one key gets extreme traffic. Fix: replicate the key across nodes, or cache in-process (local cache layer).
- **Cache penetration:** requests for keys that never exist bypass cache to DB every time. Fix: cache negative results, or Bloom filter in front.
- **Invalidation:** "the hardest problem in CS." Options: TTL (bounded staleness), explicit invalidate-on-write, event-driven invalidation via CDC/pub-sub.

## 3. Tradeoffs

**Pros:** massive latency win (µs–ms vs 10s of ms), shields DB from load, cheap relative to scaling the DB.

**Cons:** stale data, invalidation complexity, cold-start after restart, extra infrastructure, cache and DB can disagree.

**When NOT to use:** write-heavy workloads with few repeat reads (low hit rate = pure overhead), data requiring strict consistency (bank balances at moment of transaction), tiny datasets already fast to query.

## 4. Diagram

```
           read (1. check)      hit ──▶ return (fast)
  App ───────────────▶ ┌───────┐
   │                   │ Cache │  (Redis/Memcached)
   │    miss (2.)      └───┬───┘
   ▼                       │ 3. populate on miss
 ┌────┐  ◀─────────────────┘
 │ DB │
 └────┘
 Write (cache-aside): App ──▶ DB, then invalidate cache key
```

## 5. Common Interview Follow-ups

**Q: How do you keep cache and database consistent?**
A: Perfect consistency is impossible without transactions across both. Practical pattern: write DB first, then delete (not update) the cache key; next read repopulates. Add TTL as a safety net. For stricter needs, use CDC (change data capture) streams to invalidate.

**Q: Redis vs Memcached?**
A: Redis: rich data structures (sorted sets, lists), persistence, replication, pub/sub, single-threaded but fast. Memcached: simpler, multithreaded, pure key-value, slightly better raw throughput for plain strings. Default answer: Redis, for versatility.

**Q: Where can caching live?**
A: Multiple layers — browser cache, CDN (static assets), API gateway cache, application in-process cache, distributed cache (Redis), DB internal buffer pool. Each layer trades freshness for latency.

**Q: What cache hit rate is "good"?**
A: Depends on skew; 80–95%+ typical with Zipfian access. What matters is DB load with vs without cache; measure and size the cache to hold the hot set (often ~20% of data serves 80% of reads).

**Q: What happens when the cache cluster restarts?**
A: Cold cache → all requests hit the DB → potential meltdown. Mitigate: cache warming (preload hot keys), gradual traffic ramp, request coalescing, DB rate limiting as backpressure.

---
**Used in case studies:** URL Shortener, News Feed, Chat, YouTube, Distributed Cache (builds one from scratch).
