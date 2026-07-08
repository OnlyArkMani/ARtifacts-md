# Case Study: URL Shortener (Easy)

**Building blocks used:** Caching · Replication & Sharding · SQL vs NoSQL · CDN (optional) · Rate Limiting · Load Balancing · Indexing

---

## 1. Basic Version (single server)

API:
- `POST /urls {long_url}` → `{short_code}` (e.g., `abc123`)
- `GET /{short_code}` → HTTP **302** redirect to long_url

Core logic — generating the code:
- **Auto-increment ID → Base62 encode** (0-9a-zA-Z). ID 125 → `cb`. 7 chars covers 62⁷ ≈ 3.5 trillion. Deterministic, no collisions, but codes are guessable/sequential.
- **Random/hash:** take 7 chars of md5(url+salt); must check for collision and retry. Not guessable.

One table: `(short_code PK, long_url, created_at, user_id, expires_at)`, index on short_code. Done — this works to thousands of QPS.

**301 vs 302:** 301 (permanent) lets browsers cache → less load but you lose click analytics and can't change the target. 302 keeps every hit visible. Pick per requirement — classic follow-up.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Shorten URL; redirect; optional custom alias; optional expiry. (Out of scope: analytics dashboards, auth.)

### Non-functional
- 100M new URLs/month; read:write ≈ 100:1
- Redirect latency <100ms (it's in the critical path of *someone else's* page)
- High availability — redirects must basically never fail (favor AP; a slightly stale mapping is impossible anyway since mappings are immutable)

### Capacity estimate
- Writes: 100M / (30×100k s) ≈ **40/s**. Reads: ≈ **4k/s**, peak ~10k/s.
- Storage: 100M × 500 B/mo ≈ 50 GB/mo → ~3 TB over 5 yrs. Small!
- Conclusion: read-heavy, tiny data → **cache-first architecture**; sharding for availability, not capacity.

### High-level design

```
                   ┌────────────┐
 Client ──────────▶│    LB      │
                   └─────┬──────┘
                   ┌─────▼──────┐   read path: cache → DB on miss
                   │ App servers│──▶ Redis (hot codes, ~20% = most hits)
                   └─────┬──────┘        │ miss
        write path       │               ▼
   ┌──────────────┐      │         ┌──────────┐
   │ ID generator │◀─────┘         │ DB (KV)  │ code → url
   └──────────────┘                └──────────┘  replicated
```

### Deep dive 1: ID generation at scale (the classic)

Single auto-increment DB = SPOF + bottleneck. Options:
- **Range allocation (recommended answer):** a coordinator hands each app server a block of IDs (e.g., 1M at a time) via an atomic counter; servers assign from their block in memory → no per-request coordination. Lost blocks on crash = wasted IDs, harmless.
- **Snowflake IDs:** timestamp + machine-id + sequence → 64-bit, no coordination, roughly sortable. Slightly longer codes.
- **Pre-generated key pool:** offline job fills a table of unused random codes; servers claim batches. Unguessable codes for free.

### Deep dive 2: read path
Cache-aside Redis keyed by code (mappings are immutable → **no invalidation problem**, infinite TTL, LRU eviction). Hit rate should exceed 90% given Zipfian access. DB: any replicated KV store or even Postgres — data fits comfortably; choose DynamoDB-style KV if you want multi-region + zero-ops.

### Bottlenecks & fixes
- Cache node dies → cold reads hammer DB → use Redis cluster (replicas), warm standby.
- One viral link (hot key) → it's cached; add local in-process cache on app servers for the top ~100 codes.
- Malicious mass-shortening → rate limit per API key/IP at the gateway.
- Redirect service global latency → run read replicas + cache in multiple regions (mappings immutable ⇒ replication is trivially safe); GeoDNS routes users.

### Follow-up questions to expect
- **301 vs 302?** (above — analytics vs load tradeoff)
- **How to prevent guessing/enumeration?** Random codes or encrypt the counter; don't use raw sequential Base62.
- **Custom aliases?** Same table, uniqueness check at write; reserve/deny-list offensive words.
- **Expiry?** TTL column checked on read + lazy/batch cleanup job; don't do synchronous deletes.
- **Analytics?** Log click events to a queue (Kafka) → async aggregation. Never write counters synchronously in the redirect path.

### Tradeoffs to defend
Base62-of-counter (simple, no collisions, guessable) vs random (private, collision checks); 302 (analytics) vs 301 (efficiency); SQL (simple, plenty for this scale) vs KV (multi-region ease) — both defensible, say why.

## 3. Advanced Version

- **Multi-region active-active:** immutable mappings make this easy — write region-locally with region-prefixed ID blocks (no collisions), replicate async; reads always safe.
- **Failure handling:** if DB is down, serve cache-only (degraded writes, working reads) — availability preserved.
- **Abuse:** URL scanning (malware/phishing) via async check + block-list; per-key quotas.
- **Analytics pipeline:** click events → Kafka → stream aggregation (counts, geo, referrer) → OLAP store; discussed fully async from redirect path.
