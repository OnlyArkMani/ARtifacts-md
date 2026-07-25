# 28_CONCURRENCY_AND_SYNCHRONIZATION

1. Introduction

What this concept is

Concurrency and synchronization describe techniques to design and implement programs that execute multiple tasks simultaneously (threads, processes) while coordinating access to shared resources to preserve correctness.

Why it exists

To utilize multi-core CPUs and to structure responsive systems; synchronization prevents race conditions and ensures memory visibility.

What problem it solves

Manages concurrent access to shared state, avoids data races, deadlocks, and ensures liveness.

Where used

Servers, parallel algorithms, real-time systems, GUI applications.


2. Intuition & Key patterns

- Mutual exclusion (locks, mutexes)
- Atomic operations and lock-free programming
- Condition variables for signaling
- Read-write locks for many readers/few writers
- Thread pools and task queues


3. Java examples (ReentrantLock, synchronized, volatile)

```java
// simple synchronized counter
class Counter {
  private int c = 0;
  public synchronized void inc(){ c++; }
  public synchronized int get(){ return c; }
}
```

Explain: synchronized adds mutual exclusion and memory barriers.


4. Python examples (GIL, multiprocessing)

- CPython GIL prevents true parallelism in threads for CPU-bound code; use multiprocessing for CPU parallelism or use C extensions.


5. Common issues

- Data races, deadlocks, livelocks, starvation
- Memory visibility (use volatile or synchronized in Java)


6. Interview Qs

- Implement thread-safe queue
- Detect and prevent deadlocks; ordering locks consistently


7. 5-min revision

Use fine-grained locking, prefer immutable data, use higher-level concurrency primitives (Executors, async frameworks).