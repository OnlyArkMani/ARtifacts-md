# Case Study: Ride-Sharing Dispatch (Medium)

**Building blocks used:** WebSockets (location streams) · Geospatial indexing · Message Queues · Distributed Locks / atomic matching · Caching · Data Partitioning (geo) · Idempotency

Unique ingredients vs other case studies: **geospatial index** + **a matching problem with contention** (two riders, one driver).

---

## 1. Basic Version (single server)

- Drivers POST location every 4s → `drivers(driver_id, lat, lng, status)` in memory.
- Rider requests ride → naive nearest search: distance to every online driver (fine for one city, hundreds of drivers) → offer to nearest → driver accepts → trip starts.
- Trip state machine: `requested → matched → accepted → picked_up → completed`.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Rider requests ride; find nearby drivers; driver accepts; live trip tracking; basic pricing/ETA. (Cut: pool rides, surge details, payments — cross-ref Payment System.)

### Non-functional
- 10M drivers online peak, 1M ride requests/hour peak
- Matching latency < a few seconds; location updates flowing at high volume
- **A driver must never be double-dispatched** (consistency where it counts); location data itself is ephemeral/approximate (freshness > durability)

### Capacity estimate
- Location updates: 10M drivers / 4s = **2.5M updates/s** — dominates everything; must be memory-only, no DB writes per update
- Ride requests: 1M/hr ≈ 300/s — tiny by comparison
- Conclusion: the *location ingestion* pipeline and the *match* transaction are the two deep dives.

### High-level design

```
Drivers ─WS/UDP─▶ ┌────────────────┐    in-memory geo index
 (loc every 4s)   │ Location svc   │──▶ per city/region shard:
                  │ (gateway fleet)│    geohash/S2 cell → {drivers}
                  └────────────────┘    (Redis GEO or custom)
                                              ▲ query nearby
Rider ──────────▶ ┌────────────────┐          │
 "need ride"      │ Dispatch svc   │──────────┘
                  └──────┬─────────┘
                         │ offer → driver (via WS), atomic claim
                         ▼
                  Trip service (state machine, durable DB)
                         │ events → Kafka → pricing, ETA, analytics
```

### Deep dive 1: geospatial index — "find drivers near X"
Lat/lng B-tree indexes can't do 2D proximity well. Standard tools:
- **Geohash / S2 / H3 cells:** divide the world into hierarchical cells; index = `cell_id → set of drivers`. Nearby search = my cell + 8 neighbors (handles cell-boundary edge case), then exact distance sort on the small candidate set.
- Store in memory (Redis GEO does geohash+sorted-set internally), **sharded by region** — geo-partitioning is natural here: queries are inherently local. Hot cells (airport) → smaller cell granularity or split shards.
- Driver location = latest value only (overwrite, TTL for staleness) — no history in the hot path (history → Kafka → analytics store async).

### Deep dive 2: the match — avoiding double-dispatch
Two requests, same nearest driver, same moment. Options:
- **Atomic claim (recommended):** driver availability lives in one place (the region shard); claim = atomic compare-and-set `status: available → offered` (Redis Lua / single-threaded per-shard actor). Loser gets next-best driver. This is a *local* lock, cheap because geo-sharding means one driver belongs to one shard.
- Offer expires (driver doesn't accept in ~15s) → status resets, offer next driver (or offer to several with first-accept-wins — then acceptance is the atomic CAS instead).
- Trip creation is idempotent (request ID) — retries don't double-book.

### Bottlenecks & fixes
- Location firehose → don't persist per-update; batch to Kafka for offline; adaptive update frequency (moving fast = more frequent).
- One region shard hot (downtown Friday) → split cells across sub-shards; the matcher fans out.
- Gateway loss → drivers reconnect (jitter); locations self-heal in seconds (next update) — a nice property, say it.
- ETA/pricing compute → separate services consuming location streams, cached per cell.

### Follow-up questions
- **Why not store locations in PostGIS/DB?** 2.5M writes/s of ephemeral data; DB durability buys nothing for 4-second-lifetime values. Memory + async analytics stream.
- **Surge pricing?** Per-cell supply/demand ratio computed on the stream → price multiplier cached per cell; read at request time.
- **How does the rider see the driver moving?** Subscribe (WS) to that trip's driver location topic — pub/sub per trip; throttle to ~1 update/s for the map.
- **Offer to one driver or many?** Sequential (better driver experience, slower match) vs broadcast first-accept (fast, causes accept-races → CAS resolves). Real systems blend.
- **Driver on region boundary?** Query neighbor cells across shard boundary (scatter to ≤2 shards) or overlap cell ownership.

### Tradeoffs to defend
Memory geo-index (fast, lossy) vs durable store (pointless durability); geo-sharding (local queries, hot-cell risk) vs hash (even, kills locality); sequential offers (UX) vs broadcast (latency); strong consistency ONLY on the claim/trip state, eventual everywhere else — requirement-driven consistency is the flag to plant.

## 3. Advanced Version

- **Multi-region = multi-city naturally** — regions are independent (a Tokyo request never queries Paris). Global layer only for accounts/payments. Failure of one region's dispatch = local outage, not global.
- **Matching quality:** batch matching (collect requests for ~2s, solve assignment optimally — reduces ETA globally) vs greedy instant match; mention as optimization.
- **Failure handling:** trip state machine durable + event-sourced (Kafka) — any service can rebuild; exactly-once effects via idempotent transitions.
- **Pool/shared rides:** route-matching problem (much harder — vehicle routing); flag complexity.
