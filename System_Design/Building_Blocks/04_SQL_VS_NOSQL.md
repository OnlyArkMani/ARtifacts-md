# SQL vs NoSQL Tradeoffs

## 1. What Problem It Solves

Choosing a datastore is the first big fork in any design. SQL (relational) gives you strong guarantees and flexible querying; NoSQL trades some of that away for horizontal scale, flexible schemas, or specialized access patterns. The interview skill is matching the store to the access pattern — not brand loyalty.

## 2. How It Works

### SQL (PostgreSQL, MySQL)
Tables with fixed schemas, joins, ACID transactions, secondary indexes, rich queries. Scales up well and scales reads with replicas; sharding is possible but manual and painful.

### NoSQL families

| Family | Model | Examples | Sweet spot |
|---|---|---|---|
| Key-value | key → blob | Redis, DynamoDB | Caching, sessions, simple lookups at huge scale |
| Document | key → JSON doc | MongoDB, DynamoDB | Flexible/nested data, per-entity access |
| Wide-column | row key → many columns, sorted | Cassandra, HBase | Massive write throughput, time-series, feeds |
| Graph | nodes + edges | Neo4j | Deep relationship traversal (fraud rings, social graphs) |

### The real tradeoff axes

| Axis | SQL | NoSQL (typical) |
|---|---|---|
| Schema | Rigid, enforced | Flexible, app-enforced |
| Transactions | Full ACID, multi-row | Usually single-item; limited multi-item |
| Joins | Native | Rare — denormalize instead |
| Scaling writes | Vertical / manual sharding | Horizontal by design |
| Consistency | Strong | Often tunable/eventual |
| Query flexibility | Ad-hoc queries anytime | Must model around known queries |

Key mental shift for NoSQL: **you design the data model around your queries** (denormalize, duplicate data), not around normalized entities.

## 3. Tradeoffs

**Pick SQL when:** data is relational, you need multi-row transactions (payments, inventory), query patterns are unknown/evolving, scale fits one beefy primary + replicas (this covers most startups — say this in interviews, it shows judgment).

**Pick NoSQL when:** write volume or dataset size exceeds single-node practicality, access is simple key-based lookups at scale, schema varies per record, you need multi-region active-active writes (Cassandra/Dynamo style).

**When NOT to use NoSQL:** because it's trendy; when you need ad-hoc analytics or joins; when strong consistency across entities is a core requirement.

**Middle ground worth naming:** NewSQL / distributed SQL (Spanner, CockroachDB, Aurora) — SQL semantics with horizontal scale, at cost of higher write latency (consensus) and complexity.

## 4. Diagram

```
Decision sketch:

  Need multi-row ACID txns? ──yes──▶ SQL (or Spanner-class if huge scale)
        │no
  Simple key lookups, huge scale? ──yes──▶ KV / Document (DynamoDB)
        │no
  Write-heavy time-series/feed? ──yes──▶ Wide-column (Cassandra)
        │no
  Deep relationship traversals? ──yes──▶ Graph
        │no
        ▶ Default to SQL + replicas + cache
```

## 5. Common Interview Follow-ups

**Q: Why is joining hard in distributed NoSQL?**
A: Joined rows live on different machines; a join becomes a cross-network scatter-gather. Instead you denormalize — store the data pre-joined in the shape you'll read it.

**Q: Isn't denormalization dangerous?**
A: You trade update complexity for read speed: one fact stored in N places must be updated in N places (often async via queues/streams). Acceptable when reads dominate writes.

**Q: Can SQL scale to web scale?**
A: Yes — with read replicas, caching, and sharding (Facebook/YouTube ran sharded MySQL). It scales; it's the *operational cost* of manual sharding that pushes people to NoSQL.

**Q: What consistency does DynamoDB/Cassandra give?**
A: Tunable. Cassandra: per-query consistency level (ONE, QUORUM, ALL). DynamoDB: eventually-consistent reads by default, strongly-consistent reads on demand (within a region). Quorum rule: R + W > N ⇒ strong reads.

**Q: How would you store both orders (transactions) and a product catalog (reads at scale)?**
A: Polyglot persistence — SQL for orders, document/KV + cache/search index for catalog. Using multiple stores, each for its strength, is a normal and expected answer.

---
**Used in case studies:** the database choice discussion appears in every case study; contrast is sharpest in Payment System (SQL) vs Key-Value Store / News Feed (NoSQL).
