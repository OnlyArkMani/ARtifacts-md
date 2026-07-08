# Back-of-Envelope Estimation Cheat Sheet

Interviewers don't want precision — they want you to reason in powers of ten, sanity-check your own design, and know when a number forces an architecture change (e.g., "this doesn't fit on one machine").

## Latency Numbers to Memorize (rounded, order-of-magnitude)

| Operation | Time |
|---|---|
| L1 cache reference | 0.5 ns |
| Main memory reference | 100 ns |
| Read 1 MB sequentially from RAM | 0.25 ms |
| SSD random read | ~100 µs |
| Read 1 MB sequentially from SSD | ~1 ms |
| HDD seek | ~10 ms |
| Round trip within same datacenter | ~0.5 ms |
| Round trip cross-continent (US↔EU) | ~100–150 ms |
| Redis GET | ~0.2–1 ms (incl. network) |
| SQL query (indexed, warm) | ~1–10 ms |

**Takeaways:** RAM is ~1000× faster than disk; same-DC network is cheap, cross-region is not; anything cross-continent on a synchronous path costs you ~100ms+.

## Handy Constants

- 1 day ≈ **86,400 s** ≈ 10⁵ s (use 100k for mental math)
- 1 million requests/day ≈ **~12 QPS** average → ~2–5× for peak (use 10× if bursty)
- 2³⁰ ≈ 10⁹ (GB), 2⁴⁰ ≈ 10¹² (TB), 2⁵⁰ ≈ 10¹⁵ (PB)
- Char = 1 B, UUID = 16 B, typical DB row = 100 B–1 KB, image = 100 KB–1 MB, 1 min 1080p video ≈ 50–100 MB
- Single modern server ballparks: ~10k–100k simple QPS, 100s of GB RAM, TBs of SSD; MySQL: ~1–10k writes/s comfortably

## The Method (always the same 4 steps)

1. **Users → QPS:** DAU × actions/user/day ÷ 86,400 = avg QPS; ×~3–10 = peak QPS.
2. **Storage:** items/day × size/item × retention days (×replication factor ~3).
3. **Bandwidth:** QPS × payload size.
4. **Memory/cache:** apply 80/20 — cache ~20% of daily data for most of the reads.

Round aggressively. 86,400 → 100k. State assumptions out loud.

## Worked Example 1: Twitter-like Feed

- 200M DAU, each posts 2 tweets/day, reads feed 20×/day.
- **Write QPS:** 200M × 2 / 100k s ≈ **4k QPS** (peak ~12k). Trivial for a queue, fine for a sharded DB.
- **Read QPS:** 200M × 20 / 100k ≈ **40k QPS** (peak >100k) → read-heavy 10:1 → cache feeds aggressively, precompute (fan-out on write).
- **Storage:** 400M tweets/day × ~300 B ≈ 120 GB/day text → ~44 TB/yr raw, ~130 TB with 3× replication. Media dominates: if 10% carry a 200 KB image → 8 TB/day → object storage + CDN, not the DB.

## Worked Example 2: URL Shortener

- 100M new URLs/month → 100M / (30 × 100k) ≈ **40 writes/s**. Nothing.
- Reads 100:1 → **4k reads/s** (peak ~10k) → one Redis instance handles this; DB behind for misses.
- Storage: 100M/mo × 500 B ≈ 50 GB/mo → 3 TB in 5 years → fits one machine + replicas; shard only for availability/geo reasons.
- Key length: need ~6B IDs over lifetime? Base62⁶ ≈ 5.7×10¹⁰ ✅ (62⁷ ≈ 3.5×10¹² if paranoid).

## Worked Example 3: Video Platform Bandwidth

- 5M concurrent viewers × 3 Mbps average bitrate = **15 Tbps** egress → impossible from origin; this single number *forces* a CDN architecture (>90% offload).
- Upload: 500 hrs/min × 60 min × ~1.5 GB/hr-ish processed ≈ tens of PB/month → object storage, chunked upload, async transcoding pipeline.

## Red Flags Your Estimate Should Trigger

- QPS > ~50k on one path → shard/cache/queue needed.
- Dataset > a few TB hot → won't fit one node's RAM/SSD comfortably → partition.
- Fan-out × followers > millions of writes per event → hybrid push/pull (celebrity problem).
- Synchronous cross-region call on the request path → ~100ms+ → redesign (async replication, regional homing).

## Interview Delivery Tips

State assumptions ("I'll assume 100M DAU — sound OK?"), round hard, compute per-second first, sanity-check against single-machine limits, and *use* the number: every estimate should end with an architectural conclusion, not just a figure.
