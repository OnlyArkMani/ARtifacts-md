# TCP vs UDP

> **Tag: Theory/recall + diagram** — the #1 networking question. Know the table cold, the 3-way handshake by heart, and 2–3 use cases each.

## Concept

Both are transport protocols multiplexing by **port numbers**. **TCP** builds a reliable, ordered, connection-oriented byte stream on top of unreliable IP — it retransmits losses, reorders, and controls flow/congestion. **UDP** is IP with ports: fire a datagram, no promises — and therefore no overhead, no delays, no connection state.

## The Table (know cold)

| | TCP | UDP |
|---|---|---|
| Connection | Handshake first (stateful) | None (stateless) |
| Reliability | ACKs + retransmission | None (app's job) |
| Ordering | Guaranteed byte-stream order | None; independent datagrams |
| Speed/latency | Slower (handshake, ACKs, congestion ctrl) | Minimal latency |
| Header | 20+ bytes | 8 bytes |
| Flow/congestion control | Yes / Yes | No / No |
| Message boundaries | No (stream) | Yes (datagram) |
| Use cases | Web (HTTP/1–2), email, file transfer, DBs | DNS, video/voice calls, gaming, streaming telemetry, **QUIC/HTTP-3** |

## TCP Mechanics (follow-up territory)

**3-way handshake:**
```
Client ── SYN (seq=x) ──────────▶ Server
Client ◀─ SYN-ACK (seq=y, ack=x+1) ─ Server
Client ── ACK (ack=y+1) ────────▶ Server   → connection established
```
Why 3 and not 2? Both sides must confirm *both* directions and exchange initial sequence numbers; 2-way can't confirm the server's sequence was received (and stale duplicate SYNs would create ghost connections).

**Reliability machinery:** sequence numbers per byte → receiver ACKs → sender retransmits on timeout (RTO) or **fast retransmit** (3 duplicate ACKs). Checksums detect corruption.

**Connection teardown:** 4-way (FIN, ACK, FIN, ACK) — each direction closes independently. **TIME_WAIT** (~2×MSL) on the closer's side: absorbs stray late packets so a new connection on the same ports isn't corrupted — classic "why is my server full of TIME_WAIT sockets" question.

**Flow vs congestion control (don't mix them up):** flow control protects the *receiver* (advertised window); congestion control protects the *network* (cwnd — see `07_CONGESTION_FLOW_CONTROL.md`).

## Why UDP for real-time & why QUIC exists

Video call: a retransmitted packet arrives too late to be useful — better to skip it. TCP's in-order guarantee causes **head-of-line blocking** (one lost packet stalls everything behind it). UDP lets the app choose what reliability to build. **QUIC (HTTP/3)** = reliability + TLS + multiplexed streams *built on UDP* — per-stream ordering kills head-of-line blocking, and 0/1-RTT setup beats TCP+TLS's 2–3 RTTs. Mentioning QUIC upgrades your answer from textbook to current.

## Most-Asked Interview Questions

1. **TCP vs UDP differences + when each?** → table + "reliability vs latency" framing.
2. **Explain the 3-way handshake. Why not 2-way?** → diagram + both-directions + sequence-number sync argument.
3. **How does TCP guarantee reliability?** Seq numbers, cumulative ACKs, retransmission (timeout + fast retransmit), checksums; ordering via sequence reassembly.
4. **What is a SYN flood?** Attacker sends SYNs, never ACKs → server's half-open connection table fills. Defense: SYN cookies (stateless — encode state in the sequence number).
5. **Is UDP ever reliable?** The app can add ACKs/retries on top (DNS retries; QUIC is full reliability over UDP). Reliability is a spectrum you can build à la carte.
6. **Why does DNS use UDP?** Single small request/response — a handshake would triple the cost. Falls back to TCP for large responses (>512B/EDNS) and zone transfers.
7. **What's head-of-line blocking?** One lost TCP segment blocks delivery of everything after it (stream order promise). HTTP/2 multiplexing suffers it at TCP level → HTTP/3/QUIC fixes with independent streams.
8. **TIME_WAIT — what and why?** Post-close quarantine for late packets; 2×MSL; can exhaust ports on busy clients (fix: connection pooling, SO_REUSEADDR nuance).
