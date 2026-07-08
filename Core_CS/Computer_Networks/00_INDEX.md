# Computer Networks — Index & Fast Revision

## Tiered Priority

**Tier 1 (asked constantly):**
- `02_TCP_VS_UDP.md` — #1 networking question + 3-way handshake
- `03_HTTP_HTTPS.md` — methods, status codes, TLS, versions
- `01_OSI_VS_TCPIP.md` — layers + "what happens when you type a URL"
- `04_DNS.md` — resolution walk, caching

**Tier 2 (commonly asked):**
- `05_IP_ADDRESSING_SUBNETTING.md` — the numerical topic; drill it
- `07_CONGESTION_FLOW_CONTROL.md` — flow vs congestion, slow start/AIMD
- `08_SOCKETS_CLIENT_SERVER.md` — API sequence, epoll/C10K

**Tier 3 (occasional):**
- `06_ROUTING_BASICS.md` — DV vs LS, OSPF vs BGP, traceroute
- `09_LOAD_BALANCING_CDN_NETWORK.md` — light touch, cross-refs System Design

## All Comparison Tables in One Scan

**OSI vs TCP/IP:** 7 prescriptive layers vs 4–5 real; session+presentation folded into application. Mnemonic bottom-up: Please Do Not Throw Sausage Pizza Away.

**Hub vs Switch vs Router:** L1 repeater / L2 by MAC within LAN / L3 by IP between networks. MAC changes per hop; IP constant end-to-end.

**TCP vs UDP:** connection+reliable+ordered+slower vs connectionless+best-effort+fast; 20B vs 8B header; web/email/DB vs DNS/video/gaming/QUIC.

**HTTP versions:** 1.0 conn-per-request → 1.1 keep-alive → 2 multiplexed streams over one TCP → 3 QUIC/UDP (kills TCP head-of-line). 

**GET vs POST:** fetch/idempotent/cacheable/params-in-URL vs create/non-idempotent/body. **PUT vs POST vs PATCH:** replace-idempotent / create / partial. **401 vs 403:** unauthenticated vs unauthorized. **502 vs 504:** upstream broken vs upstream slow.

**Recursive vs iterative DNS:** resolver does the walking vs each server points onward. **A vs CNAME:** IP vs alias.

**IPv4 vs IPv6:** 32-bit+NAT vs 128-bit end-to-end. Private ranges: 10/8, 172.16/12, 192.168/16.

**Flow vs congestion control:** receiver's buffer (rwnd, explicit) vs network capacity (cwnd, inferred); send min(rwnd, cwnd).

**Distance vector vs link state:** rumor+Bellman-Ford+RIP vs map+Dijkstra+OSPF. **OSPF vs BGP:** intra-AS shortest path vs inter-AS policy.

**select vs epoll:** O(n) scan, fd caps vs O(1) event-driven — the C10K answer.

**LB layers:** DNS (region, slow failover) → anycast (site, BGP) → L4 (connection, fast) → L7 (request, smart).

## Cheat Numbers & Formulas
Hosts = 2^(32−N) − 2 · block size = 256 − mask octet · BDP = bandwidth × RTT · throughput ≤ window/RTT · handshake = 1 RTT (TCP) + 1 RTT (TLS1.3) · cross-continent RTT ~100–150ms.

## Standard Traces to Drill
Subnet bracket (192.168.1.100/26 → .64 net, .65–.126, .127 bcast) · split /24 into 4× /26 · 64KB window over 80ms = 6.4 Mbps ceiling · URL-to-page walkthrough in 90s.
