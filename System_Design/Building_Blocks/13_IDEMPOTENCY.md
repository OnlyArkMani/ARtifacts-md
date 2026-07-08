# Idempotency

## 1. What Problem It Solves

Networks fail ambiguously: a client sends "charge $50", the response times out — did the charge happen? The only safe client behavior is to retry, but a naive retry might charge twice. **Idempotency** means doing an operation twice has the same effect as doing it once — making retries safe. It's the glue that makes at-least-once delivery (which is what every real system has) behave like exactly-once.

## 2. How It Works

### Naturally idempotent operations
- `SET status = 'paid'` (absolute assignment) — idempotent.
- `balance = balance - 50` (relative change) — **NOT** idempotent.
- HTTP: GET/PUT/DELETE are idempotent by contract; POST is not.
- Design trick: convert relative ops to absolute ones, or attach unique identity to each logical operation.

### Idempotency keys (the standard pattern, e.g. Stripe)

1. Client generates a unique key per *logical operation* (UUID) and sends it: `Idempotency-Key: abc-123`.
2. Server, atomically: check if key exists in the idempotency store.
   - New → record key (status: in-progress), execute, store the response with the key.
   - Seen + completed → **return the stored response** without re-executing.
   - Seen + in-progress → block/409 (concurrent duplicate).
3. Keys expire after a window (e.g., 24h) to bound storage.

**Critical detail:** the "check key + execute + record result" must be atomic (DB transaction or careful state machine), or a race between two identical retries defeats the whole scheme. Simplest robust version: unique constraint on the key column — the second insert fails, so the second request is safe.

### In message consumers
Consumer processes message → crashes before ack → message redelivered. Fix: dedup table keyed by message ID, checked inside the same transaction as the business write; or make the handler's effect an upsert.

## 3. Tradeoffs

**Pros:** safe retries everywhere (clients, queues, workflows); simpler failure handling than distributed transactions; cheap to implement compared to what it prevents.

**Cons:** storage for keys/responses; atomicity requirements add complexity; correct key *scope* is subtle (per logical action, not per HTTP attempt — same key must be reused on retry); doesn't help if two *different* keys represent the same human intent (user double-clicks "Buy" and client generates two keys — fix at client: disable button, generate key per form render).

**When NOT to bother:** truly read-only operations; cases where duplicates are harmless and cheap to ignore.

## 4. Diagram

```
Client                         Server            Idempotency store
  │ POST /charge key=K1 ──────▶ │ ── K1 new? ────▶ (insert K1) 
  │                             │  execute charge, store result
  │        ✖ response lost      │
  │ POST /charge key=K1 (retry)▶│ ── K1 exists ──▶ return SAVED result
  │ ◀──── "charged $50" ─────── │      (no second charge ✅)
```

## 5. Common Interview Follow-ups

**Q: How does idempotency give "exactly-once" processing?**
A: True exactly-once *delivery* is impossible over unreliable networks; what you achieve is at-least-once delivery + idempotent processing = **exactly-once effect**. That phrasing scores points.

**Q: Where should the idempotency key come from?**
A: The *originator* of the intent — the client generates it per logical action and reuses it on retries. Server-generated keys can't dedupe client retries of the request that creates them.

**Q: How do you make the key check race-proof?**
A: Unique constraint / atomic insert on the key. Two concurrent identical requests → one insert succeeds and executes; the other fails the insert and waits/returns the stored result.

**Q: Idempotency vs distributed lock — when each?**
A: Lock prevents concurrent execution; idempotency makes duplicate execution harmless. Idempotency is usually cheaper and more robust (no TTL/fencing headaches). Use locks when the operation has external side effects that can't be deduped (e.g., physically sending an email — though even there, dedup-before-send works).

**Q: What do you store with the key?**
A: Request hash (to reject same-key-different-body misuse), status (in-progress/done), and the full response to replay. TTL to bound growth.

---
**Used in case studies:** Payment System (core), Notification System (no duplicate pushes), Job Scheduler, anything consuming queues.
