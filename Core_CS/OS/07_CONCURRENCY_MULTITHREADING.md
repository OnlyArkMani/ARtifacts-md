# Concurrency & Multithreading

> **Tag: Theory + code-reading** — expect conceptual "concurrency vs parallelism" plus practical thread-safety questions that bleed into coding rounds.

## Concept

**Concurrency:** structuring a program as independently progressing tasks (interleaved on one core is fine) — dealing with many things at once. **Parallelism:** literally executing simultaneously on multiple cores — doing many things at once. A single-core machine can be concurrent, never parallel. Interview line: *concurrency is about structure, parallelism is about execution.*

## Key Ideas

- **Amdahl's Law:** speedup ≤ 1 / (S + (1−S)/N) where S = serial fraction. If 10% of the work is serial, even ∞ cores cap at 10× — parallelism has diminishing returns; the serial part dominates. (Standard numerical: S=0.1, N=8 → 1/(0.1+0.9/8) ≈ 4.7×.)
- **I/O-bound vs CPU-bound:** I/O-bound benefits from concurrency (threads/async overlap waiting); CPU-bound needs true parallelism (more cores). Python note: the GIL blocks CPU-bound threading → multiprocessing; I/O-bound is fine with threads/asyncio.
- **Thread safety:** code behaves correctly under concurrent use. Achieved via: immutability (best), confinement (thread-local), synchronization (locks/atomics), or safe libraries (concurrent collections).
- **Thread pools:** don't create a thread per task — reuse a fixed pool + task queue (Java `ExecutorService`). Sizing intuition: CPU-bound → ~#cores; I/O-bound → ~#cores × (1 + wait/compute).
- **Async / event loop:** single thread + non-blocking I/O + callbacks/futures (Node, asyncio, Nginx) — massive I/O concurrency without thread-per-connection memory cost; but one blocking call stalls everything.
- **Common bugs:** race conditions, deadlock (see `05_DEADLOCK.md`), livelock, starvation, **visibility** problems (a write not seen by another thread due to caching/reordering — Java `volatile` guarantees visibility, NOT atomicity of compound ops like `count++`).

## Java snapshots (interview currency)

```java
// Thread creation: prefer Runnable/pool over extending Thread
ExecutorService pool = Executors.newFixedThreadPool(8);
pool.submit(() -> doWork());

// Atomicity without locks:
AtomicInteger counter = new AtomicInteger();
counter.incrementAndGet();            // CAS under the hood

// volatile = visibility only:
volatile boolean stop;                // flag OK
// count++ on volatile int — STILL a race (read-modify-write)
```

| | `synchronized` | `volatile` | `Atomic*` |
|---|---|---|---|
| Mutual exclusion | ✅ | ❌ | per-op (CAS) |
| Visibility | ✅ | ✅ | ✅ |
| Compound ops safe | ✅ | ❌ | ✅ (single var) |
| Blocking | Yes | No | No (lock-free) |

## Most-Asked Interview Questions

1. **Concurrency vs parallelism?** → structure vs simultaneous execution; single core can only interleave.
2. **How many threads should a pool have?** CPU-bound ≈ cores; I/O-bound more (waiting threads don't use CPU); cite the wait/compute ratio idea.
3. **Is this code thread-safe? / spot the race.** Look for: unsynchronized shared mutable state, check-then-act (`if(map.get(k)==null) map.put(...)`), compound ops on volatile.
4. **`volatile` vs `synchronized`?** → table above; volatile for flags/publication, synchronized/atomics for read-modify-write.
5. **What is a CAS / lock-free operation?** Atomic compare-and-swap loop: read, compute, swap-if-unchanged else retry — no blocking, but ABA problem and contention retries.
6. **Amdahl's Law — why can't 100 cores give 100×?** Serial fraction bounds speedup; also coordination overhead grows.
7. **Threads vs async — when each?** Thread-per-task: simpler mental model, fine to ~thousands. Event loop: 100k+ connections, I/O-heavy (see System Design WebSockets file — same tradeoff at system scale).
8. **What is a future/promise?** Handle to a result being computed; compose async work without callback nesting (`CompletableFuture`, `async/await`).
