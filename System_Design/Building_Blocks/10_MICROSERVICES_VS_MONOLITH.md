# Microservices vs Monolith

## 1. What Problem It Solves

This is an *organizational and operational* decision disguised as a technical one. A monolith is one deployable unit; microservices split the system into independently deployable services, each owning its data. Microservices solve problems of **scale in teams and deployment** — letting many teams ship independently — not primarily problems of runtime performance.

## 2. How It Works

**Monolith:** one codebase, one process (replicated behind a LB), one database. Function calls between modules. One deploy pipeline.

**Microservices:** each service = own codebase, own deploy, own datastore (**database-per-service** — sharing a DB couples services at the schema and defeats the point). Services communicate via network: sync (REST/gRPC) or async (queues/events). Requires supporting machinery: service discovery, API gateway, centralized logging/tracing (a request spans many services), CI/CD per service, container orchestration.

**Service boundaries** should follow business domains (orders, payments, users — DDD "bounded contexts"), not technical layers. Wrong boundaries = distributed monolith: all the network pain, none of the independence.

**Data consistency across services:** no more cross-service ACID transactions. Use **sagas** — a sequence of local transactions with compensating actions on failure (e.g., reserve inventory → charge card fails → release inventory) — and event-driven eventual consistency.

**Migration path:** the **strangler fig pattern** — peel capabilities off the monolith one at a time behind a gateway, rather than a big-bang rewrite.

## 3. Tradeoffs

| | Monolith | Microservices |
|---|---|---|
| Dev simplicity | ✅ one repo, local debugging | ❌ distributed system from day 1 |
| Deploy | one unit (whole app redeploys) | ✅ independent, small blast radius |
| Scaling | whole app scales together | ✅ scale hot services only |
| Failure isolation | one bug can down everything | ✅ contained (with care) |
| Consistency | ✅ ACID transactions | ❌ sagas/eventual consistency |
| Latency | ✅ in-process calls | ❌ network hops, retries, partial failure |
| Team autonomy | merge conflicts, coupled releases | ✅ per-team ownership |
| Ops cost | low | high (observability, orchestration, on-call) |

**When NOT to microservice:** small teams (<~20 engineers), early-stage products with shifting domains (wrong boundaries are expensive), or when you can't invest in the operational tooling. The interview-winning stance: **start with a modular monolith, extract services when team scale or independent-scaling needs demand it.**

## 4. Diagram

```
Monolith:                       Microservices:
┌──────────────────┐            ┌────────┐ ┌────────┐ ┌────────┐
│  UI │ Orders     │            │ Orders │ │Payments│ │ Users  │
│─────┼────────────│            │  svc   │ │  svc   │ │  svc   │
│Users│ Payments   │            └───┬────┘ └───┬────┘ └───┬────┘
│     │            │                ▼          ▼          ▼
└────────┬─────────┘             [own DB]   [own DB]   [own DB]
         ▼                            ↕ REST/gRPC/events ↕
    [ one DB ]                   + gateway, discovery, tracing
```

## 5. Common Interview Follow-ups

**Q: How do two services share data without sharing a database?**
A: APIs (ask the owner), or events (owner publishes changes; consumers keep local read-optimized copies). Duplication with eventual consistency is the accepted price.

**Q: How do you handle a transaction spanning services?**
A: Saga pattern — choreography (services react to each other's events) or orchestration (central coordinator drives steps). Each step has a compensating action for rollback.

**Q: What is a distributed monolith?**
A: Services so coupled they must deploy together or call each other synchronously in long chains — the costs of microservices with none of the benefits. Cause: boundaries cut along technical, not business, lines.

**Q: How do you debug a request across 10 services?**
A: Distributed tracing — a correlation/trace ID injected at the gateway and propagated in headers; spans collected in a tracer (Jaeger/Zipkin). Plus centralized structured logging.

**Q: When would you actually split a monolith?**
A: Signals: deploys blocked on unrelated teams, one component needs 10× the scale of the rest, build/test times unbearable, or org has grown so teams need ownership. Split the most independent, highest-churn domain first (strangler fig).

---
**Used in case studies:** architecture framing for all Medium/Hard studies; sagas appear in Payment System and Ride-Sharing.
