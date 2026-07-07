# 14 — Greedy

## 1. Pattern Overview

Greedy = at each step take the **locally best choice** and never look back. It's the fastest pattern when it works (usually one pass or sort + one pass) and confidently wrong when it doesn't — the entire skill is **knowing when greed is safe**.

Greed is safe when the problem has the **exchange property**: any optimal solution can be transformed, swap by swap, into the greedy one without getting worse. You rarely prove this formally in interviews; instead you argue: *"if the optimal solution didn't take my greedy choice, I could swap it in with no loss."* If you can't sketch that sentence — or you can find a counterexample — it's DP.

Recurring greedy shapes:
- **Sort by the right key, sweep once** (intervals by end time, deadlines, ratios). Picking the sort key IS the problem.
- **Running extremes** (max reach so far, min price so far, current best candidate).
- **Take best-available via heap** (scheduling: `09`).
- **Local reset** (Kadane: a negative running sum can only hurt — drop it; gas station: failing at i means no start in [start, i] works).

**Complexity:** O(n) or O(n log n) (the sort). That speed is the tell: if constraints demand O(n) on something that smells like optimization DP, look for the greedy insight.

## 2. Recognition Triggers

- "Maximum/minimum number of X you can do" where items don't interact after selection (non-overlapping intervals, task assignment).
- "Can you reach / is it possible" over jumps or resources → track the running best reach.
- "Minimum number of jumps/refuels/removals" → furthest-reach greedy or heap-assisted greedy.
- Sortable items where a comparator captures preference (deadline, end time, size ratio) → sort + sweep.
- "Maximum subarray" (Kadane), "best time to buy/sell" → running-extreme greedy.
- Assignment/pairing of two sorted groups (cookies to kids, tasks to workers) → sort both + two pointers.
- Constraint smell: n up to 10^5–10^6 with an "optimize" question — DP tables don't fit; the intended answer is an O(n log n) greedy.
- **Verification habit:** before committing, spend 30 seconds hunting a counterexample (especially with negative numbers, ties, or "coins" that aren't canonical — greedy coin-change fails on coins {1, 3, 4} for amount 6!). Counterexample found → DP (`12`).

## 3. Template Code

### Sort + sweep (Java) — activity selection shape
```java
// WHY sort by END: finishing earliest leaves maximal room for the rest —
// any schedule keeping a later-ending choice can swap in ours, no loss (exchange argument).
public int maxNonOverlapping(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[1], b[1]));  // by end time
    int count = 0, lastEnd = Integer.MIN_VALUE;
    for (int[] iv : intervals) {
        if (iv[0] >= lastEnd) {   // doesn't clash with what we've committed to
            count++;
            lastEnd = iv[1];      // commit; never reconsidered
        }
    }
    return count;
}
```

### Sort + sweep (Python)
```python
def max_non_overlapping(intervals: list[list[int]]) -> int:
    intervals.sort(key=lambda iv: iv[1])   # earliest END first — the safe greedy key
    count, last_end = 0, float("-inf")
    for start, end in intervals:
        if start >= last_end:              # compatible → take it, greedily
            count += 1
            last_end = end
    return count
```

### Running-extreme greedy (Python) — Kadane
```python
def max_subarray(nums: list[int]) -> int:
    best = cur = nums[0]
    for x in nums[1:]:
        cur = max(x, cur + x)   # a negative prefix can only drag x down → drop it
        best = max(best, cur)
    return best
```

## 4. Language-Specific Gotchas

- **Java comparator overflow:** `(a, b) -> a[1] - b[1]` breaks on large magnitudes → `Integer.compare`.
- **Python sort stability** (equal keys keep input order) can silently save or break tie-sensitive greedies; Java's `Arrays.sort` on objects is also stable, on primitives not (irrelevant for single keys, relevant for tuples).
- Multi-key sorts: Python `key=lambda x: (x[0], -x[1])` — negation only works for numbers; Java `Comparator.comparing(...).thenComparing(...)`.
- Python: `sort()` mutates; `sorted()` copies. Java: sorting `int[][]` is fine, but sorting primitives with custom order needs boxing.
- Kadane in Java: initialize with `nums[0]`, not 0 — all-negative arrays are the classic break.
- Heap-assisted greedy: see `09` gotchas (polarity, lazy deletion).

## 5. Worked Examples

### Easy — Best Time to Buy and Sell Stock II (LC 122)

**Problem:** Unlimited transactions (sell before rebuying). Max total profit.

**Recognition trace:** unlimited transactions → every price increase is independently capturable → sum all positive deltas. Exchange argument: any strategy's profit decomposes into daily deltas; taking all positive ones is an upper bound and achievable.

**Java:**
```java
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++)
        profit += Math.max(0, prices[i] - prices[i - 1]);  // capture every rise
    return profit;
}
```

**Python:**
```python
def max_profit(prices: list[int]) -> int:
    return sum(max(0, prices[i] - prices[i - 1])
               for i in range(1, len(prices)))
```

**Trace** for `[7, 1, 5, 3, 6, 4]`:
```
deltas:   -6  +4  -2  +3  -2
take:          4       3        → 7   (buy 1 sell 5, buy 3 sell 6)
```

### Medium — Jump Game (LC 55)

**Problem:** `nums[i]` = max jump length from i. Can you reach the last index?

**Recognition trace:** "can you reach" → track the single number that matters: **furthest reachable index**. If the sweep ever stands past it, you're stuck. Greedy safe because reach is monotone — more reach never hurts.

**Java:**
```java
public boolean canJump(int[] nums) {
    int reach = 0;                        // furthest index reachable so far
    for (int i = 0; i < nums.length; i++) {
        if (i > reach) return false;      // gap: this index is unreachable
        reach = Math.max(reach, i + nums[i]);
        if (reach >= nums.length - 1) return true;  // early exit
    }
    return true;
}
```

**Python:**
```python
def can_jump(nums: list[int]) -> bool:
    reach = 0
    for i, jump in enumerate(nums):
        if i > reach:                 # we're standing beyond anything reachable
            return False
        reach = max(reach, i + jump)
    return True
```

**Trace** for `nums = [2, 3, 1, 1, 4]` then `[3, 2, 1, 0, 4]`:
```
[2,3,1,1,4]: i=0 reach=2 | i=1 reach=4 ≥ last → True
[3,2,1,0,4]: i=0 reach=3 | i=1 reach=3 | i=2 reach=3 | i=3 reach=3 | i=4 > 3 → False
              index 4 sits beyond the frontier — the 0 at index 3 is a wall
```

### Hard — Candy (LC 135)

**Problem:** Children in a line with ratings. Each child ≥ 1 candy; a child with a higher rating than an adjacent neighbor must get more candy than that neighbor. Minimize total.

**Recognition trace:** two directional constraints (left neighbor, right neighbor). One greedy pass can't see both → **two sweeps**: left-to-right fixes ascending runs; right-to-left fixes descending runs; a child needs `max` of both demands. Greedy is safe because each pass gives the *minimum* satisfying its own direction, and max merges them without breaking either.

**Java:**
```java
public int candy(int[] ratings) {
    int n = ratings.length;
    int[] candies = new int[n];
    Arrays.fill(candies, 1);                        // everyone starts with 1
    for (int i = 1; i < n; i++)                     // enforce left-neighbor rule
        if (ratings[i] > ratings[i - 1])
            candies[i] = candies[i - 1] + 1;
    for (int i = n - 2; i >= 0; i--)                // enforce right-neighbor rule
        if (ratings[i] > ratings[i + 1])
            candies[i] = Math.max(candies[i],       // don't undo the left pass
                                  candies[i + 1] + 1);
    int total = 0;
    for (int c : candies) total += c;
    return total;
}
```

**Python:**
```python
def candy(ratings: list[int]) -> int:
    n = len(ratings)
    candies = [1] * n
    for i in range(1, n):                       # left-to-right: rising edges
        if ratings[i] > ratings[i - 1]:
            candies[i] = candies[i - 1] + 1
    for i in range(n - 2, -1, -1):              # right-to-left: falling edges
        if ratings[i] > ratings[i + 1]:
            candies[i] = max(candies[i], candies[i + 1] + 1)
    return sum(candies)
```

**Trace** for `ratings = [1, 3, 4, 5, 2]`:
```
ratings:  1  3  4  5  2
L→R:      1  2  3  4  1     (each rise: prev+1)
R→L:      1  2  3  4  1     → check falls: 5>2 → candies[3]=max(4, 1+1)=4 ✓ stays
total = 1+2+3+4+1 = 11
Now ratings [5,4,3]: L→R gives 1 1 1; R→L gives 3 2 1 → total 6 — right pass did the work.
```

## 6. Common Mistakes & Edge Cases

- **Assuming greed works without the exchange argument** — coin change {1,3,4} amount 6: greedy 4+1+1 = 3 coins; optimal 3+3 = 2. Thirty seconds of counterexample hunting saves the interview.
- Sorting intervals by **start** when the argument requires **end** (activity selection) — the most common wrong key.
- Kadane initialized to 0 → wrong on all-negative arrays.
- Ties: does `>=` vs `>` matter at boundaries (intervals touching)? Read the problem's definition of overlap.
- Two-pass problems (Candy): trying to do it in one pass, or letting the second pass overwrite the first (`max` is the fix).
- Comparator overflow (Java), reversed comparators.
- Local reset arguments (gas station): restarting the start pointer at `i + 1` without understanding *why* nothing in between can work.
- Empty/single-element inputs; already-sorted inputs (does your greedy degenerate?).

## 7. Fallback Map

- Found a counterexample to greed → **DP** (`12`/`13`) — same decision structure, provably correct.
- Choice interacts with the future in a bounded way ("at most k transactions") → DP with a small state dimension.
- Greedy needs "best available so far" dynamically → add a **Heap** (`09`).
- Greedy over intervals → the dedicated **Intervals** playbook (`15`).
- "Minimize the maximum" → **binary search the answer** (`05`), then the feasibility check is usually greedy — a powerful combo.
- Greedy + proof too slippery, n small (≤ 20) → **Backtracking** (`10`) brute force is a legitimate fallback.
