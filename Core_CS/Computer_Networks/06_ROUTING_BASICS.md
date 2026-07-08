# Routing Basics

> **Tag: Theory/recall** — lighter topic; know forwarding vs routing, the two algorithm families, and OSPF vs BGP's division of labor.

## Concept

**Forwarding:** per-packet action — look up destination IP in the routing table, send out the matching interface. **Routing:** the background process of *building* that table. Routers use **longest-prefix match**: `10.1.2.0/24` beats `10.0.0.0/8` for `10.1.2.5` — most specific wins; `0.0.0.0/0` is the default route (least specific, matches everything).

## Two Algorithm Families

| | Distance Vector | Link State |
|---|---|---|
| Idea | Tell *neighbors* your distance to everyone ("routing by rumor") | Tell *everyone* your links; each router runs Dijkstra on the full map |
| Algorithm | Bellman-Ford | Dijkstra |
| Info shared | My distance table → neighbors | My link states → flooded to all |
| Convergence | Slow; **count-to-infinity** problem | Fast |
| Fixes | Split horizon, poison reverse, max hop 15 | — |
| Resources | Low CPU/memory | Higher (full topology) |
| Example | RIP (hops, max 15) | **OSPF** (cost/bandwidth) |

DSA link: Dijkstra and Bellman-Ford from your graphs file are running the internet — same algorithms, real deployment.

## Interior vs Exterior: OSPF vs BGP

- **IGP (within one organization/AS):** OSPF/IS-IS — optimize for *shortest path*, trust everyone, converge fast.
- **EGP (between organizations):** **BGP** — the internet's glue. Routes between **Autonomous Systems** (AS = one admin domain: an ISP, Google, your company). BGP is **policy-based, not shortest-path**: business relationships (customer/peer/provider) decide paths. Path-vector protocol (advertises full AS path → loop prevention).
- BGP trivia that impresses: convergence takes minutes; hijacks happen (announcing someone else's prefix — YouTube/Pakistan 2008); the internet's routing table is ~1M prefixes.

| | OSPF | BGP |
|---|---|---|
| Scope | Inside an AS | Between ASes |
| Metric | Link cost (bandwidth) | Policy + AS-path length |
| Goal | Fastest path | Business-correct path |
| Trust | Full | Adversarial |

## Supporting Protocols

- **ARP:** IP → MAC within a LAN (broadcast "who has 10.0.0.5?"). Each hop re-resolves the next hop's MAC. (L2/L3 boundary glue.)
- **ICMP:** control messages — `ping` (echo), TTL-exceeded (which powers `traceroute`: send packets with TTL=1,2,3…, each hop's expiry reveals it), destination-unreachable.
- **TTL:** decremented per hop; 0 → drop — prevents packets looping forever.

## Most-Asked Interview Questions

1. **How does a router decide where to send a packet?** Longest-prefix match in the forwarding table → next hop + interface; default route as fallback.
2. **Distance vector vs link state?** → table; "rumor vs map" framing; Bellman-Ford vs Dijkstra.
3. **What is BGP and why isn't it shortest-path?** Inter-AS glue; policies encode money (prefer customer > peer > provider routes); correctness = business intent.
4. **What is an Autonomous System?** One routing-policy domain with an ASN; internet = ~100k ASes wired by BGP.
5. **How does traceroute work?** Incrementing TTLs harvest ICMP TTL-exceeded replies hop by hop — elegant protocol abuse, interviewers love it.
6. **What is ARP? Where do MAC addresses change?** IP→MAC broadcast resolution; MACs swap every L2 hop, IPs constant end-to-end.
7. **Count-to-infinity problem?** DV routers slowly incrementing a dead route via each other's stale info; split horizon/poison reverse/hop limits mitigate.
