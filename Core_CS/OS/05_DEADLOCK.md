# Deadlock

> **Tag: Theory + one applied algorithm (Banker's)** — the four conditions are a guaranteed question; Banker's shows up as a numerical.

## Concept

Deadlock: a set of processes each holding resources the others need, all waiting forever. Classic picture: P1 holds A wants B; P2 holds B wants A.

## The Four Necessary Conditions (Coffman) — memorize

1. **Mutual exclusion** — resource usable by one process at a time
2. **Hold and wait** — processes hold resources while waiting for more
3. **No preemption** — resources can't be forcibly taken
4. **Circular wait** — a cycle P1→P2→…→Pn→P1 of waiting

All four must hold simultaneously. Every strategy attacks one of them.

## The Four Strategies

| Strategy | Idea | Cost |
|---|---|---|
| **Prevention** | Break one condition by design | Conservative, wastes resources |
| **Avoidance** | Grant requests only if state stays "safe" (Banker's) | Needs advance max-claims; runtime checks |
| **Detection + recovery** | Let deadlock happen; find cycles; kill/rollback victims | Detection cost; lost work |
| **Ostrich** | Ignore it (most general-purpose OSes!) | Rare hangs accepted |

**Prevention in practice — the one that matters:** break circular wait by **global lock ordering** (always acquire locks in the same fixed order). This is the answer to "how do you prevent deadlock in your code."

## Banker's Algorithm (avoidance) — worked example

State is **safe** if there's an order in which all processes can finish with current + gradually-freed resources.

3 resource types (A,B,C), Available = (3,3,2):

| Proc | Allocated | Max | Need = Max−Alloc |
|---|---|---|---|
| P0 | 0 1 0 | 7 5 3 | 7 4 3 |
| P1 | 2 0 0 | 3 2 2 | 1 2 2 |
| P2 | 3 0 2 | 9 0 2 | 6 0 0 |
| P3 | 2 1 1 | 2 2 2 | 0 1 1 |
| P4 | 0 0 2 | 4 3 3 | 4 3 1 |

Run: Need(P1)=(1,2,2) ≤ (3,3,2) ✓ → P1 finishes, release → Avail=(5,3,2) → P3 ✓ → (7,4,3) → P4 ✓ → (7,4,5) → P0 ✓ → P2 ✓.
**Safe sequence exists: ⟨P1,P3,P4,P0,P2⟩ ⇒ safe state.** A request is granted only if the *resulting* state is still safe (pretend-grant, re-run the check).

## Detection & Recovery

- Single instance per resource: cycle in **wait-for graph** ⇔ deadlock.
- Multiple instances: matrix algorithm (like Banker's but with actual requests).
- Recovery: kill processes (all, or one-by-one cheapest-first) or preempt resources with rollback. Livelock caution: don't always pick the same victim (starvation).

## Deadlock vs Starvation vs Livelock

| | What happens | Cure |
|---|---|---|
| Deadlock | Blocked forever in a cycle | Break a Coffman condition |
| Starvation | Runnable but never chosen | Aging/fair queues |
| Livelock | Actively running but no progress (mutual retry dance) | Randomized backoff |

## Most-Asked Interview Questions

1. **Four conditions for deadlock?** → Coffman list, with one-line meanings. Follow-up: which is easiest to break in code? → circular wait, via lock ordering.
2. **Prevention vs avoidance?** Prevention: static design rule breaking a condition. Avoidance: dynamic per-request safety check (Banker's) using declared maximum needs.
3. **Run Banker's on this table.** → method above: find any process whose Need ≤ Available, "finish" it, add its allocation back, repeat. No candidate → unsafe.
4. **Safe vs unsafe state — is unsafe = deadlocked?** No: unsafe means deadlock is *possible* under some future requests, not certain.
5. **How do databases handle deadlock?** Detection: wait-for graph cycle → abort a victim transaction (cheapest to rollback) → auto-retry. Or timeouts. (Cross-ref DBMS concurrency file.)
6. **Write code that deadlocks / fix it.** Two threads, two locks, opposite order → fix by consistent ordering or `tryLock` with backoff.
7. **Why do most OSes ignore deadlock?** Checks are expensive, deadlocks rare, cost asymmetry — a reboot is cheaper than continuous detection (engineering pragmatism).
