# Case Study: Distributed Rate Limiter (Easy)

**Building blocks used:** Rate Limiting (algorithms) · Caching/Redis · Consistent Hashing · API Gateway · Load Balancing

---

## 1. Basic Version (single server)

In-process token bucket per client key:

```python
# bucket: {tokens: float, last_refill: time}
def allow(key, rate=10, burst=20):
    b = buckets[key]
    now = time()
    b.tokens = min(burst, b.tokens + (now - b.last_refill) * rate)
    b.last_refill = now
    if b.tokens >= 1:
        b.tokens -= 1
        return True     # allow
    return False        # 429
```

O(1) memory per key, lazy refill (no timers). Return `429` + `Retry-After` + `X-RateLimit-Remaining` headers. This is complete and correct for one node.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Limit requests per client (user ID / API key / IP) per rule (e.g., 100/min per user, different tiers); reject with 429; rules configurable without redeploys.

### Non-functional
- Must add **<1–2ms** latency (it's on every request)
- Highly available — limiter failure must not take down the API (decide: fail-open or fail-closed)
- Accuracy: slight over-admission acceptable; systematic bypass not
- Scale: say 100k RPS across the fleet, 10M distinct keys

### Capacity estimate
10M active keys × ~50 B state = 500 MB — fits in one Redis easily; QPS is the real constraint: 100k RPS → each check is 1 Redis round trip → a few shards for headroom. Counters are ephemeral → no durable storage needed.

### High-level design

```
          ┌─────────────────────────────┐
Client ──▶│ API Gateway / middleware    │──▶ backend services
          │   allow(key)?               │
          └───────────┬─────────────────┘
                      │ Lua script (atomic)
              ┌───────▼────────┐
              │ Redis cluster  │  key → {tokens, ts}, TTL
              │ shard by       │  rules cached from config svc
              │ hash(key)      │
              └────────────────┘
```

- Enforcement lives in the **gateway** so rejected traffic never reaches backends.
- Bucket state in **Redis**, sharded by consistent hashing on the client key → all checks for one key hit one shard (no cross-node coordination).
- The check-and-decrement runs as a **Lua script** — atomic, eliminating read-modify-write races.
- Rules (limits per tier/endpoint) come from a config service, cached in each gateway with a short TTL.

### Deep dive 1: algorithm choice
Token bucket (default): allows bursts to `burst`, O(1). Sliding-window counter if smoother enforcement needed: `count = curr_window + prev_window × overlap_fraction` — fixes fixed-window boundary bursts with O(1) memory. Mention leaky bucket only if downstream needs a strictly constant rate. (Full comparison: `../Building_Blocks/08_RATE_LIMITING.md`.)

### Deep dive 2: latency vs accuracy
A Redis hop per request may be too slow for some paths. Hybrid: each gateway keeps **local buckets** for a fraction of the limit (limit/N per node) or syncs deltas to Redis every ~100ms. Consequence: temporary over-admission up to the sync gap. State it explicitly: **strict accuracy needs centralized state; low latency prefers local state; hybrid tunes between them.**

### Bottlenecks & fixes
- Redis down → **fail-open** (allow all, protect availability; log loudly) is the usual choice — fail-closed only for security-sensitive limits (login attempts).
- Hot key (one huge client) → their shard gets hammered → local caching for that key + per-shard replicas.
- Clock skew across gateways → use Redis server time (`TIME` in the Lua script), not app clocks.
- Config change stampede → rules cached with jittered TTL.

### Follow-up questions
- **Fail-open or fail-closed?** Availability-critical APIs: open. Auth/fraud endpoints: closed. Say it depends and give both.
- **Race between two requests for the last token?** Atomic Lua script — check + decrement in one step.
- **How do clients behave well?** Rate-limit headers; exponential backoff + jitter on 429.
- **Throttle by IP or user?** User/API key post-auth (fair); IP pre-auth (stops anonymous abuse; NAT false-positives). Layer both.
- **Different limits per endpoint?** Composite bucket key: `{user}:{endpoint_class}`.

### Tradeoffs to defend
Token bucket vs sliding window (burst tolerance vs smoothness); centralized (accurate, +1ms) vs local (fast, approximate); fail-open (availability) vs fail-closed (protection).

## 3. Advanced Version

- **Multi-region:** don't synchronize buckets globally (latency kills you). Either split quota per region (global/N, resize by observed traffic) or enforce regionally + async global reconciliation for billing-grade accounting.
- **Tiered/dynamic limits:** adaptive limits that shrink under backend stress (load-shedding integration) — limiter becomes part of the overload-protection story.
- **Hierarchies:** per-user AND per-org AND global service cap — evaluate cheapest-to-reject first.
- **Observability:** emit throttle metrics per key/rule; top-N throttled clients dashboard — often how abuse is discovered.
