# System Design — Index, Learning Order & Decision Tree

## Folder Map

- `Building_Blocks/` — 16 components (the vocabulary)
- `Concepts/` — CAP/consistency, estimation, interview framework (the grammar)
- `Case_Studies/` — 13 systems, Easy → Hard (the practice)

## Suggested Learning Order

**Phase 1 — Grammar first (½ day):**
1. `Concepts/requirements_gathering_framework.md` — read FIRST; it's the skeleton every session hangs on
2. `Concepts/back_of_envelope_estimation.md` — memorize the latency table + the 4-step method
3. `Concepts/CAP_theorem_and_consistency_models.md`

**Phase 2 — Core blocks (1–2 days):** Load Balancing (01) → Caching (02) → Replication & Sharding (03) → SQL vs NoSQL (04) → Message Queues (05) → then attempt **URL Shortener** and **Pastebin** with only these.

**Phase 3 — Remaining blocks (1–2 days):** Consistent Hashing (07) → Rate Limiting (08) → Partitioning (14) → Indexing (11) → Idempotency (13) → CDN (06) → WebSockets (16) → then **Rate Limiter, Key-Value Store, Chat, News Feed**.

**Phase 4 — Advanced (as time allows):** API Gateway (09), Micro/Mono (10), Locks (12), Search (15) → **Crawler, Notifications, Ride-Sharing** → Hard tier last (each Hard study mostly *recombines* earlier blocks — read them as composition exercises).

**Time-crunched (2 days total)?** Phase 1 + blocks 01–05, 07, 13, 14 + URL Shortener, Chat, News Feed. That covers ~80% of what interviews touch.

## Tiered Priority

**Tier 1 — must-master (in almost every interview):**
Load Balancing · Caching · Replication & Sharding · SQL vs NoSQL · Message Queues · CAP/consistency · Estimation · the interview framework. Case studies: URL Shortener, News Feed, Chat.

**Tier 2 — important (very common):**
Consistent Hashing · Rate Limiting · Partitioning strategies · Idempotency · CDN · WebSockets · Indexing. Case studies: Rate Limiter, Key-Value Store, Notification System, Ride-Sharing.

**Tier 3 — advanced/differentiating:**
Distributed Locks · Search internals · API Gateway details · Micro-vs-mono nuance. Case studies: Crawler, Distributed Cache, Video Platform, Payments, Job Scheduler.

## Master Tradeoff / Decision Reference (pre-interview scan)

| Need | Reach for | You pay |
|---|---|---|
| Strong consistency | SQL/quorums/consensus (CP) | Latency (coordination), availability under partition |
| Always-on availability | AP store, async replication | Stale reads, conflict resolution |
| Read-heavy scale | Cache (cache-aside) + read replicas | Staleness, invalidation complexity, cold-start |
| Write-heavy scale | LSM store (Cassandra), queues, sharding | Weak cross-row txns, eventual consistency |
| Storage > 1 node | Sharding/partitioning | Cross-shard queries/txns, resharding ops |
| Bursty traffic | Message queue buffer | Results arrive async, consumer lag |
| Low global latency | CDN + multi-region replicas | Cost, consistency across regions |
| No duplicate side effects | Idempotency keys | Key storage + atomic check discipline |
| Exclusive access | Prefer DB atomic ops → locks last | TTL/fencing complexity, lock service SPOF |
| Server→client push | WebSocket (2-way) / SSE (1-way) | Stateful connections, reconnect logic |
| Smooth node add/remove | Consistent hashing (vnodes) | Statistical (not perfect) balance |
| Full-text search | Inverted index (Elasticsearch) | Sync lag from source of truth, another cluster |
| Fan-out to millions | Push (precompute) + pull for hot publishers | Write amplification vs read latency — hybrid |
| Protect from abuse | Token-bucket rate limiter at gateway | Redis hop latency, fail-open/closed choice |
| Exactly-once | At-least-once + idempotency = exactly-once *effect* | Dedup state; true exactly-once impossible |

**Consistency quick-picks:** money/inventory/locks → linearizable · social/feeds/counts → eventual · user-sees-own-action → read-your-writes session guarantee.

**Latency anchors:** RAM 100ns · SSD 100µs · same-DC RTT 0.5ms · cross-continent RTT ~100ms+ · Redis ~0.5ms · indexed SQL ~1–10ms. 1M req/day ≈ 12 QPS.

## Who Asks What (rough map)

- **Meta:** News Feed, Chat/Messenger, Instagram-style media, live comments — fan-out & social graph heavy
- **Google:** Search-adjacent (crawler, indexing), YouTube, Maps-ish geo, Drive — data pipelines & scale
- **Amazon:** retail (inventory, orders, carts), marketplace, AWS-flavored infra (queue, KV store) — consistency & partitioning
- **Netflix/streaming:** video platform, CDN strategy, recommendations pipeline
- **Uber/Lyft/DoorDash:** dispatch/geo matching, real-time location, surge — geospatial + real-time
- **Stripe/PayPal/fintech:** payments, ledgers, idempotency, reconciliation — correctness obsession
- **Slack/Discord/WhatsApp:** chat, presence, WebSocket fleets
- **Startups generally:** URL shortener/rate limiter/notification tier — testing fundamentals + judgment about NOT over-engineering
- **Infra teams anywhere:** KV store, distributed cache, job scheduler, rate limiter

Any company can ask anything — the ladder here transfers; the map is just prior probability.

## How to Use the Case Studies

Don't read passively. For each: (1) read only the requirements, (2) design it yourself on paper using the framework file, (3) compare against the walkthrough, (4) note which *building block choices* you missed — the blocks are the transferable part. Every case study header lists its blocks; after 6–8 studies you'll see the same ~10 blocks recombining, and unseen problems stop being scary.
