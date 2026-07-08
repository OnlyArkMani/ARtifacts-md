# IP Addressing & Subnetting

> **Tag: Applied** — subnetting calculations are THE numerical of networking rounds. Drill the /26 example until it's 30 seconds.

## Concept

IPv4 address = 32 bits, dotted decimal (`192.168.1.10`), split into **network part | host part** by the subnet mask. Routers route on the network part; the host part identifies machines within the network. **Subnetting** = borrowing host bits to create more, smaller networks — for organization, security zones, and broadcast-domain control.

## CIDR Notation

`/N` = first N bits are network. Mask `/24` = `255.255.255.0`.

**The formulas:**
- Hosts per subnet = 2^(32−N) − 2 (minus network address + broadcast address)
- Subnets from borrowing b bits: 2^b
- Block size in the "interesting octet" = 256 − mask-octet-value

| CIDR | Mask | Hosts |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /30 | 255.255.255.252 | 2 (point-to-point links) |

## Worked Numericals (the standard two)

**1) What subnet is 192.168.1.100/26 in?**
/26 → block size 256−192 = 64 → subnets: .0, .64, .128, .192.
100 falls in **.64 subnet**: network `192.168.1.64`, first host `.65`, last host `.126`, broadcast `.127`. That's the entire method: block size → count up → bracket the address.

**2) Split 10.0.0.0/24 into 4 equal subnets.**
Need 4 = 2² → borrow 2 bits → /26 each:
`10.0.0.0/26` (.1–.62), `10.0.0.64/26`, `10.0.0.128/26`, `10.0.0.192/26`. Each has 62 usable hosts.

**Sanity anchors:** can two IPs talk directly? AND both with the mask — same result = same subnet (direct), different = via gateway.

## Special Addresses (recognize instantly)

- **Private (RFC 1918):** `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16` — not routable on the internet → NAT needed
- **Loopback:** `127.0.0.1` · **Link-local/APIPA:** `169.254.x.x` (DHCP failed — classic troubleshooting clue)
- Network address (host bits all 0) and broadcast (all 1) — unusable for hosts

## NAT (always follows this topic)

Router rewrites private source IP:port → its public IP:unique-port, keeps a translation table, reverses on replies. Why: IPv4 exhaustion (+ a side effect of hiding topology). Cost: breaks inbound connections (needs port forwarding / hole punching — why P2P and game hosting is painful), breaks end-to-end principle.

## IPv4 vs IPv6

| | IPv4 | IPv6 |
|---|---|---|
| Size | 32-bit, ~4.3B | 128-bit, effectively unlimited |
| Notation | dotted decimal | hex groups `2001:db8::1` |
| NAT | Essential | Unnecessary (end-to-end restored) |
| Config | DHCP | SLAAC auto-config |
| Header | Variable, checksum | Simpler fixed, no checksum |

## Most-Asked Interview Questions

1. **Given IP/mask, find network, broadcast, host range.** → block-size method above.
2. **Split this network into N subnets.** → borrow ⌈log₂N⌉ bits.
3. **Why subtract 2 from host counts?** Network + broadcast addresses reserved. (/31 exception for point-to-point exists — nice extra.)
4. **What is NAT and why do we need it?** → exhaustion + private ranges; table-based rewriting; breaks inbound.
5. **Public vs private IP?** Routable vs RFC-1918 internal + NAT at the edge.
6. **How does a host know to use the gateway?** Mask-AND comparison: same subnet → ARP direct; different → send to default gateway's MAC.
7. **What does DHCP do?** Leases IP + mask + gateway + DNS to joining hosts (DORA: Discover, Offer, Request, Ack).
8. **169.254.x.x on a machine — what happened?** DHCP unreachable → self-assigned link-local → check the DHCP server/cable/VLAN.
