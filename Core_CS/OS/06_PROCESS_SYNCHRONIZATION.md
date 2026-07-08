# Process Synchronization

> **Tag: Theory + code** — race conditions, semaphores, and producer-consumer are near-guaranteed; be ready to write the semaphore solution from memory.

## Concept

Threads sharing memory can interleave badly. `counter++` is three instructions (load, add, store); two threads interleaving them lose updates — a **race condition**. The code region touching shared data is the **critical section**; we need **mutual exclusion**: at most one thread inside at a time.

A correct critical-section solution needs: **mutual exclusion**, **progress** (no one waiting forever if section is free), **bounded waiting** (no starvation).

## The Tools

| Tool | What it is | Use |
|---|---|---|
| **Mutex/lock** | Binary; only the locker may unlock | Protect a critical section |
| **Semaphore** | Counter + wait queue; `wait()/P` decrements (block if <0), `signal()/V` increments | Mutual exclusion (init 1) or resource counting / ordering (init N or 0) |
| **Monitor** | Language construct: object whose methods are implicitly mutually exclusive + **condition variables** (`wait/signal`) | Java `synchronized`, higher-level safety |
| **Spinlock** | Busy-wait loop on atomic flag | Very short waits on multicore (kernel); wastes CPU otherwise |
| **Atomic instructions** | Hardware `test-and-set`, `compare-and-swap` | The primitive everything above is built on; lock-free structures |

Mutex vs binary semaphore (favorite question): mutex has *ownership* (locker must unlock; enables priority-inheritance); semaphore can be signaled by anyone (usable for ordering/signaling between threads).

## Producer–Consumer (bounded buffer) — memorize this

```c
semaphore mutex = 1;      // protects buffer
semaphore empty = N;      // empty slots
semaphore full  = 0;      // filled slots

Producer:                    Consumer:
while(true) {                while(true) {
  item = produce();            wait(full);      // wait for an item
  wait(empty);   // slot?      wait(mutex);
  wait(mutex);                 item = remove();
  insert(item);                signal(mutex);
  signal(mutex);               signal(empty);   // freed a slot
  signal(full);  // +1 item    consume(item);
}                            }
```

**The trap they test:** swap `wait(empty)` and `wait(mutex)` in the producer → producer holds mutex while blocking on a full buffer → consumer can't acquire mutex to consume → **deadlock**. Always take the counting semaphore *before* the mutex.

## Dining Philosophers

5 philosophers, 5 forks, each needs both neighbors' forks. All grab left simultaneously → circular wait → deadlock. Fixes: (1) **asymmetry** — odd philosophers pick left-first, even pick right-first (breaks the cycle); (2) allow only 4 at the table (semaphore init 4); (3) pick up both forks atomically (monitor). Readers–Writers: many concurrent readers OR one writer; know that reader-priority starves writers and vice versa.

## Java flavor (since OOP rounds overlap)

```java
synchronized(obj) { ... }            // monitor entry
obj.wait();                          // release lock + sleep until…
obj.notify(); / notifyAll();         // …signaled (always in while-loop check!)
// java.util.concurrent: ReentrantLock, Semaphore, BlockingQueue (producer-consumer solved)
```

`wait()` must be called in a `while(condition)` loop, not `if` — spurious wakeups and re-check semantics (Mesa monitors).

## Most-Asked Interview Questions

1. **What's a race condition? Give an example.** Two threads doing `counter++` → lost update; show the 3-instruction interleaving.
2. **Mutex vs semaphore?** Ownership + binary vs counter usable for signaling/resources. Binary semaphore ≈ mutex without ownership.
3. **Write producer-consumer with semaphores.** → above, including the ordering trap.
4. **What are condition variables / why `while` around `wait()`?** Waiting for a predicate inside a monitor; recheck after wakeup (spurious wakeups, competing threads).
5. **Dining philosophers — deadlock and a fix?** → asymmetric acquisition or limiting to N−1.
6. **Spinlock vs mutex — when spin?** Expected wait shorter than two context switches, and on multicore. Kernels spin; apps usually shouldn't.
7. **What is priority inversion?** Low-priority thread holds a lock a high-priority thread needs, while medium-priority threads run — high waits on medium (Mars Pathfinder story). Fix: priority inheritance.
8. **Peterson's solution?** Two-process software mutex via `flag[]` + `turn` — know it exists and satisfies all three properties on old memory models (modern CPUs reorder → use atomics).
