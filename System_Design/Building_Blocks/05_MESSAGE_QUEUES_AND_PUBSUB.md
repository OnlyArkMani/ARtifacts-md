# Message Queues & Pub/Sub

## 1. What Problem It Solves

Synchronous calls couple services: if B is slow or down, A fails too, and A can only go as fast as B. A message queue decouples them — A drops a message and moves on; B processes at its own pace. This buys you: **decoupling** (services don't know each other), **buffering** (absorb traffic spikes), **async work** (don't make users wait for emails/thumbnails), and **retry/durability** (work survives crashes).

## 2. How It Works

### Two messaging patterns

| | Message Queue (point-to-point) | Pub/Sub |
|---|---|---|
| Delivery | Each message consumed by **one** worker | Each message delivered to **all** subscribers |
| Use | Task distribution (resize this image) | Event broadcast (order_placed → email svc, analytics, inventory) |
| Examples | SQS, RabbitMQ | Kafka topics, SNS, Redis pub/sub |

### Core mechanics

- **Producer** appends messages; **broker** stores them durably; **consumer** pulls (or is pushed) messages and **acks** on success. Unacked messages are redelivered.
- **Kafka model:** topics split into **partitions** (ordering is per-partition only). Consumers in a **consumer group** split partitions among themselves — this is how consumption scales horizontally. Messages are retained for a time window (log, not queue) so multiple groups can read independently and replay.
- **Delivery semantics:**
  - *At-most-once* — fire and forget; can lose messages.
  - *At-least-once* — redeliver until acked; can duplicate. **The practical default** — pair with idempotent consumers (see `13_IDEMPOTENCY.md`).
  - *Exactly-once* — very expensive/limited; in practice = at-least-once + idempotency/dedup.
- **Dead-letter queue (DLQ):** message failing N retries is shunted aside for inspection instead of poisoning the queue forever.
- **Backpressure:** if producers outpace consumers, queue depth grows — monitor lag, autoscale consumers, or shed load.

## 3. Tradeoffs

**Pros:** failure isolation, spike absorption, independent scaling of producers/consumers, replayability (Kafka), natural fit for event-driven architecture.

**Cons:** eventual consistency (work happens *later*), harder debugging (where did my message go?), ordering only per-partition, duplicates to handle, one more stateful system to operate.

**When NOT to use:** when the caller needs the result *now* (auth checks, reads), simple CRUD apps with no async work, ultra-low-latency paths where broker hop is too slow.

## 4. Diagram

```
Queue (work distribution):            Pub/Sub (fan-out):

Producer ─▶ [ ▣ ▣ ▣ ▣ ] ─┬─▶ Worker1     order_placed
              queue      ├─▶ Worker2         │
   (each msg to ONE)     └─▶ Worker3    ┌────┼─────────┐
                                        ▼    ▼         ▼
                                     Email  Analytics  Inventory
                                     (each msg to ALL subscribers)

Kafka: topic ─ partition0 [1|2|3|4...] ─▶ consumer A   } consumer
             ─ partition1 [1|2|3...]   ─▶ consumer B   } group
```

## 5. Common Interview Follow-ups

**Q: How do you guarantee a message isn't lost?**
A: Producer waits for broker ack (replicated write), broker persists to disk with replication, consumer acks only *after* successful processing. Every hop must confirm.

**Q: How do you handle duplicate messages?**
A: Assume at-least-once. Make processing idempotent: dedup table keyed by message ID, upserts instead of inserts, or naturally idempotent operations ("set status=paid").

**Q: How do you preserve ordering?**
A: Global ordering doesn't scale. Get per-key ordering by partitioning on that key (e.g., all events for user X → same partition → processed in order by one consumer).

**Q: Queue keeps growing — what do you do?**
A: Check consumer health, scale out consumers (more partitions if needed), optimize processing, apply backpressure at producers, or shed/expire low-value messages. Monitor consumer lag as a first-class metric.

**Q: Kafka vs RabbitMQ vs SQS?**
A: Kafka — high-throughput event log, replay, streaming pipelines. RabbitMQ — flexible routing, per-message acks, classic task queues. SQS — zero-ops managed queue, default for AWS async work.

---
**Used in case studies:** Notification System (core), Web Crawler (URL frontier), Chat, News Feed fan-out, Payment System, Job Scheduler.
