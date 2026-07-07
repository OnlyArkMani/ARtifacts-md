# 00 — Master Index & Decision Tree

Your pre-interview refresher. If you read one file the morning of an interview, read this one.

## The Files

| # | File | Pattern |
|---|---|---|
| 01 | `01_ARRAYS_AND_HASHING.md` | Arrays & Hashing (+ prefix sums) |
| 02 | `02_TWO_POINTERS.md` | Two Pointers |
| 03 | `03_SLIDING_WINDOW.md` | Sliding Window |
| 04 | `04_STACK.md` | Stack & Monotonic Stack |
| 05 | `05_BINARY_SEARCH.md` | Binary Search (+ answer-space) |
| 06 | `06_LINKED_LIST.md` | Linked List |
| 07 | `07_TREES_BFS_DFS.md` | Trees (DFS/BFS/BST) |
| 08 | `08_TRIES.md` | Tries |
| 09 | `09_HEAPS_PRIORITY_QUEUE.md` | Heaps / Priority Queue |
| 10 | `10_BACKTRACKING.md` | Backtracking |
| 11 | `11_GRAPHS.md` | Graphs (BFS/DFS/Union-Find/Topo/Dijkstra) |
| 12 | `12_1D_DYNAMIC_PROGRAMMING.md` | 1D DP |
| 13 | `13_2D_DYNAMIC_PROGRAMMING.md` | 2D DP |
| 14 | `14_GREEDY.md` | Greedy |
| 15 | `15_INTERVALS.md` | Intervals |
| 16 | `16_MATH_BIT_MANIPULATION.md` | Math & Bit Manipulation |

---

## 1. Learning Order (Conceptual Dependencies)

Learn in this sequence — each stage builds machinery the next one assumes:

```
STAGE 1 — Foundations (everything depends on these)
  01 Arrays & Hashing  →  02 Two Pointers  →  03 Sliding Window
  (03 is literally 02 with maintained state; 02 assumes 01's cost model)

STAGE 2 — Core mechanics
  04 Stack  →  05 Binary Search  →  06 Linked List
  (independent of each other — do all three; 05 unlocks answer-space thinking
   used later in Greedy/DP alternatives)

STAGE 3 — Recursion habitat
  07 Trees  →  10 Backtracking  →  11 Graphs
  (Trees teach recursion trust; Backtracking = recursion + undo;
   Graphs = trees + visited-set + 4 named algorithms. Do 07 before 10 before 11.)

STAGE 4 — Derived structures (quick wins after Stage 3)
  08 Tries (short — trees applied to strings)
  09 Heaps (short — feeds directly into Dijkstra & Intervals-hard)

STAGE 5 — Optimization thinking (hardest conceptual jump; do last)
  14 Greedy  →  12 1D DP  →  13 2D DP
  (Greedy first: DP is what you do when greedy breaks — learning them in this
   order teaches you the counterexample habit that tells them apart)

ANYTIME FILLERS (low dependency, slot into gaps)
  15 Intervals (needs 14's sort-and-sweep instinct)
  16 Math/Bits (needs nothing; memorize the trick table)
```

**Time-boxing per stage** (if you have ~4 weeks at ~2h/day): Stage 1: 5 days. Stage 2: 5 days. Stage 3: 8 days. Stage 4: 3 days. Stage 5: 7 days. Anytime: interleave. Solve 3–5 problems per pattern: the file's worked Easy + Medium, then fresh Mediums from NeetCode 150's category until you can start from a blank editor and reach the template unprompted.

---

## 2. Priority Tiers (Where to Spend Limited Time)

**Tier 1 — Must-master (highest interview frequency; ~70% of questions touch these):**

- 01 Arrays & Hashing — the substrate of everything
- 02 Two Pointers / 03 Sliding Window — the most common "optimal solution" mechanism
- 07 Trees — the most common data-structure question, period
- 11 Graphs (BFS/DFS + grids at minimum) — every company asks graphs now
- 05 Binary Search — cheap to master, disproportionately asked
- 09 Heaps — "top K" is a perennial

**Tier 2 — Important (regularly asked; differentiates mid vs strong candidates):**

- 04 Stack (monotonic stack is a frequent Medium)
- 12 1D DP (House Robber / Coin Change / LIS shapes)
- 10 Backtracking (subsets/permutations/combination-sum trio)
- 06 Linked List (easy to prep, embarrassing to fumble)
- 15 Intervals (short pattern, common at Big Tech)
- 14 Greedy (mostly about judgment; pairs with 12)

**Tier 3 — Advanced / lower frequency (prep after Tiers 1–2 are solid):**

- 13 2D DP (asked at DP-heavy companies; hard to fake)
- 11-advanced: Union-Find, Topological Sort, Dijkstra (know the templates cold, they're mechanical)
- 08 Tries (a handful of canonical problems; high leverage per hour because the question pool is tiny)
- 16 Math/Bits (memorize the trick table; don't over-invest)

**If you have one week:** Tier 1 only, 2 problems/pattern/day, re-derive templates from memory each morning.

---

## 3. Master Decision Tree

Read the problem, extract: input type, output ask, constraint sizes. Then walk this tree.

### Step 0 — Constraint sniff (do this FIRST, it prunes everything)

```
n ≤ 20        → exponential OK → Backtracking (10) or bitmask DP
n ≤ 500       → O(n³) OK      → interval DP (13), Floyd-Warshall
n ≤ 5,000     → O(n²) OK      → 2-seq DP (13), quadratic two-pointer, LIS n²
n ≤ 10^5–10^6 → need O(n log n) or O(n) → sort/heap/binary search/
                 sliding window/hashing/monotonic stack/greedy
n ≥ 10^9 (a VALUE, not a size) → O(log): binary search (05), math (16),
                 fast exponentiation — you can't even touch each unit
"O(1) space" demanded → two pointers (02), bit tricks (16), in-place marking (01-hard)
```

### Step 1 — Route by input type

```
INPUT IS...
├─ Array/string, question about CONTIGUOUS runs
│    ├─ window with a validity rule ("longest/shortest/at most K") → SLIDING WINDOW (03)
│    ├─ subarray SUM to target (negatives present) → PREFIX SUM + HASHMAP (01)
│    └─ "next greater/smaller", spans, histograms → MONOTONIC STACK (04)
│
├─ Array, question about PAIRS/TRIPLES or in-place rearrangement
│    ├─ sorted (or sortable, indices irrelevant) → TWO POINTERS (02)
│    └─ unsorted, indices matter → HASHMAP complement lookup (01)
│
├─ Array, "kth / top K / most frequent / median of stream" → HEAP (09)
│    └─ one-shot kth, mutation allowed → quickselect (09-fallback)
│
├─ Sorted anything / "minimize the max" / feasibility threshold → BINARY SEARCH (05)
│
├─ Intervals [start, end] → INTERVALS (15); "max non-overlap" → GREEDY by end (14)
│
├─ Linked list → LINKED LIST (06): dummy node + fast/slow + reversal — that's 90%
│
├─ Tree
│    ├─ "level / nearest / zigzag" → BFS (07)
│    ├─ anything else → DFS (07): decide top-down (pass constraints)
│    │                            vs bottom-up (return subtree answers)
│    └─ BST → exploit sorted in-order / prune by bounds (07)
│
├─ Words + prefixes / autocomplete / many-words-in-grid → TRIE (08)
│
├─ Graph, grid, or "states with transitions"
│    ├─ fewest steps, unweighted → BFS (11)   [multi-source? seed them all]
│    ├─ weighted shortest path → DIJKSTRA (11)
│    ├─ ordering with prerequisites / cycle in DAG → TOPO SORT (11)
│    ├─ dynamic connectivity / merge groups → UNION-FIND (11)
│    └─ explore/count regions → DFS flood fill (11)
│
└─ No structure, pure logic/numbers → MATH/BITS (16)
```

### Step 2 — Route by output ask (when Step 1 is ambiguous)

```
ASKED FOR...
├─ "ALL solutions/combinations/paths" (output itself exponential) → BACKTRACKING (10)
├─ "COUNT the ways" (just the number, n large) → DP (12/13)
├─ "MAX/MIN value" of sequential decisions
│    ├─ sketch the exchange argument… holds? → GREEDY (14)
│    └─ found a counterexample → DP (12: one index; 13: two indices/interval)
├─ "Is it POSSIBLE" → reachability (BFS/DFS 11) or boolean DP (12) or greedy reach (14)
└─ "MINIMIZE THE MAXIMUM / maximize the minimum" → BINARY SEARCH the answer (05)
     with a greedy feasibility check (14) — the classic combo
```

### Step 3 — Sanity checks before coding

```
□ Complexity of chosen approach fits Step 0's budget?
□ Edge cases: empty, size 1, all-equal, all-negative, overflow (Java!), duplicates?
□ Can I state the invariant in one sentence? (window validity / stack order /
  dp[i] meaning / loop bound meaning) — if not, I don't understand my own plan yet.
```

---

## 4. Company / Interview-Style Emphasis Map

General knowledge, changes over time — treat as directional, and check recent interview reports for your target company on levels.fyi / Blind / Glassdoor.

| Company style | Emphasis |
|---|---|
| **Google** | Graphs, DP (incl. 2D), clean problem decomposition, follow-up scaling questions. Less template-y; expect novel phrasings of `11`/`12`/`13`. |
| **Meta** | Speed + polish on Tier 1: arrays/hashing, two pointers, trees, BFS/DFS, intervals. Often 2 mediums in 35–40 min — templates must be automatic. |
| **Amazon** | Practical mediums: hashmaps, BFS/DFS grids, heaps/top-K, intervals; leadership-principle narration while coding. `01`, `09`, `11`, `15`. |
| **Microsoft** | Balanced classics: linked lists, trees, strings, moderate DP. `06`, `07`, `12`. |
| **Apple** | Arrays/strings, design-flavored DS questions (LRU, iterators), trees. `01`, `06`-design, `07`. |
| **Netflix / senior roles** | Fewer puzzles, more design; DSA rounds skew Tier 1 mediums done impeccably. |
| **Quant/HFT (Jane Street, Citadel, HRT)** | Math, probability, bits, heaps, clever O(1)/O(log) tricks. `16`, `09`, `05` — plus mental arithmetic speed. |
| **Startups / mid-size** | NeetCode-150-style mediums straight from the list: `01`–`05`, `07`, `11`, `12`. |
| **India services / walk-ins (TCS→product transitions)** | Arrays, strings, sorting, basic DP — Tier 1 + `12` easies at speed. |

Interview-style notes: phone screens skew Tier 1 easies/mediums; onsites add one hard (usually graph, DP, or heap-hybrid); "design" rounds love `06`+`01` combos (LRU) and `09` (rate limiters, medians).

---

## 5. The 10-Minute Pre-Interview Refresher

1. Reread **Step 0 constraint table** above — sizes → complexity → pattern family.
2. Skim each file's **Recognition Triggers** section only (~30s each).
3. Recite the five invariants: window validity (03) · monotonic stack order (04) · binary search half-open `[lo, hi)` (05) · BFS-marks-at-enqueue (11) · dp[i] in words before code (12).
4. Java: `Integer.compare` not subtraction · `long` for sums · `.equals()` for boxed. Python: copy before appending paths · `deque` for BFS · recursion limit.
5. Breathe. State the brute force out loud first — it buys thinking time and partial credit.
