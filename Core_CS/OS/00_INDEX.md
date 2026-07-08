# OS — Index & Fast Revision

## Tiered Priority

**Tier 1 (asked constantly):**
- `01_PROCESS_VS_THREAD.md` — THE most common OS question
- `06_PROCESS_SYNCHRONIZATION.md` — race conditions, mutex vs semaphore, producer-consumer
- `05_DEADLOCK.md` — four conditions, prevention vs avoidance
- `03_MEMORY_MANAGEMENT.md` — paging vs segmentation, virtual memory, page fault

**Tier 2 (commonly asked):**
- `02_PROCESS_SCHEDULING.md` — algorithm comparisons + waiting-time traces
- `04_PAGE_REPLACEMENT.md` — LRU/FIFO traces, Belady's anomaly
- `07_CONCURRENCY_MULTITHREADING.md` — concurrency vs parallelism, volatile/atomics

**Tier 3 (occasional):**
- `08_FILE_SYSTEMS_DISK_SCHEDULING.md` — inodes, links, SCAN traces, journaling

## All Comparison Tables in One Scan

**Process vs Thread:** own address space / shared; heavy / light creation; TLB-flush switch / cheap switch; IPC / shared vars; crash-isolated / crash shared.

**Preemptive vs Non-preemptive scheduling:** timer-forced switches, responsive, sync complexity ↔ voluntary yield, simple, convoy-prone.

**Scheduling algorithms:** FCFS (convoy) · SJF/SRTF (optimal wait, starves long) · RR (response time, quantum tradeoff) · Priority (+aging vs starvation) · MLFQ (adaptive, real OSes).

**Paging vs Segmentation:** fixed/physical/internal-frag/invisible ↔ variable/logical/external-frag/visible.

**Page replacement:** FIFO (Belady's ✔) · LRU (no Belady, costly true impl) · OPT (benchmark) · Clock (practical LRU approx).

**Deadlock strategies:** Prevention (break a condition; lock ordering) · Avoidance (Banker's; safe states) · Detection (cycles; kill victim) · Ostrich (most OSes).

**Deadlock vs Starvation vs Livelock:** blocked-cycle / never-chosen / active-no-progress → break condition / aging / random backoff.

**Mutex vs Semaphore:** ownership + unlock-by-locker ↔ counter, signal-by-anyone, doubles as resource-counter/ordering tool.

**synchronized vs volatile vs Atomic:** exclusion+visibility / visibility only / lock-free CAS per-op.

**Concurrency vs Parallelism:** structure (interleaving OK) vs simultaneous multi-core execution.

**Hard vs Soft link:** same inode, survives delete, same FS ↔ path-file, can dangle, cross-FS.

**Disk scheduling:** FCFS · SSTF (starves) · SCAN/LOOK (elevator) · C-variants (uniform waits). SSDs: none of this matters — wear leveling instead.

## Cheat Numbers
Context switch ~1–10µs · page fault to disk ~ms (100k× RAM) · quantum ~10–100ms · EAT = h(TLB+mem) + (1−h)(TLB+2·mem).
