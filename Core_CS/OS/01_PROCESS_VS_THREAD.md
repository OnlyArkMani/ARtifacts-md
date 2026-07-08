# Process vs Thread

> **Tag: Theory/recall** — pure explain-and-compare; almost guaranteed in any OS round.

## Concept

A **process** is a program in execution — the OS gives it its own address space, file descriptors, and resources. A **thread** is an execution stream *within* a process — its own stack, registers, and program counter, but **sharing** the process's memory (code, heap, globals) with sibling threads. Processes isolate; threads share. That single sentence answers half the follow-ups.

**PCB (Process Control Block):** kernel structure per process — PID, state, registers, memory maps, open files, scheduling info. Thread's TCB is much smaller (stack pointer, registers, state).

**Process states:** New → Ready → Running → (Waiting/Blocked on I/O) → Ready → ... → Terminated.

## Compare & Contrast

| | Process | Thread |
|---|---|---|
| Memory | Own address space | Shares address space of process |
| Creation cost | Heavy (new memory maps, ~ms) | Light (just stack + TCB) |
| Context switch | Expensive (memory map change, TLB flush) | Cheap (same address space, no TLB flush) |
| Communication | IPC needed (pipes, sockets, shared mem, message queues) | Direct via shared variables (needs synchronization!) |
| Failure isolation | One crash doesn't kill others | One thread's crash (e.g., segfault) kills whole process |
| Security | Isolated | No isolation between threads |
| Example | Chrome's per-tab processes | Web server's worker threads |

## Context Switch (follow-up favorite)

CPU switches from one process/thread to another: save current registers/PC into PCB/TCB → scheduler picks next → restore its state. Costs: direct (register save/restore, kernel entry) + indirect (**cache and TLB pollution** — the bigger cost). Thread switches within a process skip the address-space switch → much cheaper.

## User-level vs Kernel-level Threads

- **User-level:** managed by a runtime library, kernel sees one thread. Fast switching; but one blocking syscall blocks ALL threads, no true parallelism. (Green threads, goroutines are M:N hybrids.)
- **Kernel-level:** kernel schedules each (Linux pthreads). True parallelism on multicore; heavier operations.

## Most-Asked Interview Questions

1. **Difference between process and thread?** → the table above; lead with "isolation vs sharing."
2. **Why are threads called lightweight processes?** Creation, switching, and communication are all cheaper because the address space is shared.
3. **Why is process context switch costlier?** Address-space switch → page-table change → TLB flush → cache misses afterward.
4. **When use processes over threads?** Fault isolation (browser tabs), security boundaries, unrelated programs, bypassing runtime locks (Python multiprocessing vs GIL).
5. **What do threads share / not share?** Share: code, data/heap, open files. Private: stack, registers, PC, thread-local storage.
6. **What is a zombie process?** Terminated child whose exit status hasn't been `wait()`ed by parent — PCB lingers. **Orphan:** parent died first; init/systemd adopts it.
7. **fork() vs exec()?** `fork()` clones the calling process (copy-on-write); `exec()` replaces the current process image with a new program. Shell runs commands via fork+exec.
8. **What does fork() return?** 0 in the child, child's PID in the parent, −1 on failure — classic "what does this print" setup: `fork(); fork(); printf("hi")` prints 4 times.
