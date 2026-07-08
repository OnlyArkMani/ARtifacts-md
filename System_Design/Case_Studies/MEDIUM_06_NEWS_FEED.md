# Case Study: News Feed (Medium)

**Building blocks used:** Caching (heavy) · Message Queues (fan-out) · Data Partitioning · Replication · CDN (media) · SQL vs NoSQL

The heart of this problem is one decision: **fan-out on write vs fan-out on read.** Everything else supports it.

---

## 1. Basic Version (single server)

- `POST /posts` → insert into posts table.
- `GET /feed` → `SELECT * FROM posts WHERE author_id IN (my_followees) ORDER BY created_at DESC LIMIT 20`.

That IN-query with a join on follows is fan-out-on-read in its rawest form. Works until followee counts and QPS grow.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Post (text+media); follow users; view reverse-chron feed of followees. (Ranking, ads, stories = extensions.)

### Non-functional
- 200M DAU; read:write ~100:1 (feed loads vs posts)
- Feed latency <200ms p99; eventual consistency fine (a post appearing after 10s is OK — say this, it unlocks the whole design)
- Availability over consistency (AP)

### Capacity estimate
- Writes: 200M × 2 posts/day ≈ 4k/s. Reads: 200M × 20 loads/day ≈ **40k/s, peak >100k/s**
- Fan-out volume: 4k posts/s × avg 200 followers = **800k feed-inserts/s** — this number is why fan-out needs queues and why celebrities break it
- Feed cache: 200M users × 400 post-IDs × 8B ≈ 640 GB — fits a Redis cluster ✓

### High-level design (hybrid fan-out — the expected answer)

```
 POST /posts                                GET /feed
     │                                          │
     ▼                                          ▼
┌──────────┐   ┌───────┐   ┌──────────────┐  ┌──────────────┐
│ Post svc │──▶│ Kafka │──▶│ Fanout workers│  │ Feed service │
└────┬─────┘   └───────┘   │ push post_id  │  └──────┬───────┘
     │                     │ to followers' │         │ read my feed list
     ▼                     │ feed lists    │         ▼
 Posts DB                  └──────┬────────┘   Redis: feed:user42 =
 (part. by post_id)               ▼            [postID…] (top ~400)
 + post cache            Redis feed lists           │
                                                    ▼ hydrate
                         celebrity posts NOT   post cache / DB
                         fanned out — merged
                         at read time
```

- **Fan-out on write (push):** when a normal user posts, workers insert the post ID into each follower's cached feed list. Feed read = one Redis fetch + hydrate. Fast reads, write amplification.
- **Fan-out on read (pull):** for **celebrities** (>~100k followers), don't push to millions of lists; at read time, merge the user's precomputed list with recent posts from any celebrities they follow.
- **Hybrid** is the standard answer; state the threshold idea explicitly.

### Deep dive 1: the celebrity (hot key / write amplification) problem
100M-follower account posts → pure push = 100M list inserts (minutes of delay, huge queue spike). Pure pull for everyone = every feed read does K queries for K followees. Hybrid: push for the many small accounts (cheap, read-fast), pull for the few huge ones (bounded merge cost — users follow few celebrities). Also cache celebrity recent posts hard — they're read by millions (replicated hot cache).

### Deep dive 2: feed storage & hydration
Feed list = capped list of post IDs (~400) in Redis, not full posts (dedupe content, small memory). Hydration: batch-get post bodies from a post cache (cache-aside over posts DB). Pagination via cursor (last seen post ID / timestamp, not offset). Inactive users' lists evicted (LRU) and rebuilt on demand via pull — self-healing cold path.

### Bottlenecks & fixes
- Fan-out queue backlog on viral moments → scale workers horizontally (partition by follower ID), tolerate delay (eventual by design).
- Redis feed cluster node loss → replicas; or rebuild lists lazily via pull path (degraded latency, no outage).
- Post cache stampede on a viral post → request coalescing, replicated hot keys.
- Media bandwidth → object storage + CDN (never in this pipeline).

### Follow-up questions
- **How would ranking change this?** Push candidate IDs, rank at read time (features fresh) — precomputing ranked feeds goes stale; ranking service scores the hydrated candidates. Mention two-stage: candidate retrieval → scoring.
- **New follow → old posts in feed?** On follow, backfill by pulling that user's recent posts into the feed list (async).
- **Unfollow/block?** Filter at read time (cheap check) rather than scrubbing lists synchronously.
- **Why eventual consistency OK here but not in chat?** Feed = discovery surface, no one knows what's missing; chat = direct communication, gaps are visible. Requirement-driven consistency — strong interview point.
- **Deletes?** Tombstone the post; filter at hydration (IDs in lists may dangle briefly).

### Tradeoffs to defend
Push (read latency) vs pull (write cost) vs hybrid; IDs-in-cache vs full-posts-in-cache (memory vs hydration hop); eventual consistency (enables everything) vs freshness.

## 3. Advanced Version

- **Multi-region:** posts replicate async cross-region; fan-out runs per-region for local followers; feed staleness between regions acceptable (AP).
- **Failure:** entire fan-out pipeline down → reads fall back to pull (slower, correct) — design has a natural degraded mode; queues buffer the backlog and drain.
- **ML ranking pipeline:** impressions/engagement events → Kafka → feature store; candidate generation (feed lists + follows graph + trending) → ranker. Feed becomes a recommender system — flag it, don't build it unprompted.
- **Ads/injection:** separate ad-selection service merged at hydration time.
