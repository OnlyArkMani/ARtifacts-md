# Case Study: Web Crawler (Medium)

**Building blocks used:** Message Queues (URL frontier) · Consistent Hashing · Caching (DNS, robots) · Data Partitioning · Rate Limiting (politeness) · Search/Inverted Index (downstream) · Bloom filters

The distinctive challenges: **politeness** (don't DoS websites), **dedup at billions-scale**, and **frontier prioritization** — it's a pipeline design, not a request/response system.

---

## 1. Basic Version (single machine)

```
seed URLs → queue → fetch page → parse HTML → extract links
              ▲                       │
              └── new, unseen URLs ◀──┘ (seen = hash set)
store page content; repeat
```

BFS over the web graph with a visited-set. Immediately hits: traps (infinite calendars), politeness, and scale of the seen-set. Those become the scaled design.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Crawl seeded portion of the web; store page content for indexing; re-crawl on freshness schedule; respect robots.txt.

### Non-functional
- Scale: 1B pages/month; storage for content
- **Politeness is a hard correctness constraint** (max ~1 concurrent req + delay per domain)
- Robustness: malformed HTML, traps, dead servers must never wedge the pipeline
- Freshness: prioritized re-crawling (news hourly, static pages monthly)

### Capacity estimate
- 1B pages / (30×86,400s) ≈ **400 pages/s sustained** (peaks higher)
- 400/s × ~100 KB avg ≈ 40 MB/s ingest; content: 1B × 100 KB = **100 TB/month** → object storage, compressed, dedup identical content
- Seen-set: billions of URLs → a hash set in one node's RAM won't fly → partitioned store + Bloom filter front

### High-level design

```
┌───────────────── URL Frontier ─────────────────┐
│ front: priority queues (freshness, PageRank…)  │
│ back:  per-domain FIFO queues + politeness     │
│        timer per domain (1 req / delay)        │
└──────────────┬─────────────────────────────────┘
               │ domain → fetcher via consistent hashing
        ┌──────▼──────┐   DNS cache · robots.txt cache
        │ Fetcher fleet│──▶ fetch page
        └──────┬──────┘
               ▼
        ┌─────────────┐  content hash → dedup (simhash for near-dup)
        │ Parser/     │──▶ content → object storage → indexer
        │ Extractor   │──▶ links → URL filter/normalizer
        └──────┬──────┘        │
               ▼               ▼
        seen? (Bloom + KV) ──no──▶ back into Frontier
```

### Deep dive 1: the URL frontier (the heart)
Two-level queue system (Mercator design):
- **Front queues** = priority (site importance, update frequency, depth). 
- **Back queues** = **one FIFO per domain**; a domain-to-queue map ensures all URLs of a domain serialize through one queue; each queue has a next-allowed-fetch timestamp (crawl-delay). Fetchers pull only from queues whose timer expired — **politeness enforced structurally**, not by convention.
- Partition domains across fetcher machines by consistent hashing → politeness state stays local to one machine per domain; no distributed coordination needed. (This is the elegant trick worth saying out loud.)

### Deep dive 2: dedup at scale
- **URL-seen test:** billions of entries. Bloom filter in RAM (no false negatives → never re-crawl misses; small false-positive rate → occasionally skip a new URL, acceptable) backed by a sharded KV for the authoritative set.
- **Content dedup:** exact — content hash (MD5) lookup; near-duplicate (boilerplate variants) — **simhash**, compare within hamming distance. Skip re-processing dup content.

### Bottlenecks & fixes
- DNS resolution at 400/s → local caching resolvers (DNS is often the sneaky bottleneck — good flag).
- Traps/infinite spaces → max depth, max URLs/domain, URL pattern heuristics, trap blacklists.
- Slow/huge responses → timeouts, size caps, streaming parse.
- One huge domain saturating a fetcher → politeness already throttles it; frontier prioritizes other domains meanwhile.
- Frontier storage (billions of queued URLs) → mostly on disk, head-of-queue in memory (queues are FIFO — sequential I/O friendly).

### Follow-up questions
- **How do you re-crawl for freshness?** Per-URL re-crawl interval adapted from observed change rate (exponential backoff on unchanged content); priority feeds the front queues.
- **How do you avoid crawling the same URL twice when two parsers extract it simultaneously?** Seen-check is a partitioned atomic set-if-absent (same idempotency pattern as everywhere).
- **JavaScript-rendered pages?** Headless-browser rendering farm for the subset that needs it — 10–100× cost, so classify first. 
- **How would you crawl politely AND fast for one giant site?** You can't — politeness caps per-domain throughput by design; overall speed comes from domain breadth.
- **Distributed consensus needed?** No — partition-by-domain removes shared state; only the seen-set is shared (sharded KV). Crawlers are embarrassingly parallel done right.

### Tradeoffs to defend
Bloom filter (RAM-cheap, rare false skip) vs exact set (heavy); BFS breadth (coverage) vs priority crawl (freshness/value); politeness (slow per domain) vs coverage speed (parallel domains); render-JS (complete) vs HTML-only (cheap).

## 3. Advanced Version

- **Multi-region/geo-distributed crawling:** fetch from region nearest the site (latency, some sites geo-block); frontier partitioning gains a geo dimension.
- **Failure:** fetcher dies → its domains reassign via consistent hashing (politeness timers rebuilt conservatively); frontier checkpointed; everything idempotent → at-least-once re-fetch is harmless.
- **Freshness at news-scale:** sitemap/feed ingestion + push signals (WebSub) instead of pure polling.
- **Downstream:** content → indexing pipeline (inverted index), link graph → PageRank — mention the handoff, don't design the search engine.
