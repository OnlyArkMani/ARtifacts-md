# Case Study: Notification System (Medium)

**Building blocks used:** Message Queues/PubSub (core) · Rate Limiting · Idempotency (dedup) · Data Partitioning · Caching (prefs/tokens) · API Gateway

A pure **asynchronous pipeline** problem: fan-in from many producer services, fan-out to millions of devices across channels (push/SMS/email), with reliability, dedup, and user-respect (prefs, quiet hours, rate caps).

---

## 1. Basic Version (single server)

`POST /notify {user_id, type, message}` → look up device token / email → call FCM/APNs/SES/Twilio synchronously → return.

Breaks immediately: provider slowness blocks callers, no retries, no prefs, no dedup, no bulk sends. The whole scaled design is "make this async and reliable."

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Multiple channels (push, SMS, email, in-app); triggered by internal services (single + bulk/campaign); user preferences & opt-outs; templates; delivery tracking.

### Non-functional
- Scale: 100M notifications/day, campaign bursts of 10M+ in minutes
- **At-least-once with dedup** (never spam twice for retries; missing a security alert is worse than missing a like)
- Latency: security/OTP = seconds (priority lane); marketing = minutes OK
- Provider outages must not lose notifications

### Capacity estimate
100M/day ≈ 1.2k/s average, but bursts dominate: a 10M campaign in 10 min = **17k/s** → queues absorb; workers autoscale. Token/prefs store: 100M users × ~1 KB = 100 GB — cached KV.

### High-level design

```
Producer services (order svc, security svc, marketing…)
        │  notify(user, template, data, priority, dedup_key)
        ▼
┌─────────────────┐    validate, dedup, prefs/opt-out check,
│ Notification API │──▶ rate-cap per user, template render
└────────┬────────┘
         ▼
   Kafka topics by channel × priority
   [push.high] [push.low] [email] [sms]
         │
   ┌─────▼──────┐  channel workers: batch, retry w/ backoff,
   │ Push worker│─▶ APNs/FCM     ──▶ delivery receipts
   │ Email wrkr │─▶ SES/SendGrid ──▶ bounces → suppression list
   │ SMS worker │─▶ Twilio        ──▶ DLQ after N retries
   └────────────┘
         ▼
   Status store (per notification lifecycle) + analytics stream
```

### Deep dive 1: reliability & dedup
- Producer → API: sync ack after enqueue (Kafka persisted) — from here, delivery is the system's responsibility.
- Workers: at-least-once consumption → **idempotency via dedup key** (`user + event_id`): check-and-set in Redis/KV before send; TTL-bounded. 
- Provider fails → exponential backoff retries → **DLQ** after N attempts (inspect/replay); provider *outage* → circuit breaker, messages wait in queue (that's the point of queues), optional channel failover (push fails → email for critical).
- **Priority lanes:** separate topics/worker pools so a 10M marketing blast never delays an OTP — the queue-per-priority pattern is the key insight here.

### Deep dive 2: user respect layer (what distinguishes good answers)
Before enqueue: preferences (per-channel, per-category opt-outs), quiet hours (timezone-aware scheduling — hold until 9am local), **per-user rate cap** (max N/day, collapse similar into digests), suppression lists (bounced emails, invalid tokens — feedback loops from providers prune these). All this data cached (read on every notification) with async invalidation on user updates.

### Bottlenecks & fixes
- Campaign fan-out (one event → 10M user notifications): a **fan-out service** expands audience queries into per-user messages in batches, streaming into the queue — never one giant synchronous loop.
- Token invalidation churn → process provider feedback (APNs/FCM responses) to clean the token store continuously.
- Status writes (100M/day lifecycle updates) → write-optimized store / streaming aggregation, not per-row OLTP updates.
- Hot user (gets everything) → per-user rate cap + digesting solves UX and load together.

### Follow-up questions
- **Exactly-once delivery?** Impossible end-to-end (provider may deliver after your timeout). At-least-once + dedup key + provider-side collapse IDs ≈ exactly-once *effect*.
- **Ordering?** Generally unnecessary across notifications; per-user ordering if needed → partition queues by user_id.
- **How do in-app notifications differ?** They're just rows in a store the client polls/streams (see WebSockets) — no external provider, much simpler.
- **Scheduled/delayed sends?** Scheduler service (see Job Scheduler case study) feeding the same pipeline at fire time.
- **How do you test/preview a 10M campaign safely?** Staged rollout (1% → monitor → ramp), sample sends, kill switch on the campaign ID.

### Tradeoffs to defend
Queue-based async (reliability, burst absorption) vs sync (simplicity, tight latency) — async wins here; per-channel queues (isolation, priority) vs one queue (simplicity); dedup TTL length (memory vs protection window); channel failover (reach) vs annoyance.

## 3. Advanced Version

- **Multi-region:** route sends from region nearest the user; queues and token stores replicated per region; dedup keys need region-consistent store for global events (or home each user to a region — simpler).
- **Failure:** region down → replay from Kafka in surviving region using user-homing metadata; accept duplicate-window risk during failover (dedup keys mitigate).
- **Smart delivery:** send-time optimization (per-user best hour), frequency modeling, channel selection by engagement — ML layer on top of the same pipeline.
- **Compliance:** GDPR consent enforcement at the API layer (hard gate), audit trail of every send (append-only log).
