# Rate Limiting

## 1. What Problem It Solves

Without limits, one abusive client (or a bug, or a scraper) can consume all your capacity and take the service down for everyone. Rate limiting caps how many requests a client can make per time window — protecting backends, enforcing fair use and pricing tiers, and blunting brute-force/DoS attempts.

## 2. How It Works

Identify the client (API key, user ID, IP), count their recent requests, and reject (HTTP **429**, ideally with `Retry-After`) when over limit.

### Algorithms

| Algorithm | Mechanism | Bursts? | Memory | Weakness |
|---|---|---|---|---|
| **Token bucket** | Bucket holds up to B tokens, refills at r/sec; each request takes one; empty → reject | Allows bursts up to B | O(1)/key | Burst may still spike backend |
| **Leaky bucket** | Requests enter a queue drained at fixed rate | No — smooths to constant rate | O(queue) | Adds queueing delay; bursts wait or drop |
| **Fixed window** | Counter per window (e.g., per minute), reset at boundary | — | O(1)/key | **Boundary burst:** 2× limit possible straddling the reset |
| **Sliding window log** | Store timestamp of every request; count those in last window | Exact | O(requests) — expensive | Memory heavy |
| **Sliding window counter** | Weighted blend of current + previous window counts | Approximate, good | O(1)/key | Slight approximation |

**Interview default:** token bucket — simple, O(1), tolerates natural burstiness. (This is what many real APIs use.)

### Distributed rate limiting

Multiple API servers must share counts → central store (Redis: `INCR` + `EXPIRE`, or Lua script for atomic token bucket). Tradeoffs:

- Redis hop adds latency → keep limiter data in-memory per node with a *local allowance* synced periodically (slightly loose but fast), or accept small over-admission.
- Race conditions → atomic ops (Lua scripts) rather than read-modify-write.
- Where to enforce: usually at the **API Gateway** (see `09_API_GATEWAY.md`) so backends never see rejected traffic.

## 3. Tradeoffs

**Pros:** protects availability, fair usage, cost control, security throttling; O(1) algorithms are nearly free.

**Cons:** legitimate users get throttled during spikes, distributed accuracy vs latency tension, choosing the right key is tricky (IP punishes NAT'd users; user ID doesn't stop pre-auth abuse).

**When NOT to use (or use carefully):** internal trusted service-to-service traffic (prefer load shedding/circuit breakers), or as your only defense against DDoS (you need edge/network-level protection too).

## 4. Diagram

```
             ┌──────────────────────────┐
 Client ───▶ │ API Gateway              │ ──▶ Backend services
             │  ┌────────────────────┐  │
             │  │ Token bucket (Redis)│ │      429 + Retry-After
             │  │ key: user:42        │ │ ──▶  when bucket empty
             │  │ tokens: 3/10 r=5/s  │ │
             │  └────────────────────┘  │
             └──────────────────────────┘

Token bucket:  refill r/sec ──▶ [●●●○○○○○○○] cap B
                                  ▲ request consumes 1; empty → reject
```

## 5. Common Interview Follow-ups

**Q: Token bucket vs leaky bucket — which and why?**
A: Token bucket if occasional bursts are acceptable (most APIs — users are bursty). Leaky bucket if downstream needs a strictly smooth rate (e.g., feeding a fragile legacy system).

**Q: What's wrong with fixed windows?**
A: Boundary problem: limit 100/min → client sends 100 at 0:59 and 100 at 1:01 = 200 in 2 seconds. Sliding window counter fixes this cheaply.

**Q: How do you rate limit across many servers?**
A: Centralized counters in Redis with atomic Lua scripts; or local buckets with periodic sync for lower latency at the cost of slight over-admission. State the accuracy-vs-latency tradeoff.

**Q: What should the client experience be?**
A: 429 status, `Retry-After` header, and rate-limit headers (`X-RateLimit-Remaining`) so well-behaved clients back off. Clients should implement exponential backoff with jitter.

**Q: Rate limiting vs load shedding vs circuit breaker?**
A: Rate limiting = per-client fairness (proactive). Load shedding = drop excess when *the server* is overloaded regardless of who (reactive). Circuit breaker = *caller* stops calling a failing dependency. Complementary.

---
**Used in case studies:** Rate Limiter (dedicated case study), API Gateway contexts, Notification System, Web Crawler (politeness limits per domain).
