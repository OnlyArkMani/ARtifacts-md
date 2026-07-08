# Case Study: Distributed Job Scheduler (Hard)

**Building blocks used:** Message Queues · Distributed Locks / Leader Election · Idempotency · Data Partitioning · Indexing (due-time) · Replication

"Design distributed cron": run jobs at scheduled times (once or recurring), at scale, with failures — **exactly-once-ish execution when machines die mid-run** is the crux.

---

## 1. Basic Version (single server)

- Table: `jobs(id, cron_expr/run_at, payload, state)`; index on `next_run_at`.
- Loop: every second, `SELECT * FROM jobs WHERE next_run_at <= now() AND state='scheduled'` → execute → compute next occurrence (for cron) → update.
- In-process execution, timer drift handled by polling the DB clock. This IS cron with a DB. Fails on: box dies (nothing runs), long jobs block the loop, no retries.

First improvement even at small scale: **separate scheduling from execution** — the loop only *enqueues* due jobs; a worker pool executes. That split is the skeleton of the scaled design.

## 2. Scaled Version — full interview walkthrough

### Functional requirements
Create/cancel jobs: one-time (`run_at`) and recurring (cron); retries with backoff on failure; job status/history; per-job timeout. (Nice-to-have: DAG dependencies, priorities.)

### Non-functional
- Scale: 100M scheduled jobs, 10k executions/s peak
- **A due job must run** (no silent skips) — at-least-once; duplicates minimized and made harmless (idempotency)
- Trigger precision: within a few seconds is fine (say it — sub-second precision changes the design)
- Scheduler itself must survive node failures (HA)

### Capacity estimate
10k exec/s × ~1 KB job row ≈ trivial I/O; the challenge is **coordination** — who fires which job, exactly one dispatch per due time. 100M jobs / shard by time+hash → each scheduler partition scans its own slice: 10k/s ÷ 16 partitions ≈ 600 dispatch/s each — comfortable.

### High-level design

```
 API ──▶ Job store (sharded DB: job_id → def, state,
  │       next_run_at; index on (partition, next_run_at))
  ▼
┌────────────────────────────────────────┐
│ Scheduler fleet — each owns partitions │  partition ownership via
│ loop: poll due jobs in my partitions   │  leader election / lease
│  → atomically claim (state: due→queued)│  (ZooKeeper/etcd or DB
│  → push to execution queue             │   lease rows)
└──────────────────┬─────────────────────┘
                   ▼
              Kafka/SQS (per priority)
                   ▼
┌──────────────────────────────────────┐
│ Worker fleet: pull → execute →       │  heartbeat during run;
│ report status; retries w/ backoff;   │  invisibility timeout →
│ DLQ after N failures                 │  re-delivery if worker dies
└──────────────────────────────────────┘
                   ▼
        Status store + history (append-only) 
```

### Deep dive 1: no duplicate dispatch, no missed jobs
- **Partition ownership:** jobs partitioned (hash(job_id) or time-bucket); each partition has exactly one scheduler owner holding a **lease** (etcd/ZK ephemeral or DB row with expiry — see `../Building_Blocks/12_DISTRIBUTED_LOCKS.md`). Owner dies → lease expires → another node takes over and rescans → **missed-job window bounded by lease TTL**.
- **Atomic claim:** dispatch = single conditional update `UPDATE jobs SET state='queued' WHERE id=? AND state='due'` — even if two schedulers race (lease overlap), one wins. Defense in depth: lease prevents *most* races; CAS makes the survivor safe.
- Enqueue after claim; crash between claim and enqueue → recovery scan for `queued but not enqueued older than T` re-enqueues (idempotent since consumers dedupe by job_id+scheduled_time).

### Deep dive 2: execution reliability
- At-least-once via queue redelivery: worker takes job, heartbeats; dies → visibility timeout expires → redelivered to another worker.
- **Idempotent executions:** execution key = `(job_id, scheduled_time)`; side-effectful jobs check/record this key (or the job's own logic is idempotent — push responsibility to job authors with a documented contract).
- Long jobs: heartbeat extends lease; timeout kills + retries with backoff; poison jobs → DLQ + alert.
- Recurring jobs: on successful dispatch, compute and write `next_run_at` (in the claim transaction) — schedule from *scheduled* time, not completion time, to avoid drift (or completion-based if job must not overlap itself — ask which semantics! `concurrencyPolicy` in k8s CronJob terms).

### Bottlenecks & fixes
- **Thundering herd at midnight** (everyone schedules :00) → jitter windows (run within [00:00, 00:05]), rate-limit dispatch, spread by hash into the minute.
- Hot partition (time-ordered scan) → partition by hash, index by time *within* partition — the compound-key trick again.
- Status-write volume → append-only events, aggregate async.
- Queue backlog (workers slow) → autoscale on queue depth; priorities so cron-critical beats batch backfill.

### Follow-up questions
- **Exactly-once execution?** Impossible in general (worker can die after side effect, before ack). At-least-once + idempotent jobs = exactly-once effect. Same phrasing as payments/queues — the pattern transfers.
- **How precise are triggers?** Poll interval + queue latency ⇒ seconds. For sub-second: in-memory timing wheels on partition owners — more complexity; ask if it's required first.
- **DAG dependencies (job B after A)?** Completion events trigger dependents (event-driven) rather than time — this becomes Airflow; scheduling primitive stays the same.
- **Misfires (system down when job was due)?** Policy per job: run-immediately-once, skip, or catch-up-all — expose as config (name it: misfire policy).
- **Why not just use cron on many boxes?** No HA/failover, duplicate runs, no retries, no visibility — enumerate what the design adds.

### Tradeoffs to defend
Poll-based scheduling (simple, seconds precision) vs timing wheels (precise, complex); lease + CAS (two layers — justify: cheap insurance for correctness); at-least-once + idempotency (practical) vs attempted exactly-once (impossible); schedule-time vs completion-time recurrence (overlap semantics).

## 3. Advanced Version

- **Multi-region:** jobs homed to a region (owner partition lives there); region failure → leases expire, survivor region's schedulers adopt partitions from replicated job store (async replication ⇒ small window of possibly re-run jobs → idempotency absorbs). Truly global jobs: run in one elected region, failover with the same lease machinery.
- **Scale-out coordination:** avoid one global coordinator; partition ownership scales linearly; rebalance via consistent hashing of partitions across scheduler nodes.
- **Multi-tenancy:** per-tenant quotas + fair scheduling (weighted queues) so one tenant's 1M-job backfill can't starve others; per-tenant DLQs.
- **Observability as a feature:** execution history, SLA-miss alerts ("job late by >X"), dependency visualizations — for schedulers, operator trust is the product.
