# API Gateway

## 1. What Problem It Solves

With many microservices, clients would otherwise need to know every service's address, each service would reimplement auth/rate limiting/logging, and changing internal structure would break clients. An API gateway is the single front door: one endpoint that handles cross-cutting concerns and routes requests to the right internal service.

## 2. How It Works

Client sends all requests to the gateway, which performs (in rough order):

1. **TLS termination** — decrypt once at the edge.
2. **Authentication/authorization** — validate JWT/API key once; forward identity downstream via headers. Services trust the gateway.
3. **Rate limiting & quotas** — per user/key (see `08_RATE_LIMITING.md`).
4. **Routing** — `/users/*` → user service, `/orders/*` → order service (path/host/header based).
5. **Extras** — request/response transformation, response caching, retries/timeouts, circuit breaking, logging/metrics/tracing headers, CORS, payload validation.

**Related patterns:**

- **BFF (Backend-for-Frontend):** separate gateway per client type (mobile vs web) so each gets tailored aggregation instead of one bloated gateway.
- **Gateway aggregation:** one client call fans out to several services and merges results — saves mobile round trips.
- **Gateway vs Load Balancer:** LB distributes identical traffic across copies of one service; gateway routes *different* requests to *different* services and applies policy. Gateways usually sit in front of per-service LBs.
- **Gateway vs Service Mesh:** gateway = north-south traffic (external→internal); mesh (sidecars, e.g., Istio) = east-west (service↔service). Complementary.

Examples: Nginx/Kong, AWS API Gateway, Envoy, Zuul.

## 3. Tradeoffs

**Pros:** centralizes cross-cutting logic (huge duplication savings), hides internal topology, single choke point for security/observability, enables API versioning and canary routing.

**Cons:** potential single point of failure (must be replicated), added latency hop, risk of becoming a **bloated monolith-at-the-edge** if business logic creeps in, one team's bad config can affect everyone.

**When NOT to use:** a monolith with one client (needless indirection); ultra-low-latency internal paths; tiny systems where an LB + middleware library covers it.

## 4. Diagram

```
 Mobile/Web/3rd-party
        │
        ▼
 ┌─────────────────┐   auth ✓  rate-limit ✓  log ✓
 │   API Gateway   │
 └──┬────┬─────┬───┘
    │    │     │        (each service behind its own LB)
    ▼    ▼     ▼
 ┌─────┐┌─────┐┌───────┐
 │User ││Order││Payment│ ...
 │ svc ││ svc ││  svc  │
 └─────┘└─────┘└───────┘
```

## 5. Common Interview Follow-ups

**Q: Isn't the gateway a single point of failure?**
A: Run multiple stateless gateway instances behind a load balancer / anycast; externalize state (rate-limit counters in Redis). Managed gateways handle this for you.

**Q: Where should authentication live — gateway or services?**
A: Authenticate (verify identity) at the gateway once; pass verified claims downstream. Fine-grained *authorization* (can user X edit doc Y?) stays in services since it needs domain data. Zero-trust setups also verify service-to-service identity (mTLS via mesh).

**Q: What belongs in the gateway vs a service?**
A: Cross-cutting, business-agnostic concerns → gateway. Anything with domain logic → service. If gateway config starts encoding business rules, that's the smell.

**Q: How does the gateway know where services are?**
A: Service discovery — a registry (Consul, etcd, Kubernetes DNS) that services register with; gateway resolves logical name → healthy instances.

**Q: API Gateway vs reverse proxy?**
A: A gateway *is* a reverse proxy plus API-management features (auth, quotas, versioning, developer keys). Nginx is a reverse proxy; Kong = Nginx + those features.

---
**Used in case studies:** Rate Limiter, Notification System, News Feed, Ride-Sharing — any microservices-based design's entry point.
