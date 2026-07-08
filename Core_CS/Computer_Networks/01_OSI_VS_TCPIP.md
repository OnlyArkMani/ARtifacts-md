# OSI vs TCP/IP Model

> **Tag: Theory/recall** — layer names + one protocol/device per layer + "what happens when you type a URL" is the bread and butter of network rounds.

## Concept

Networking is organized in layers: each layer offers a service to the one above and uses the one below, hiding its details. OSI is the 7-layer teaching model; TCP/IP (4–5 layers) is what reality runs.

```
OSI                         TCP/IP          Protocols            Unit / device
7 Application ┐
6 Presentation│──────────▶  Application     HTTP, DNS, SMTP, TLS  data
5 Session     ┘
4 Transport   ──────────▶  Transport       TCP, UDP              segment
3 Network     ──────────▶  Internet        IP, ICMP              packet / router
2 Data Link   ┐──────────▶ Link/Network    Ethernet, WiFi, ARP   frame / switch
1 Physical    ┘             Access          cables, radio         bits / hub
```

Layer one-liners:
- **Physical:** raw bits on a medium.
- **Data link:** frames between *directly connected* nodes; MAC addresses; error detection (CRC); switches.
- **Network:** packets across networks end-to-end; IP addressing + routing; routers.
- **Transport:** process-to-process delivery (ports); reliability, ordering, flow/congestion control (TCP) or bare datagrams (UDP).
- **Session/Presentation:** connection dialogs; encryption (TLS), serialization — folded into Application in TCP/IP.
- **Application:** the protocols apps speak — HTTP, DNS, SMTP, SSH.

**Encapsulation:** each layer wraps the layer above's data with its header: `[Eth [IP [TCP [HTTP data]]]]`. Receiving side unwraps in reverse. **MAC addresses change hop-by-hop; IP addresses stay end-to-end** — the single most clarifying fact in this topic.

## Compare

| | OSI | TCP/IP |
|---|---|---|
| Layers | 7 (prescriptive theory) | 4–5 (descriptive of the Internet) |
| Origin | ISO standard, model-first | ARPANET, protocols-first |
| Session/Presentation | Separate layers | Merged into Application |
| Usage today | Vocabulary/teaching ("L4 LB", "L7 proxy") | The actual stack |

## "What happens when you type google.com?" (the mega-question — 60-second version)

1. Browser cache / OS cache check → **DNS** resolution (recursive resolver → root → TLD → authoritative) → IP.
2. **TCP handshake** (SYN, SYN-ACK, ACK) to IP:443.
3. **TLS handshake** — certificate verification, key agreement.
4. **HTTP** request → server (through LB) → response.
5. Under the hood every packet: ARP for next-hop MAC, IP routing hop-by-hop, NAT at your router.
6. Browser parses HTML → fetches assets (many via CDN) → renders.

Each step name-drops a layer — interviewers steer into whichever they care about.

## Most-Asked Interview Questions

1. **Name the OSI layers + a protocol per layer.** (Mnemonic: *Please Do Not Throw Sausage Pizza Away*, bottom-up.)
2. **Why layering at all?** Separation of concerns, independent evolution (WiFi replaced Ethernet without touching TCP), interoperability, easier debugging ("which layer is broken?").
3. **Switch vs router vs hub?** Hub: L1 repeater (dumb broadcast). Switch: L2, forwards by MAC within a LAN. Router: L3, forwards by IP between networks.
4. **MAC vs IP address?** MAC: link-local hardware identity, per-hop scope. IP: global logical address, end-to-end. ARP translates IP→MAC on the local segment.
5. **Where do TLS, DNS, and load balancers sit?** TLS: between transport and application (5/6-ish; "L7" colloquially). DNS: application protocol. LBs: L4 (TCP) or L7 (HTTP) — cross-ref System Design load balancing.
6. **What happens when you type a URL?** → the 6 steps; practice a tight 60–90s delivery.
