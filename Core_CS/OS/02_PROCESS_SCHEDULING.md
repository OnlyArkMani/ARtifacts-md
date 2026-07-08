# Process Scheduling

> **Tag: Theory + light numericals** — know each algorithm's one-line identity, tradeoffs, and be able to compute average waiting time on a small trace.

## Concept

The scheduler decides which ready process gets the CPU next. Goals in tension: **throughput** (jobs/sec), **turnaround time** (submit→finish), **waiting time** (time in ready queue), **response time** (submit→first output), **fairness**. **Preemptive** scheduling can yank the CPU away (timer interrupt); **non-preemptive** waits for the process to yield/block.

Key vocab: **burst time** (CPU time needed), **arrival time**, **starvation** (a process waits forever), **aging** (gradually raise priority of waiters to cure starvation).

## The Algorithms

| Algorithm | Idea | Preemptive? | Pros | Cons |
|---|---|---|---|---|
| **FCFS** | Run in arrival order | No | Simple, fair-ish | **Convoy effect**: short jobs stuck behind long one |
| **SJF** | Shortest burst first | No | Provably minimal avg waiting time | Needs burst prediction; starves long jobs |
| **SRTF** (preemptive SJF) | Shortest *remaining* time first | Yes | Even better avg waiting | More switches; starvation |
| **Priority** | Highest priority first | Either | Expresses importance | Starvation → fix with **aging** |
| **Round Robin** | Each gets time quantum q, then back of queue | Yes | Great response time, fair | q too small → switch overhead; too large → FCFS |
| **MLFQ** (multilevel feedback) | Multiple queues; new jobs start high; CPU hogs sink lower | Yes | Adapts, favors interactive — real OSes' basis | Complex tuning |

**Quantum rule of thumb:** q should be large vs context-switch cost (~80% of bursts complete within q).

## Worked Numerical (the standard exam/interview trace)

Processes: P1(arrival 0, burst 8), P2(arrival 1, burst 4), P3(arrival 2, burst 2).

**FCFS:** order P1,P2,P3 → finish 8,12,14 → waiting: P1=0, P2=8−1=7, P3=12−2=10 → **avg 5.67**

**SRTF:** t0 P1 runs; t1 P2 arrives (4<7 remaining) → preempt; t2 P3 arrives (2<3) → preempt; P3 done t4; P2 resumes done t7; P1 resumes done t14.
Waiting = turnaround − burst: P1=(14−0)−8=6, P2=(7−1)−4=2, P3=(4−2)−2=0 → **avg 2.67** — SRTF halves FCFS here; that comparison is the expected takeaway.

**RR (q=2):** P1(0–2) P2(2–4) P3(4–6) P1(6–8) P2(8–10) P1(10–14) → waiting: P1=6, P2=5, P3=2 → avg 4.33.

Method: draw the Gantt chart, then waiting = (finish − arrival − burst).

## Most-Asked Interview Questions

1. **Preemptive vs non-preemptive?** Timer-driven forced switches vs voluntary yield; preemption gives responsiveness at the cost of switch overhead + synchronization complexity.
2. **Which algorithm gives minimum average waiting time?** SJF/SRTF (provably optimal) — but burst times must be known/predicted (exponential averaging of history), and long jobs starve.
3. **What is the convoy effect?** In FCFS, one CPU-bound process makes everyone queue behind it — like a truck on a one-lane road. I/O-bound processes suffer most.
4. **Round Robin quantum tradeoff?** Small q → interactive feel, high overhead; huge q → degenerates to FCFS. 
5. **How do real OSes schedule?** (Linux) CFS — Completely Fair Scheduler: red-black tree ordered by virtual runtime; picks the process with least vruntime; priorities via weights (nice values). MLFQ-like behavior emerges.
6. **What is starvation and how is it fixed?** Low-priority processes never run → **aging** (priority rises with wait time).
7. **I/O-bound vs CPU-bound — why does the mix matter?** Good schedulers favor I/O-bound (short bursts) to keep devices busy; MLFQ does this automatically as CPU hogs sink.
