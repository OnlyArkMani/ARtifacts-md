# The System Design Interview Framework

A 45-minute interview has a standard skeleton. Following it visibly is half the score — it shows you can drive an ambiguous problem. Budget below assumes 40 min of design time.

## Step 1: Functional Requirements (~5 min)

What does the system DO? Scope aggressively — the question is always too big.

- List core features, then explicitly cut: "I'll focus on shortening + redirecting; analytics and custom aliases are out of scope unless you want them."
- Ask about the *actors*: who uses it, from what clients (mobile/web/API)?
- Write 3–5 bullet features. Get interviewer nod before moving on.

**Good questions:** "Should URLs expire?" "One-on-one chat only, or groups?" "Do we need search or just browse?"

## Step 2: Non-Functional Requirements (~3 min)

How WELL must it do it? This is where the architecture actually comes from.

- **Scale:** DAU? QPS? Data size? Read-heavy or write-heavy? (ask, or propose numbers)
- **Latency:** what's the tolerance? (feed load <200ms; payment can take 2s)
- **Availability vs Consistency:** which does this domain favor? (say it in CAP terms — see `CAP_theorem_and_consistency_models.md`)
- **Durability:** can we ever lose data? (messages: no; metrics: maybe)
- Others when relevant: security, cost, compliance/data-residency.

## Step 3: Capacity Estimation (~5 min)

QPS (avg + peak), storage/year, bandwidth, cache size — see `back_of_envelope_estimation.md`. Keep it brisk, and end each number with a conclusion ("40k reads/s ⇒ we'll need a cache layer and read replicas").

## Step 4: API & Data Model (~5 min)

- 3–6 core endpoints: `POST /urls {long_url} → {short_url}`, `GET /{code} → 302`.
- Core entities + fields + the critical access patterns. Choose DB *after* seeing access patterns (justify: "key-value lookups at high scale, no joins → DynamoDB").

## Step 5: High-Level Design (~10 min)

Draw the boxes: client → LB/gateway → services → cache → DB → queue → workers. Walk one request through the diagram end-to-end, out loud, for the main read path AND the main write path. Keep it boring and correct — cleverness comes in step 6.

## Step 6: Deep Dive (~10 min)

Interviewer picks a component, or you propose the hardest 1–2. This is the differentiator round:

- Name the building block you're reaching for and *why* (e.g., "fan-out on write via a queue, because reads dominate 100:1").
- Discuss failure modes: what if this cache dies / this queue backs up / two workers grab the same job?
- Quantify: "each gateway holds ~500k connections, so 10M concurrent needs ~20 nodes + headroom."

## Step 7: Bottlenecks, Scaling & Wrap-up (~5 min)

Sweep the diagram for: single points of failure (fix: replication), hot spots (fix: better partition key, cache, salting), unbounded growth (fix: TTL/archival), thundering herds (fix: jitter, coalescing). Then multi-region if time. Close with tradeoffs you consciously made: "I chose eventual consistency on the feed for latency; if the interviewer's product needed strict ordering, I'd switch X."

## Cross-Cutting Behaviors That Score Points

- **Drive:** propose, don't wait to be dragged. Check in at each transition: "Shall I go deeper here or move on?"
- **Tradeoffs, always:** never present a choice without its cost. "Redis pub/sub is fast but loses messages on restart; Kafka is durable but adds latency."
- **Numbers anchor decisions:** justify components with the estimate from step 3.
- **Evolve, don't gold-plate:** start simple, scale when a number demands it. Saying "a single Postgres handles this until ~X, so I wouldn't shard yet" is a *strong* signal.

## Anti-Patterns (instant signal-killers)

Jumping to boxes before requirements; name-dropping tech without a reason ("we'll use Kafka" — for what?); designing for Google scale when asked for a startup; monologuing without check-ins; ignoring failure cases; refusing to commit to a choice.

## 30-Second Template to Memorize

> "Let me clarify functional requirements → then non-functional (scale, latency, consistency) → quick capacity estimate → API + data model → high-level diagram → deep-dive where you'd like → finish with bottlenecks and tradeoffs."

Saying this skeleton out loud in the first minute sets the frame and buys you control of the interview.
