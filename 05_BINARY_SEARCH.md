# 05 — Binary Search

## 1. Pattern Overview

Binary search is not "find x in a sorted array." It is: **any time you can define a predicate that is `false false ... false true true ... true` over a search space, you can find the boundary in O(log n).** The sorted-array version is just the special case where the predicate is `arr[i] >= target`.

Why it works: monotonicity. If `check(mid)` is true and the predicate is monotone, *every* value right of mid is also true — one comparison eliminates half the space. That's the entire idea; everything else is bookkeeping.

Three levels of mastery:
1. **Search sorted data** for a value or insertion point.
2. **Search modified sorted data** (rotated arrays, 2D matrices, mountain arrays) — locate which half is "clean" and recurse into the right one.
3. **Binary search on the answer** — the search space is candidate *answers* (a speed, a capacity, a max-sum), and `check(x)` = "is x feasible?" (usually a greedy O(n) simulation). This is the level that unlocks Hard problems.

**Complexity:** O(log n) per search; answer-space search is O(n log(range)).

## 2. Recognition Triggers

- Input is **sorted** (or rotated-sorted, or "sorted then modified") → near-automatic.
- "Find first/last occurrence", "insertion position", "floor/ceiling of x" → boundary search.
- O(log n) explicitly required, or n up to 10^9+ (can't even iterate!) → must be log.
- **"Minimize the maximum" / "maximize the minimum"** → binary search on the answer. This phrasing is almost a guarantee (split array, Koko bananas, ship capacity, aggressive cows).
- "Least k such that condition holds" where the condition is monotone in k.
- Search space is a numeric *range* and feasibility is cheap to check → answer-space search.
- Matrix with sorted rows/columns → treat as 1D (fully sorted) or staircase search.
- Two sorted arrays + "median" or "kth smallest" → partition binary search.
- Constraint smell: n ≤ 10^5 but values up to 10^9, asked for an optimal threshold → binary search values, not indices.

## 3. Template Code

### The one true template — lower bound (Java)
```java
// Finds the FIRST index where check() is true ("leftmost true").
// Works for every variant if you can phrase the problem as a predicate boundary.
// Invariant: everything < lo is false, everything >= hi is... unknown;
// at exit lo == hi == first true (or n if none).
public int lowerBound(int[] arr, int target) {
    int lo = 0, hi = arr.length;          // hi is EXCLUSIVE → can return n = "not found"
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;     // avoids (lo+hi) int overflow in Java
        if (arr[mid] >= target) {         // check(mid): "is mid at-or-past the target?"
            hi = mid;                     // mid might be the answer — keep it in range
        } else {
            lo = mid + 1;                 // mid is definitely false — discard it
        }
    }
    return lo;                            // first index with arr[idx] >= target
}
```

### Lower bound (Python)
```python
def lower_bound(arr: list[int], target: int) -> int:
    lo, hi = 0, len(arr)              # half-open [lo, hi)
    while lo < hi:
        mid = (lo + hi) // 2          # no overflow risk in Python
        if arr[mid] >= target:        # predicate: monotone false...true
            hi = mid                  # mid could be the boundary → don't skip it
        else:
            lo = mid + 1              # mid is left of the boundary → skip it
    return lo
# stdlib equivalent: bisect.bisect_left(arr, target)
```

### Binary search on the answer (Python — identical shape in Java)
```python
def min_feasible_answer(lo: int, hi: int, feasible) -> int:
    # feasible(x) must be monotone: once true, stays true as x grows.
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid                  # mid works → try smaller
        else:
            lo = mid + 1              # mid fails → need bigger
    return lo                         # smallest feasible value
```

**Discipline that prevents 90% of binary search bugs:**
1. Decide the predicate and verify it's monotone.
2. Use half-open `[lo, hi)` with `while lo < hi` — no `lo <= hi`, no `mid ± 1` on the true side, no infinite loops.
3. `hi = mid` on true, `lo = mid + 1` on false. Always. Done.

## 4. Language-Specific Gotchas

- **Java:** `(lo + hi) / 2` overflows when both near `Integer.MAX_VALUE` — write `lo + (hi - lo) / 2`. (Python ints can't overflow.)
- **Python:** `bisect_left` = lower bound, `bisect_right` = upper bound (first strictly greater). Knowing these saves writing the loop — but be able to write it.
- **Java:** `Arrays.binarySearch` returns `-(insertionPoint) - 1` for misses — awkward; usually cleaner to hand-roll.
- **Python:** integer division `//` floors toward −∞; Java `/` truncates toward 0 — matters when lo can be negative (rare, but real for value-space searches over negatives).
- Both: on doubles/floats, loop a fixed 100 iterations or until `hi - lo < eps` instead of `lo < hi`.
- Python's arbitrary-size ints make answer-space bounds forgiving; in Java pick `long` for `hi` when capacities/sums can exceed int.

## 5. Worked Examples

### Easy — Binary Search (LC 704)

**Problem:** Sorted array, return index of `target` or −1.

**Recognition trace:** sorted + find → direct application. Use lower bound then verify.

**Java:**
```java
public int search(int[] nums, int target) {
    int lo = 0, hi = nums.length;
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (nums[mid] >= target) hi = mid;
        else lo = mid + 1;
    }
    return (lo < nums.length && nums[lo] == target) ? lo : -1;
}
```

**Python:**
```python
def search(nums: list[int], target: int) -> int:
    lo, hi = 0, len(nums)
    while lo < hi:
        mid = (lo + hi) // 2
        if nums[mid] >= target:
            hi = mid
        else:
            lo = mid + 1
    return lo if lo < len(nums) and nums[lo] == target else -1
```

**Trace** for `nums = [-1, 0, 3, 5, 9, 12]`, `target = 9`:
```
[lo=0, hi=6) mid=2 nums[2]=3  < 9 → lo=3
[lo=3, hi=6) mid=4 nums[4]=9 >= 9 → hi=4
[lo=3, hi=4) mid=3 nums[3]=5  < 9 → lo=4
lo==hi=4 → nums[4]==9 → return 4
```

### Medium — Koko Eating Bananas (LC 875)

**Problem:** `piles[i]` bananas per pile, `h` hours. Koko eats at speed `k` bananas/hour (one pile at a time; finishing a pile early wastes the hour). Find minimum `k` to finish in `h` hours.

**Recognition trace:** "Find the minimum speed such that ...feasible" → minimize over a monotone predicate → **binary search on the answer**. If speed k works, k+1 works too (monotone ✓). `feasible(k)` = total hours `Σ ceil(pile/k) ≤ h`, O(n) to check. Search k in `[1, max(piles)]`.

**Java:**
```java
public int minEatingSpeed(int[] piles, int h) {
    int lo = 1, hi = Arrays.stream(piles).max().getAsInt();
    while (lo < hi) {
        int mid = lo + (hi - lo) / 2;
        if (hoursNeeded(piles, mid) <= h) hi = mid;  // fast enough → try slower
        else lo = mid + 1;                            // too slow → must speed up
    }
    return lo;
}
private long hoursNeeded(int[] piles, int k) {
    long hours = 0;
    for (int p : piles) hours += (p + k - 1L) / k;   // ceil division, long to be safe
    return hours;
}
```

**Python:**
```python
import math

def min_eating_speed(piles: list[int], h: int) -> int:
    def feasible(k: int) -> bool:
        return sum(math.ceil(p / k) for p in piles) <= h

    lo, hi = 1, max(piles)
    while lo < hi:
        mid = (lo + hi) // 2
        if feasible(mid):
            hi = mid          # mid works; the answer is mid or smaller
        else:
            lo = mid + 1
    return lo
```

**Trace** for `piles = [3, 6, 7, 11]`, `h = 8`:
```
k range [1, 11]
mid=6:  hours = 1+1+2+2 = 6 ≤ 8 ✓ → hi=6
mid=3:  hours = 1+2+3+4 = 10 > 8 ✗ → lo=4
mid=5:  hours = 1+2+2+3 = 8 ≤ 8 ✓ → hi=5
mid=4:  hours = 1+2+2+3 = 8 ≤ 8 ✓ → hi=4
lo==hi=4 → answer 4
predicate over k:  1:✗ 2:✗ 3:✗ 4:✓ 5:✓ 6:✓ ... — classic F..F T..T boundary
```

### Hard — Median of Two Sorted Arrays (LC 4)

**Problem:** Two sorted arrays sizes m, n. Find the median in O(log(min(m,n))).

**Recognition trace:** O(log) demanded on two sorted arrays → binary search a **partition**, not a value. Cut both arrays so the left halves together hold `(m+n+1)/2` elements; the cut is correct when every left element ≤ every right element. Search the cut position in the smaller array; the other cut is forced.

**Java:**
```java
public double findMedianSortedArrays(int[] A, int[] B) {
    if (A.length > B.length) return findMedianSortedArrays(B, A); // search smaller
    int m = A.length, n = B.length;
    int half = (m + n + 1) / 2;              // size of combined left part
    int lo = 0, hi = m;                      // i = how many of A go left (0..m)
    while (lo < hi) {
        int i = lo + (hi - lo) / 2;
        int j = half - i;                    // B's cut is forced
        // predicate: "cut i is not too far right" == A[i-1] <= B[j] side is fine;
        // we search first i where A's left doesn't overshoot B's right.
        if (i < m && B[j - 1] > A[i]) lo = i + 1;   // A-left too small → move right
        else hi = i;
    }
    int i = lo, j = half - i;
    int leftA  = (i == 0) ? Integer.MIN_VALUE : A[i - 1];
    int leftB  = (j == 0) ? Integer.MIN_VALUE : B[j - 1];
    int rightA = (i == m) ? Integer.MAX_VALUE : A[i];
    int rightB = (j == n) ? Integer.MAX_VALUE : B[j];
    if (((m + n) & 1) == 1) return Math.max(leftA, leftB);
    return (Math.max(leftA, leftB) + Math.min(rightA, rightB)) / 2.0;
}
```

**Python:**
```python
def find_median_sorted_arrays(A: list[int], B: list[int]) -> float:
    if len(A) > len(B):
        A, B = B, A                        # binary search the smaller array
    m, n = len(A), len(B)
    half = (m + n + 1) // 2
    lo, hi = 0, m
    while lo < hi:
        i = (lo + hi) // 2                 # elements of A on the left side
        j = half - i                       # elements of B on the left side (forced)
        if i < m and B[j - 1] > A[i]:      # B's left leaks past A's right → need more A
            lo = i + 1
        else:
            hi = i
    i = lo
    j = half - i
    left_a  = A[i - 1] if i > 0 else float("-inf")
    left_b  = B[j - 1] if j > 0 else float("-inf")
    right_a = A[i]     if i < m else float("inf")
    right_b = B[j]     if j < n else float("inf")
    if (m + n) % 2:
        return float(max(left_a, left_b))
    return (max(left_a, left_b) + min(right_a, right_b)) / 2
```

**Trace** for `A = [1, 3]`, `B = [2, 4, 5]` (m=2, n=3, half=3):
```
search i ∈ [0, 2]
i=1, j=2:  B[1]=4 > A[1]=3 → A-left too short → lo=2
i=2, j=1:  i==m → predicate side: hi stays... lo==hi=2
final: i=2, j=1
  A: [1 3 | ]      leftA=3, rightA=+inf
  B: [2 | 4 5]     leftB=2, rightB=4
odd total (5) → median = max(3, 2) = 3
combined: 1 2 3 | 4 5  ✓
```

## 6. Common Mistakes & Edge Cases

- **Infinite loop:** `while lo <= hi` mixed with `hi = mid` — the half-open template avoids this class entirely.
- Off-by-one on which side keeps `mid`: the side where the predicate is *true* keeps it (`hi = mid`), the false side discards (`lo = mid + 1`).
- Java overflow in `(lo + hi) / 2` and in feasibility sums (use `long`).
- Testing `nums[lo]` without checking `lo < n` first (target greater than everything).
- Answer-space search: wrong bounds (lo must be a possibly-valid answer: speed ≥ 1, capacity ≥ max element).
- Non-monotone predicate — binary search silently returns garbage. Prove monotonicity in one sentence before coding.
- Rotated arrays: forgetting duplicates break the "which half is sorted" test (LC 81 degrades to O(n)).
- Floats: comparing with `==`; loop on iteration count instead.

## 7. Fallback Map

- Data unsorted and can't be sorted meaningfully → **Hashing** (`01`) for lookups, **Heap** (`09`) for running extremes.
- Predicate not monotone → can't binary search; consider **DP** (`12`) or brute force with pruning.
- "Kth smallest in a matrix/stream" → binary search on value works, but **Heap** (`09`) is often simpler.
- Search in trees → BST property = binary search structurally (`07`).
- Need *all* positions matching, not a boundary → scan or hashing.
- "Minimize max" where feasibility check itself is hard → maybe **Greedy** (`14`) or DP feasibility inside the search.
