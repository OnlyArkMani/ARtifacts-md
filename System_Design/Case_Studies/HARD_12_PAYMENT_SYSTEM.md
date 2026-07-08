# Case Study: Multi-Region Payment System (Hard)

**Building blocks used:** Idempotency (core) · SQL/ACID · Message Queues · Distributed Locks/Sagas · Replication (sync) · CAP (choosing C!)

The one case study where you **choose consistency over availability** — and must defend it. Correctness > latency > availability. Money cannot be eventually consistent in ways that lose or duplicate it.

---

## 1. Basic Version (single server)

`POST /payments {amount, card, merchant}` → call bank/PSP (Stripe-style) → record result.

Immediate hard problem even at N=1: **the ambiguous failure** — you call the bank, timeout... did it charge? You cannot know. This is why payments is really a *reconciliation and idempotency* problem, not a CRUD problem. Everything in the scaled design exists because of this.

Core tables: `payments(id, idempotency_key UNIQUE, state, amount, …)` + append-only `ledger(txn_id, account, debit, credit)` — **double-entry: every movement writes matching debit+credit; sum of all entries = 0, always auditable.** Never store balance as a mutable cell; balance = SUM(ledger) (+ cached snapshots).

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Accept payment (card via external PSP); record in ledger; refunds; query status; payouts to merchants. (Cut: fraud scoring, FX — flag.)

### Non-functional
- 10k payments/s peak (big but not the challenge — correctness is)
- **Zero lost/duplicated money** — the prime directive
- Latency: 1–2s acceptable (users tolerate it for payments)
- Auditability: immutable trail, reconciliation with bank statements
- Availability: high, but **consistency wins conflicts** (CP)

### Capacity estimate
10k TPS × ~5 KB (payment + ledger + events) ≈ 50 MB/s writes — modest. The hard part is **the writes must be transactional**; shard SQL by merchant/account, each shard strongly consistent. Numbers show: this design is shaped by *correctness*, not throughput — good to say aloud.

### High-level design

```
Client ──▶ API GW ──▶ Payment service
                        │ 1. INSERT payment (state=INITIATED,
                        │    idempotency_key UNIQUE) ← dedup gate
                        │ 2. outbox event in SAME txn
                        ▼
                   SQL (sharded, sync-replicated)
                        │ outbox poller
                        ▼
                     Kafka ──▶ PSP-caller worker
                                │ call Stripe/bank with SAME
                                │ idempotency key, retry safely
                                ▼
                        update state: AUTHORIZED/FAILED
                        + ledger entries (double-entry)
                                ▼
                   webhook/async events ──▶ reconciliation svc
                                            (daily: our ledger vs
                                             PSP/bank statements)
```

### Deep dive 1: exactly-once *effect* via idempotency everywhere
- Client → us: client-generated idempotency key; UNIQUE constraint makes duplicate submits/retries return the original result (see `../Building_Blocks/13_IDEMPOTENCY.md`).
- Us → PSP: forward *our* key; PSPs support idempotency keys precisely for the timeout-ambiguity case — a retry can't double-charge.
- Timeout from PSP → state=PENDING_UNKNOWN → poll PSP status API / await webhook — **never blind-retry without the key, never assume failure.** This state machine handling of "unknown" is what interviewers probe.
- **State machine with legal transitions only** (INITIATED→AUTHORIZED→CAPTURED→SETTLED; +FAILED/REFUNDED); transitions are DB transactions; illegal transition = bug caught, not money lost.

### Deep dive 2: the outbox pattern (DB + queue atomicity)
Problem: "write DB AND publish event" isn't atomic across two systems — crash between them = inconsistency. Solution: write business row + event row in **one DB transaction** (outbox table); a poller/CDC publishes events from outbox to Kafka (at-least-once; consumers dedupe). Every reliable money-moving pipeline uses this — name it explicitly.

### Deep dive 3: wallet/ledger transfers (internal money movement)
A→B transfer touching two shards: no 2PC if avoidable — **saga**: debit A (local txn) → credit B (local txn) → on failure, compensate (reverse debit). Ledger entries make every step auditable/replayable. For same-shard accounts it's one ACID transaction — shard by account to make the common case local.

### Bottlenecks & fixes
- Hot merchant account (every sale credits one row) → don't serialize on one row: append ledger entries (no row contention by design), balance materialized async; or sub-accounts summed.
- DB write ceiling → shard by merchant_id/account_id; each payment mostly touches one shard.
- PSP latency (1–2s) → the async worker model already isolates it; API returns "processing" + webhook/poll for result.
- Reconciliation load → streaming compare of ledgers vs PSP reports; discrepancies → ops queue (they *will* happen; the process is the product).

### Follow-up questions
- **Why SQL not NoSQL?** Multi-row ACID (payment + ledger + outbox in one txn) is the entire correctness story; scale via sharding, not by dropping transactions.
- **What if the crash happens after PSP success but before you record it?** Recovery via idempotent re-query: pending-unknown reconciler polls PSP by key; webhooks replay. Money is never decided by our memory alone — the ledger + PSP + reconciliation triangle catches it.
- **Refunds?** New transaction referencing the original (never mutate history), same idempotency machinery, compensating ledger entries.
- **Why is the ledger append-only?** Auditability, replayability, no lost-update races, regulatory demands; mutable balances can't be audited.
- **Exactly-once — really?** At-least-once everything + idempotency keys + dedup = exactly-once *effect*. Precise phrasing scores.

### Tradeoffs to defend
CP over AP (declining payments during partition beats double-charging — defend with a concrete failure story); sync replication (durability of acked payments) vs latency; saga (available, complex compensations) vs 2PC (simple model, blocking/fragile); async PSP calls (isolation) vs sync UX (simplicity).

## 3. Advanced Version (multi-region)

- **Home-region per account/merchant:** all writes for an account route to its home region (strong consistency locally, no cross-region write conflicts *by construction*); async replicate for reads/DR. Cross-region transfer = saga between homes.
- **Region failure:** promote replicas in survivor region — but async replication may lose tail writes → reconciliation + PSP as external source of truth to replay the gap; accept minutes of degraded writes for affected accounts (state the RPO/RTO honestly — pretending zero-loss failover is a red flag).
- **True global strong consistency alternative:** Spanner-class DB (sync cross-region quorums) → pay ~50–100ms write latency, simpler app logic — a legitimate different point on the tradeoff curve; name both.
- **Compliance:** data residency (EU payments stay in EU → regional homing helps), PCI scope minimization (never touch raw PANs — tokenize at edge/PSP), immutable audit exports.
- **Fraud checks:** inline pre-auth scoring with a hard latency budget (~100ms) + async post-auth deep analysis → the two-tier pattern again.
