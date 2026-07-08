# Load Balancing

## 1. What Problem It Solves

One server can only handle so much traffic. When you have multiple servers, something has to decide which server gets each incoming request — fairly, quickly, and without sending traffic to dead servers. That "something" is a load balancer. It also hides your fleet behind one address, so you can add/remove servers without clients noticing.

## 2. How It Works

A load balancer sits between clients and servers, receiving requests and forwarding them by an algorithm:

- **Round Robin** — rotate through servers in order. Simple, works when servers and requests are uniform.
- **Weighted Round Robin** — stronger servers get proportionally more requests.
- **Least Connections** — send to the server with fewest active connections. Better when request durations vary a lot.
- **Least Response Time** — send to the fastest-responding server.
- **IP Hash / Consistent Hash** — hash client IP (or a key) to pick a server. Gives *sticky* routing — same client hits same server (useful for sessions, caches).

**Layer 4 vs Layer 7:**

| | L4 (Transport) | L7 (Application) |
|---|---|---|
| Routes on | IP + port | URL, headers, cookies, method |
| Speed | Faster (no payload parsing) | Slower, but smarter |
| Can do | TCP passthrough | Path routing (`/api` → service A), TLS termination, sticky sessions via cookie |
| Examples | AWS NLB, HAProxy (TCP mode) | Nginx, AWS ALB, Envoy |

**Health checks:** the LB pings servers (e.g., `GET /health` every 5s); a server failing N consecutive checks is pulled from rotation, and re-added when healthy.

**Avoiding the LB as a single point of failure:** run LBs in pairs (active-passive with a floating IP/VRRP), or use DNS round-robin across multiple LBs. At global scale: DNS-based/anycast routing (e.g., Route 53, Cloudflare) balances across regions, then regional LBs balance across servers.

## 3. Tradeoffs

**Pros:** horizontal scaling, high availability (dead servers bypassed), zero-downtime deploys (drain one server at a time), single public entry point.

**Cons:** extra network hop (small latency), another component to operate and to fail, sticky sessions create uneven load and complicate autoscaling, L7 termination means the LB must handle TLS (CPU cost).

**When NOT to use:** single-server apps (needless complexity); stateful protocols where connection pinning defeats balancing anyway; extremely latency-sensitive paths where you'd rather do client-side load balancing (client picks a server directly from a service registry — common in internal microservices, e.g., gRPC + service discovery).

## 4. Diagram

```
                 ┌──────────────┐
   Clients ────▶ │ Load Balancer │  (health checks ↓)
                 └──────┬───────┘
              ┌─────────┼─────────┐
              ▼         ▼         ▼
         ┌───────┐ ┌───────┐ ┌───────┐
         │ Web 1 │ │ Web 2 │ │ Web 3 │  ← add/remove freely
         └───────┘ └───────┘ └───────┘
         (Web 2 fails health check → traffic goes to 1 & 3 only)
```

## 5. Common Interview Follow-ups

**Q: How do you make the load balancer itself not a single point of failure?**
A: Redundant LB pair with a virtual IP and failover (VRRP/keepalived), DNS pointing at multiple LBs, or anycast. Managed cloud LBs are already replicated internally.

**Q: Round robin vs least connections — when does it matter?**
A: When request costs are uneven. Round robin assumes uniform work; if some requests take 10× longer, one server accumulates slow requests. Least connections adapts to actual load.

**Q: How do you handle sessions with a load balancer?**
A: Best: make servers stateless — store session state in Redis or a signed cookie (JWT), so any server can serve any request. Fallback: sticky sessions (cookie/IP hash), but this hurts even distribution and breaks when a server dies.

**Q: L4 or L7 — which would you pick?**
A: L7 when you need path-based routing, TLS termination, or per-service routing (typical web/API). L4 when you need raw throughput, non-HTTP protocols, or end-to-end encryption without termination.

**Q: How does the LB know a server is down?**
A: Active health checks (periodic probes) + passive checks (mark down on connection errors/timeouts). Tune thresholds to avoid flapping.

---
**Used in case studies:** virtually all of them — see any Scaled version in `../Case_Studies/`.
