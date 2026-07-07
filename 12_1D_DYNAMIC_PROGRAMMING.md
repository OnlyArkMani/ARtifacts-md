# 12 — 1D Dynamic Programming

## 1. Pattern Overview

DP = **recursion + memory**. When a brute-force recursion solves the same subproblem repeatedly (overlapping subproblems) and the best answer to the whole is built from best answers to parts (optimal substructure), caching collapses an exponential tree into a polynomial table.

The reliable 4-step derivation (never start by "writing a DP table"):

1. **Define the state in words.** "`dp[i]` = the answer to the problem for the first i items / ending exactly at i." If you can't say it in a sentence, stop.
2. **Find the recurrence.** How does `dp[i]` follow from earlier states? This is just: "what was the last choice?"
3. **Base cases.** The smallest states with obvious answers.
4. **Order + space.** Fill so dependencies are ready before use; if `dp[i]` only needs `dp[i-1], dp[i-2]`, shrink the array to two variables.

Two flavors of state definition — choosing the right one is half the battle:
- **Prefix state:** `dp[i]` = answer for the first i elements (House Robber, Climbing Stairs).
- **Ending-at state:** `dp[i]` = answer for subarrays/subsequences *ending exactly at i* (LIS, Max Subarray) — needed when the answer requires continuity/reference to the last element; final answer = max over all i.

**Complexity:** typically O(n) or O(n²) time, O(n) space → often reducible to O(1).

## 2. Recognition Triggers

- **"Count the number of ways..."** → DP, nearly always (backtracking counts by enumerating — too slow).
- **"Minimum/maximum cost/value to reach/achieve..."** over sequential choices → DP.
- "Can you reach / is it possible to form..." (partition, word break) → boolean DP.
- Choices at each step + **can't take adjacent / conflicting items** → House-Robber-shaped: `dp[i] = max(dp[i-1], dp[i-2] + val[i])`.
- Fibonacci-shaped ("ways to climb", "decode messages") → `dp[i]` from `dp[i-1]`, `dp[i-2]`.
- "Longest increasing/valid subsequence" → ending-at DP.
- Target sum / subset-sum / coin change → knapsack family: `dp[amount]` looping over items.
- n ≤ 10^3–10^4 with O(n²) allowed, or n ≤ 10^5 needing O(n) → sizes fit DP; contrast with n ≤ 20 (backtracking) and n ≥ 10^5 with pairwise interaction (greedy/sliding window).
- The brute-force recursion has ≤ 2–3 changing parameters → memoize it: that IS the DP.
- **Anti-trigger:** subproblems don't overlap (each explored once) → plain recursion/divide & conquer; local choice provably safe → greedy (`14`).

## 3. Template Code

### Bottom-up with space compression (Java)
```java
// House Robber shape — the most reusable 1D template.
// dp[i] = best using the first i houses; only i-1 and i-2 are ever read → 2 vars.
public int rob(int[] nums) {
    int prev2 = 0, prev1 = 0;          // dp[i-2], dp[i-1]
    for (int num : nums) {
        int cur = Math.max(prev1,       // skip this house: carry best-so-far
                           prev2 + num); // rob it: best up to i-2, plus this
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

### Top-down memoization (Python)
```python
from functools import lru_cache

def rob(nums: list[int]) -> int:
    # Top-down reads as the DECISION: rob house i or not.
    @lru_cache(maxsize=None)
    def best(i: int) -> int:
        if i < 0:
            return 0
        return max(best(i - 1),            # skip house i
                   best(i - 2) + nums[i])  # rob house i
    return best(len(nums) - 1)
```

### Unbounded knapsack shape (Python) — Coin Change
```python
def coin_change(coins: list[int], amount: int) -> int:
    INF = float("inf")
    dp = [0] + [INF] * amount           # dp[a] = fewest coins to make amount a
    for a in range(1, amount + 1):
        for c in coins:                 # last coin used was c → subproblem a-c
            if c <= a and dp[a - c] + 1 < dp[a]:
                dp[a] = dp[a - c] + 1
    return dp[amount] if dp[amount] != INF else -1
```

**Loop-order rule for knapsacks (memorize):**
- items outer, amount **inner ascending** → each item usable unlimited times (coin change).
- items outer, amount **inner descending** → each item once (0/1 knapsack, partition).
- amount outer, items inner → counts **permutations** (combination-sum-IV); items outer counts **combinations**. Order of loops = whether order matters.

## 4. Language-Specific Gotchas

- **Python:** `@lru_cache` makes top-down nearly free — but the recursion limit (1000) bites at n ≥ ~1000: convert to bottom-up for large n. Java has no built-in memoizer: use a `HashMap` or an `int[]` filled with a sentinel (−1).
- **Java:** `int[]` defaults to 0 — dangerous when 0 is a legal answer; use −1 sentinel or `Integer[]` with null.
- **Python:** `[[0]*n]*m` aliases rows (2D trap that leaks into 1D-of-lists); use comprehensions.
- Infinity: Java `Integer.MAX_VALUE` overflows when you add to it — guard `if (dp[x] != Integer.MAX_VALUE)` or use `MAX_VALUE / 2`. Python `float('inf')` is safe to add to.
- `lru_cache` on list arguments fails (unhashable) — pass indices, not slices. Slicing (`s[i:]`) in recursion also silently makes each call O(n) — pass `i`.
- Java streams in hot DP loops → measurable slowdowns; plain loops in interviews.

## 5. Worked Examples

### Easy — Climbing Stairs (LC 70)

**Problem:** n stairs, climb 1 or 2 at a time. How many distinct ways to reach the top?

**Recognition trace:** "how many ways" → DP. Last move was 1 or 2 steps → `ways(n) = ways(n-1) + ways(n-2)`. Fibonacci with different base cases.

**Java:**
```java
public int climbStairs(int n) {
    int prev2 = 1, prev1 = 1;          // ways(0)=1 (stand still), ways(1)=1
    for (int i = 2; i <= n; i++) {
        int cur = prev1 + prev2;       // arrive via 1-step from i-1, or 2-step from i-2
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

**Python:**
```python
def climb_stairs(n: int) -> int:
    prev2, prev1 = 1, 1                # ways(0), ways(1)
    for _ in range(2, n + 1):
        prev2, prev1 = prev1, prev1 + prev2
    return prev1
```

**Trace** for n = 5:
```
i:      0  1  2  3  4  5
ways:   1  1  2  3  5  8   ← each = sum of previous two
```

### Medium — House Robber (LC 198)

**Problem:** Rob houses on a street (values `nums`), no two adjacent. Maximize loot.

**Recognition trace:** max value + adjacency constraint → the canonical skip-or-take DP. State: `dp[i]` = best over the first i houses. Recurrence: skip i (`dp[i-1]`) or take i (`dp[i-2] + nums[i]`). Templates above are the full solution.

**Trace** for `nums = [2, 7, 9, 3, 1]`:
```
house:   2    7    9    3    1
dp:      2    7   11   11   12
              │    │         └ max(11, 11+1)=12  (take 1 + best through house 9's index)
              │    └ max(7, 2+9)=11
              └ max(2, 0+7)=7
answer 12  (rob 2 + 9 + 1)
```

### Hard — Longest Increasing Subsequence (LC 300, incl. O(n log n))

**Problem:** Length of the longest strictly increasing subsequence.

**Recognition trace:** "longest subsequence" + ordering condition → ending-at DP: `dp[i]` = LIS ending exactly at i → O(n²). The interview-differentiating follow-up is the O(n log n) **patience sorting** view: keep `tails[k]` = smallest possible tail of any increasing subsequence of length k+1; each element replaces (binary search) or extends. `tails` stays sorted → binary search is valid.

**Java (O(n log n)):**
```java
public int lengthOfLIS(int[] nums) {
    // tails[k] = smallest tail among all increasing subsequences of length k+1.
    // Smaller tails = more room to grow later → always keep the smallest.
    int[] tails = new int[nums.length];
    int size = 0;
    for (int x : nums) {
        int lo = 0, hi = size;               // lower_bound: first tails[i] >= x
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (tails[mid] >= x) hi = mid;
            else lo = mid + 1;
        }
        tails[lo] = x;                        // replace (tighten) or append (extend)
        if (lo == size) size++;
    }
    return size;
}
```

**Python (both versions):**
```python
def length_of_lis_n2(nums: list[int]) -> int:
    n = len(nums)
    dp = [1] * n                        # each element alone = length 1
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:       # subsequence ending at j can extend to i
                dp[i] = max(dp[i], dp[j] + 1)
    return max(dp)

import bisect
def length_of_lis(nums: list[int]) -> int:
    tails = []                           # tails[k] = min tail of an LIS of length k+1
    for x in nums:
        i = bisect.bisect_left(tails, x) # first tail >= x
        if i == len(tails):
            tails.append(x)              # x extends the longest subsequence
        else:
            tails[i] = x                 # x tightens a tail → future elements benefit
    return len(tails)
```

**Trace** for `nums = [10, 9, 2, 5, 3, 7, 101, 18]`:
```
x=10 : tails [10]
x=9  : replaces 10       → [9]
x=2  : replaces 9        → [2]
x=5  : appends           → [2, 5]
x=3  : replaces 5        → [2, 3]        ← tighten: same length, better future
x=7  : appends           → [2, 3, 7]
x=101: appends           → [2, 3, 7, 101]
x=18 : replaces 101      → [2, 3, 7, 18]
answer = len(tails) = 4   (e.g., 2,3,7,18)
NOTE: tails is NOT itself the LIS — only its length is guaranteed.
```

## 6. Common Mistakes & Edge Cases

- Vague state definition — if `dp[i]` means "something about the first i things" you'll write a wrong recurrence. Say it precisely, including "ending at i" vs "among first i."
- Ending-at problems: returning `dp[n-1]` instead of `max(dp)`.
- Base cases: `ways(0) = 1` (empty way) vs 0 — reason it from the recurrence, don't guess.
- Knapsack loop order/direction wrong → items reused when they shouldn't be (or counting permutations as combinations).
- `Integer.MAX_VALUE + 1` overflow in min-DP.
- Off-by-one between "i-th element" and "first i elements" indexing (`dp[i]` ↔ `nums[i-1]`).
- Forgetting LIS strictness (`<` vs `<=` → `bisect_left` vs `bisect_right`).
- Space-compressing before the 2D version works — get it right, then shrink.
- Empty input; single element; all-negative values (max-subarray-style: initialize with `nums[0]`, not 0).

## 7. Fallback Map

- State needs **two** independent indices (two strings, grid position) → **2D DP** (`13`).
- Local exchange argument proves a best choice → **Greedy** (`14`) — try to break greedy with a counterexample first; if you can't, greed wins and is faster.
- "Count ways" but n is tiny and you need the actual objects → **Backtracking** (`10`).
- Optimal answer over subarrays with monotone window validity → **Sliding Window** (`03`) is O(n) and simpler.
- "Minimize the max" phrasing → **binary search on answer** (`05`) + greedy check often replaces a hard DP.
- Recurrence over a DAG's nodes → **topological-order DP** (`11`).
- If the recurrence needs "best over all j < i with property P" and O(n²) is too slow → segment tree / monotonic structure — mention, rarely required.
