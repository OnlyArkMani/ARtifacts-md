# Page Replacement Algorithms

> **Tag: Applied** — you WILL be asked to trace a reference string by hand. Practice the traces below until automatic.

## Concept

RAM is full and a page fault needs a frame → which resident page do we evict? Wrong choice = that page is needed again soon = another fault. Goal: minimize fault count.

## The Algorithms

- **FIFO:** evict the page that has been in memory longest. Simple; ignores usage. Suffers **Belady's anomaly** — more frames can *increase* faults (the classic gotcha).
- **Optimal (OPT/MIN):** evict the page not needed for the longest future time. Impossible in practice (needs the future) — used as the benchmark.
- **LRU:** evict least-recently-used. Approximates OPT via locality; no Belady's anomaly (it's a "stack algorithm"). True LRU is costly in hardware → real OSes approximate.
- **Approximations used in practice:** reference-bit **Clock (second chance)** — circular scan, skip-and-clear pages with ref bit set, evict first unset one. Cheap, near-LRU.

## Worked Trace (memorize the method)

Reference string: `1 2 3 4 1 2 5 1 2 3 4 5`, **3 frames**.

**FIFO:**
```
1→[1]F  2→[1,2]F  3→[1,2,3]F  4→[2,3,4]F  1→[3,4,1]F  2→[4,1,2]F
5→[1,2,5]F  1 hit  2 hit  3→[2,5,3]F  4→[5,3,4]F  5 hit
Faults = 9
```
Same string, **4 frames** → **10 faults** — Belady's anomaly demonstrated (know this exact example exists).

**LRU (3 frames):** evict by recency:
```
1F 2F 3F | 4 evicts 1 F | 1 evicts 2 F | 2 evicts 3 F | 5 evicts 4 F
1 hit | 2 hit | 3 evicts 5 F | 4 evicts 1 F | 5 evicts 2 F  → 10 faults
```
(On this adversarial string LRU ≥ FIFO — worth noting LRU isn't *always* better on paper, just usually in practice.)

**Optimal (3 frames):** evict the page used furthest in future → **7 faults** (the floor).

Method discipline: draw frame columns per step, mark F/H, count. State hit ratio = hits/refs at the end.

## Compare

| | FIFO | LRU | Optimal | Clock |
|---|---|---|---|---|
| Basis | Age in memory | Recency of use | Future knowledge | Ref-bit approximation of LRU |
| Overhead | Queue | Timestamp/stack per access (heavy) | — | 1 bit/page (cheap) |
| Belady's anomaly | Yes | No | No | Possible (it's FIFO-based) |
| Used where | Simple systems | Ideal target | Benchmark only | Real OSes |

## Most-Asked Interview Questions

1. **Trace this reference string with FIFO/LRU/Optimal.** → the drill above; be fast and neat.
2. **What is Belady's anomaly?** More frames → more faults, for FIFO on some strings (e.g., the one above). LRU/OPT are immune (stack property: pages in k frames ⊆ pages in k+1 frames).
3. **Why not implement true LRU?** Requires updating a timestamp/list on *every memory access* — hardware cost. Hence clock/second-chance approximations with reference bits.
4. **How does the clock algorithm work?** Circular list + ref bit: pointer scans, gives "second chance" (clear bit, move on) to referenced pages, evicts first page with bit=0.
5. **What is the dirty bit for?** Modified pages must be written back to disk before reclaim; clean pages can be dropped free. Prefer evicting clean pages (enhanced second chance uses (ref, dirty) pairs).
6. **Connection to caching?** Same problem as cache eviction (see System Design `02_CACHING.md`) — LRU appears in Redis, CPU caches, page replacement. One concept, three interview contexts.
