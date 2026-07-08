# Case Study: Chat Application (Medium)

**Building blocks used:** WebSockets · Message Queues/PubSub · Data Partitioning (compound keys) · Replication · Caching · Idempotency · Push notifications

---

## 1. Basic Version (single server)

- Clients connect via WebSocket to one server; server holds `{user_id → connection}` in memory.
- Send: A → server → persist message → look up B's connection → push. B offline → store; deliver on reconnect (fetch undelivered since last seq).
- Table: `messages(conversation_id, message_id, sender, content, created_at)`, ordered by insertion.

This genuinely works to ~tens of thousands of concurrent users.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
1:1 chat; group chat (say ≤500 members); delivery/read receipts; online presence; message history; offline push notifications. (Cut: E2E encryption, media — mention as extensions.)

### Non-functional
- 50M DAU, 10M concurrent connections
- Delivery latency <100ms online; **no message loss** (durability!); message **ordering within a conversation**
- Consistency: per-conversation ordering matters; cross-conversation doesn't. Availability over global consistency (AP-leaning, but writes must be durable).

### Capacity estimate
- 50M DAU × 40 msgs/day = 2B msgs/day ≈ **20k msgs/s avg, ~100k peak**
- Storage: 2B × 100 B ≈ 200 GB/day → partition + archive tiers
- Connections: 10M concurrent / ~500k per gateway ≈ **20 gateway nodes** + headroom
- Conclusions: queue-backed write path; partitioned message store; WebSocket gateway fleet with a routing layer.

### High-level design

```
 UserA ◀─WS─▶ ┌─────────┐    ┌──────────────┐   ┌───────────────┐
              │Gateway 1│───▶│ Chat service │──▶│ Message store │
 UserB ◀─WS─▶ │Gateway 7│◀┐  │ (stateless)  │   │ Cassandra:    │
              └─────────┘ │  └──────┬───────┘   │ part=conv_id  │
   session registry       │         │           │ sort=msg_id   │
   user→gateway (Redis) ──┘   pub/sub or queue  └───────────────┘
                              per conversation
                                    │ offline?
                                    ▼
                            Push service → APNs/FCM
```

**Message flow:** A sends (with client-generated dedup ID) → gateway → chat service: (1) assign conversation-scoped sequence number, (2) persist (this is the durability point — ack A only after), (3) route to recipients' gateways via registry lookup or pub/sub channel, (4) offline recipients → push notification + marked undelivered.

### Deep dive 1: message store & ordering
Cassandra-style wide-column: **partition key = conversation_id, clustering key = message_id (time-ordered)** → "last 50 messages of conversation X" = one-partition read; writes distribute across conversations. Message IDs: per-conversation monotonic sequence (from the partition's coordinator or a Snowflake-ish ID) — gives ordering + gap detection for clients. Global ordering across conversations: explicitly *not* needed — say so.

### Deep dive 2: delivery guarantees
At-least-once end-to-end: server acks A after persist; B acks receipt back; unacked → redeliver on reconnect (client sends `last_seq`, server returns the gap). Duplicates handled by client-side dedup on message ID (**idempotency**). Receipts (delivered/read) are just tiny messages flowing the same path. Group chat: fan-out to N members' gateways — for 500 members it's a loop over registry lookups; per-member delivery cursors track who has what.

### Bottlenecks & fixes
- Gateway crash → 500k reconnects: jittered backoff, spare capacity; no data loss since messages persist before ack.
- Hot conversation (huge group blowing up) → its partition is hot: cap group size, or split delivery via a per-group queue with multiple consumers.
- Presence updates (online/offline) are chatty: batch + throttle; presence is allowed to be eventually consistent/approximate.
- History reads → recent messages cached (Redis, per conversation); old history from cold storage.

### Follow-up questions
- **How does typing indicator work?** Ephemeral event via pub/sub, never persisted, best-effort.
- **How do you sync multiple devices?** Per-device delivery cursors; each device fetches since its own last_seq; server keeps messages until all devices ack (or TTL).
- **Message ordering across A and B sending simultaneously?** Server assigns sequence at persist time — arrival order at the partition is the order. Client timestamps are hints only (clock skew).
- **E2E encryption impact?** Server stores ciphertext; can't do server-side search/moderation; key distribution (Signal protocol) — flag complexity, don't deep-dive unless asked.
- **Why not just poll?** 10M clients polling every 2s = 5M QPS of mostly-empty responses vs 10M idle sockets — sockets win clearly here.

### Tradeoffs to defend
WebSocket (latency, cost of statefulness) vs polling; Cassandra (write throughput, per-conversation model) vs SQL (simpler, harder to scale writes); registry routing (targeted) vs pub/sub broadcast (simpler fan-out, more waste); at-least-once + dedup (practical) vs exactly-once (impossible).

## 3. Advanced Version

- **Multi-region:** users connect to nearest region; conversation "homed" where most members are; cross-region delivery via inter-region message bus; per-conversation sequencing stays single-home to preserve ordering (or use per-region senders + causal merge — harder).
- **Failure:** region down → clients reconnect elsewhere; conversation homes fail over via leader election; accept brief delivery pause, never loss.
- **Hot problems:** celebrity broadcast channels → this becomes pub/sub fan-out like News Feed, not group chat — switch pattern (one write, millions of read subscriptions).
- **Media:** presigned upload to object storage + CDN; message carries the URL (see Pastebin pattern).
