# Congestion Control & Flow Control

> **Tag: Theory + one numerical (bandwidth-delay / throughput)** — "flow vs congestion" is the guaranteed comparison; slow start / AIMD the guaranteed follow-up.

## The Distinction (lead with this)

| | Flow control | Congestion control |
|---|---|---|
| Protects | The **receiver** (don't overflow its buffer) | The **network** (don't melt the routers) |
| Signal | Receiver's advertised window (rwnd) in every ACK | Inferred: packet loss, delay, ECN marks |
| Set by | Receiver explicitly | Sender's algorithm (cwnd) |

Sender may transmit min(rwnd, cwnd) unacknowledged bytes. Two different problems, one sending limit.

## TCP Congestion Control (the narrative)

1. **Slow start:** cwnd starts ~1–10 segments, **doubles every RTT** (exponential — the name is a lie) until loss or threshold (ssthresh).
2. **Congestion avoidance (AIMD):** past ssthresh, cwnd += 1 MSS per RTT (additive increase); on loss, cwnd halves (multiplicative decrease) → the sawtooth. AIMD provably converges to fair sharing between flows.
3. **Loss signals:** timeout (bad — back to slow start) vs 3 duplicate ACKs → **fast retransmit** (resend immediately) + **fast recovery** (halve, stay in avoidance — the missing packet was probably isolated).
4. Modern variants: **CUBIC** (Linux default — window growth by cubic function, better for fat pipes), **BBR** (Google — models bandwidth+RTT instead of reacting to loss; great on lossy links where loss ≠ congestion).

```
cwnd
 │      ╱╲    ╱╲    ╱╲
 │     ╱  ╲  ╱  ╲  ╱     ← AIMD sawtooth
 │    ╱    ╲╱    ╲╱
 │   ╱ ← slow start (exponential)
 └──────────────────────▶ time
```

## Worked Numericals

**Bandwidth-Delay Product (the pipe's capacity):**
100 Mbps link, RTT 80ms → BDP = 100×10⁶ × 0.08 = 8×10⁶ bits = **1 MB**. The window must be ≥1 MB to fill the pipe — smaller window ⇒ sender idles awaiting ACKs. (This is why long fat networks need window scaling.)

**Throughput ceiling from window:** throughput = window / RTT. 64 KB window, 80ms RTT → 64KB/0.08s = 800 KB/s ≈ **6.4 Mbps** — on a 100 Mbps link! Window, not bandwidth, is the bottleneck. This calculation impresses; know it.

## Flow Control Mechanics

Receiver advertises free buffer (rwnd) in each ACK; rwnd=0 → sender pauses (probes periodically — zero-window probe, avoids deadlock). Sliding window = the general mechanism: send up to W unacked segments, slide forward as ACKs arrive (pipelining vs stop-and-wait: stop-and-wait uses the pipe RTT-fraction only).

## Most-Asked Interview Questions

1. **Flow vs congestion control?** → the table; receiver's buffer vs network capacity; rwnd vs cwnd; min() rule.
2. **Explain slow start + AIMD.** → doubling to threshold, then +1/halve sawtooth; why: probe capacity fast, then be gentle and fair.
3. **How does TCP detect congestion?** Loss via timeout (severe) or 3 dup ACKs (mild) → different reactions; modern: delay (BBR) and ECN marks.
4. **Compute BDP / why is my transfer slow on high-latency link?** → window/RTT ceiling calculation above.
5. **Why does one lost packet halve my speed?** AIMD multiplicative decrease — designed so competing flows converge to fairness; the sawtooth is the price of decentralized fairness.
6. **What are fast retransmit/recovery?** 3 dup ACKs → immediate resend; skip slow start since data is still flowing.
7. **UDP has neither — what goes wrong?** Unfair to TCP flows, can congest networks; apps must self-limit (QUIC reimplements congestion control in userspace — see TCP vs UDP file).
