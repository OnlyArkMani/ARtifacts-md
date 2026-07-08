# DNS (Domain Name System)

> **Tag: Theory/recall** — the resolution walk-through is the core; caching and record types are the follow-ups.

## Concept

DNS translates names (`www.example.com`) to IPs. It's a **globally distributed, hierarchical, heavily cached database** — arguably the world's largest. Nothing on the internet starts without it, which is also why it's a famous single point of failure when misconfigured.

## Resolution (the walk-through to memorize)

```
Browser cache → OS cache → hosts file
      │ miss
      ▼
Recursive resolver (ISP / 8.8.8.8 / 1.1.1.1)   ← does the legwork, caches hard
      │ (on cache miss, iterates:)
      ├─▶ Root server:        "ask .com" (13 logical, anycast to 100s)
      ├─▶ TLD server (.com):  "ask ns1.example.com"
      └─▶ Authoritative NS:   "www.example.com = 93.184.216.34" ✅
      ▼
Answer cached at every level per TTL → returned to client
```

**Recursive vs iterative:** your resolver performs *recursion* on your behalf; its queries to root/TLD/authoritative are *iterative* (each server points to the next, doesn't fetch for you).

## Record Types

| Record | Maps | Note |
|---|---|---|
| A / AAAA | name → IPv4 / IPv6 | the core |
| CNAME | name → another name | alias; can't coexist w/ other records at same name; not at zone apex |
| MX | domain → mail servers | with priorities |
| NS | zone → authoritative servers | delegation glue |
| TXT | name → text | SPF/DKIM, domain verification |
| SRV | service → host:port | service discovery |

## Caching & TTL

Every answer carries a TTL; resolvers/OS/browsers cache until expiry. Low TTL (60s) = fast failover/migration, more query load; high TTL (24h) = efficient, slow to change. **Changing DNS is never instant** — old TTLs must drain ("propagation").

## DNS in System Design (cross-ref)

- **DNS load balancing / GeoDNS:** return different IPs per query or per client location — the top layer of global load balancing (cross-ref `System_Design/Building_Blocks/01_LOAD_BALANCING.md`, CDN routing).
- Failover: health-checked DNS (Route 53) pulls dead IPs — but client caches delay it (why TTL matters).

## Transport (favorite trivia cluster)

UDP port 53 for queries (small, retry-friendly); TCP for zone transfers and responses >512B (EDNS extends UDP); modern: DoH/DoT (DNS over HTTPS/TLS — privacy).

## Most-Asked Interview Questions

1. **Walk through what happens on a DNS lookup.** → the diagram; mention every cache layer — caching is the point.
2. **Recursive vs iterative query?** → resolver recurses for you; upstream is iterative.
3. **Why UDP for DNS?** One tiny round trip; handshake would triple latency; app-level retry suffices; TCP fallback exists for big answers.
4. **A vs CNAME?** Direct IP vs alias-to-name (extra lookup); CNAME forbidden at apex → ALIAS/ANAME workarounds.
5. **You changed DNS but users still hit the old server — why?** Cached TTLs downstream; plan migrations by lowering TTL in advance.
6. **How does DNS support load balancing / geo-routing?** Multiple A records (round-robin), GeoDNS, health-checked answers — coarse but global; client caching blunts precision.
7. **What is DNS spoofing/poisoning?** Attacker injects fake cached answers → hijack traffic. Mitigations: randomized query IDs/ports, DNSSEC (signed records), DoH.
8. **What are root servers — really 13?** 13 logical addresses, hundreds of anycast instances worldwide.
