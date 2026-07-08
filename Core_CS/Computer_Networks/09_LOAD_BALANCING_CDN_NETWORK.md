# Load Balancing & CDN — Network-Level View

> **Tag: Theory/recall, light touch** — full treatment lives in the System Design artifact (`System_Design/Building_Blocks/01_LOAD_BALANCING.md` and `06_CDN.md`). This file covers only the *network-layer* angles a CN interviewer probes.

## Load Balancing Through the Layers

- **DNS level (global):** GeoDNS returns different IPs per client location; health-checked answers. Coarse (client caching), but it's how traffic picks a *region*.
- **Anycast (IP level):** same IP announced via BGP from many sites; internet routing delivers to the nearest — instant failover, used by CDNs/DNS roots (1.1.1.1).
- **L4 (transport):** balance on 5-tuple, NAT/DSR forwarding; blind to content, very fast; connection stickiness by hash.
- **L7 (application):** parse HTTP, route by path/host/cookie, terminate TLS. Smarter, costlier.

Full request path in a big service: **DNS/GeoDNS → anycast edge → L4 LB → L7 LB/gateway → service**. Being able to draw that chain is the differentiator.

| Layer | Granularity | Speed | Failover | Example |
|---|---|---|---|---|
| DNS | Region/site | Minutes (TTL) | Slow (caches) | Route 53 |
| Anycast | Site | BGP re-route (~secs) | Fast | Cloudflare |
| L4 | Connection | µs | Health checks | AWS NLB, IPVS |
| L7 | Request | ms | Health checks | Nginx, ALB |

## CDN — Network View

- Edge selection = GeoDNS or anycast (see above).
- Why edges help beyond distance: **TCP+TLS handshakes terminate near the user** (3 RTTs × 20ms instead of × 150ms), persistent warmed connections edge→origin, congestion-control performs better on short links.
- Cache behavior driven by HTTP headers (`Cache-Control`, ETag/304) — cross-ref HTTP file.

## Networking Facts That Come Up Here

- **Sticky sessions** via consistent hashing on client IP (breaks behind CGNAT — many users share an IP) or L7 cookies.
- **Direct Server Return (DSR):** LB forwards inbound only; servers reply straight to client — LB bandwidth halved (asymmetric routing trick).
- **Health checks:** L4 (TCP connect) vs L7 (GET /health — catches app-level death). 
- **Keep-alive pools:** LB↔backend connection reuse eliminates handshake cost per request.

## Most-Asked Interview Questions

1. **How does a load balancer work at L4 vs L7?** → table + what each can see (ports vs URLs/cookies).
2. **How does traffic reach the nearest datacenter?** GeoDNS and/or anycast — explain both, note DNS-cache coarseness vs BGP's route-level precision.
3. **What is anycast?** One IP, many locations, BGP picks nearest; stateless-friendly (DNS, CDN edges); long-lived TCP on anycast needs care (route flaps).
4. **Why do CDNs make even dynamic (uncacheable) content faster?** Handshake termination at edge + optimized/warm edge-to-origin backbone.
5. **How do health checks and failover interact with DNS TTLs?** In-LB failover is instant; DNS-level failover waits out client caches → layer your failover.

**Deep dives → System Design artifact:** algorithms (round-robin/least-connections), LB high availability, CDN push/pull, invalidation strategies.
