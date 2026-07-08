# CAP Theorem — DBMS Angle

> **Tag: Theory/recall, light touch** — full treatment in `System_Design/Concepts/CAP_theorem_and_consistency_models.md`. This is the DBMS-round quick version: definition + classifying databases.

## 60-Second Statement

Distributed database + network partition ⇒ choose: **Consistency** (refuse possibly-stale answers) or **Availability** (answer with what you have). Partitions are inevitable, so every distributed DB has made this choice. Single-node databases are CA trivially — CAP only bites when data is replicated across nodes.

## Classifying the Databases You Know

| Database | Lean | Under partition |
|---|---|---|
| Single-node Postgres/MySQL | (CAP n/a) | — it's one node |
| MySQL w/ async replicas | AP-ish reads (stale possible), CP writes | replica lag = eventual reads |
| MongoDB (default) | CP | primary steps down without majority; reads/writes pause |
| Cassandra / DynamoDB | AP (tunable) | all reachable replicas serve; reconcile later |
| Spanner / CockroachDB | CP | minority side stalls; consensus per write |
| Redis Sentinel/Cluster | AP-leaning cache semantics | possible lost writes on failover |

Tunable consistency (Cassandra `QUORUM`, Dynamo R/W): pick per query — **R + W > N ⇒ strongly consistent read**; drop to R=1/W=1 for speed. This single formula is the most quotable fact in the topic.

## The DBMS-flavored Follow-ups

1. **How does replication lag relate to CAP?** Async replica reads = choosing availability/latency over consistency in normal operation (that's PACELC's "else" branch: Latency vs Consistency even without partitions).
2. **Read-your-own-writes problem?** User writes to primary, reads stale replica → route own reads to primary briefly or use session tokens (cross-ref System Design replication file).
3. **Is a bank on Cassandra a good idea?** Ledger needs multi-row ACID + strong consistency → no; use CP/SQL for money, AP for the activity feed next to it — per-data-classification answers score.
4. **ACID's C vs CAP's C?** Different things! ACID C = integrity constraints hold; CAP C = all nodes agree on latest value (linearizability). Distinguishing them is a classic gotcha.
5. **What is BASE?** The AP-side philosophy: Basically Available, Soft state, Eventually consistent (see `08_SQL_VS_NOSQL.md`).
