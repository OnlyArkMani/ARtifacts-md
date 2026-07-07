# 02 — Two Pointers

## 1. Pattern Overview

Two indices moving through a sequence with **coordinated logic**, so you examine each element a constant number of times instead of comparing all pairs.

Why it works: when data is **sorted (or has monotonic structure)**, moving a pointer tells you something definitive about *all* the pairs you just skipped. In `l + r` too small → *no* partner for `nums[l]` exists further left, so `l++` safely discards it. That's the whole trick: **each pointer move eliminates a batch of candidates provably, not hopefully.**

Variants:
- **Opposite ends** (converging): sorted-pair sums, palindromes, container problems.
- **Same direction** (fast/slow or reader/writer): in-place de-duplication, cycle detection, partitioning.

**Complexity:** O(n) time (each pointer moves at most n steps), O(1) space. That O(1) space is often *why* this beats hashing.

## 2. Recognition Triggers

- Input is **sorted** (or you're allowed to sort) + "find pair/triplet with sum/property" → converging pointers.
- "In-place", "O(1) extra space" on an array problem → reader/writer pointers.
- "Palindrome" checks → converge from both ends.
- "Remove duplicates / move zeros / partition" → slow pointer = write position, fast pointer = read position.
- "Container/trapping water", "max area" between ends → converging with a greedy move rule.
- Linked list: "find middle", "detect cycle", "nth from end" → fast/slow (see `06`).
- kSum family (3Sum, 4Sum) → sort + fix outer elements + two-pointer the innermost pair.
- Comparing two sequences/merging sorted arrays → one pointer per sequence.
- Constraint smell: n ≤ 10^5 and O(n²) too slow, but data sortable and answer about pairs → sort O(n log n) + two pointers O(n).

## 3. Template Code

### Converging pointers (Java)
```java
// WHY: sorted order means moving l right strictly increases the sum,
// moving r left strictly decreases it — so every move is forced, never a guess.
public int[] twoSumSorted(int[] nums, int target) {
    int l = 0, r = nums.length - 1;
    while (l < r) {
        int sum = nums[l] + nums[r];
        if (sum == target) return new int[]{l, r};
        if (sum < target) l++;   // nums[l] can't pair with ANY remaining r → discard it
        else r--;                // nums[r] can't pair with ANY remaining l → discard it
    }
    return new int[]{-1, -1};
}
```

### Converging pointers (Python)
```python
def two_sum_sorted(nums: list[int], target: int) -> list[int]:
    l, r = 0, len(nums) - 1
    while l < r:
        s = nums[l] + nums[r]
        if s == target:
            return [l, r]
        if s < target:
            l += 1        # sum too small: only raising the low side can help
        else:
            r -= 1        # sum too big: only lowering the high side can help
    return [-1, -1]
```

### Reader/writer — in-place compaction (Java)
```java
// WHY: 'write' marks the boundary of the "kept" prefix. 'read' scans everything.
// Invariant: nums[0..write) is the answer-so-far, always valid.
public int removeDuplicates(int[] nums) {
    int write = 1;                          // first element always kept
    for (int read = 1; read < nums.length; read++) {
        if (nums[read] != nums[write - 1]) // differs from last kept → keep it
            nums[write++] = nums[read];
    }
    return write;                           // length of kept prefix
}
```

### Reader/writer (Python)
```python
def remove_duplicates(nums: list[int]) -> int:
    write = 1
    for read in range(1, len(nums)):
        if nums[read] != nums[write - 1]:   # compare to last KEPT value, not nums[read-1]
            nums[write] = nums[read]
            write += 1
    return write
```

## 4. Language-Specific Gotchas

- **Java:** no tuple swap — need a temp variable. `Arrays.sort(int[])` is fine; `Arrays.sort(Integer[])` needed for custom comparators.
- **Python:** `nums[l], nums[r] = nums[r], nums[l]` swaps cleanly. Beware `sorted(nums)` returns a *copy* while `nums.sort()` mutates — pick deliberately when "in-place" matters.
- **Python:** negative indexing (`s[-1]`) can silently "work" and hide an off-by-one bug that would crash in Java. If your pointer went below 0 you *want* the crash.
- **Java:** `s.charAt(i)` char comparisons vs Python string slicing; Python `s[::-1] == s` palindrome check is O(n) space — interviewers may want the pointer version.
- Both: with strings + case/alphanumeric filters (Valid Palindrome), Java `Character.isLetterOrDigit` / `Character.toLowerCase` vs Python `c.isalnum()` / `c.lower()`.
- Loop condition `l < r` vs `l <= r`: converging pair problems want `l < r` (a pointer meeting itself isn't a pair).

## 5. Worked Examples

### Easy — Valid Palindrome (LC 125)

**Problem:** Return true if `s` is a palindrome considering only alphanumeric characters, ignoring case.

**Recognition trace:** "Palindrome" → compare from both ends. Filtering non-alphanumerics inline avoids O(n) extra space of building a cleaned string.

**Java:**
```java
public boolean isPalindrome(String s) {
    int l = 0, r = s.length() - 1;
    while (l < r) {
        while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;  // skip junk
        while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
        if (Character.toLowerCase(s.charAt(l)) != Character.toLowerCase(s.charAt(r)))
            return false;
        l++; r--;
    }
    return true;
}
```

**Python:**
```python
def is_palindrome(s: str) -> bool:
    l, r = 0, len(s) - 1
    while l < r:
        while l < r and not s[l].isalnum():
            l += 1
        while l < r and not s[r].isalnum():
            r -= 1
        if s[l].lower() != s[r].lower():
            return False
        l += 1
        r -= 1
    return True
```

**Trace** for `"A man, a plan"` → cleaned view `amanaplan`... (excerpt on `"race a car"`):
```
r a c e   a   c a r
l=0(r) r=8(r) ✓ → l=1(a) r=7(a) ✓ → l=2(c) r=6(c) ✓ → l=3(e) r=5(a) ✗ → false
```

### Medium — 3Sum (LC 15)

**Problem:** Find all unique triplets summing to 0.

**Recognition trace:** Triplet sum → fix one element, the rest is Two Sum on a sorted array → sort + outer loop + converging pointers. Uniqueness → skip duplicate values at every level. O(n²) total.

**Java:**
```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length - 2; i++) {
        if (nums[i] > 0) break;                      // sorted: no negatives left to cancel
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate anchors
        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum < 0) l++;
            else if (sum > 0) r--;
            else {
                res.add(List.of(nums[i], nums[l], nums[r]));
                l++;
                while (l < r && nums[l] == nums[l - 1]) l++;  // skip dup on l side only
            }
        }
    }
    return res;
}
```

**Python:**
```python
def three_sum(nums: list[int]) -> list[list[int]]:
    nums.sort()
    res = []
    for i in range(len(nums) - 2):
        if nums[i] > 0:
            break                       # anchors only grow from here; sum can't be 0
        if i > 0 and nums[i] == nums[i - 1]:
            continue                    # same anchor value → same triplets → skip
        l, r = i + 1, len(nums) - 1
        while l < r:
            s = nums[i] + nums[l] + nums[r]
            if s < 0:
                l += 1
            elif s > 0:
                r -= 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                while l < r and nums[l] == nums[l - 1]:
                    l += 1              # skip duplicate second elements
    return res
```

**Trace** for `[-1, 0, 1, 2, -1, -4]` → sorted `[-4, -1, -1, 0, 1, 2]`:
```
i=0 (-4): l=1 r=5  sum=-3 <0 l++ ... never reaches 0 (max is -4+1+2=-1)
i=1 (-1): l=2 r=5  -1-1+2=0 ✓ [-1,-1,2]; l→3; -1+0+2=1>0 r--; -1+0+1=0 ✓ [-1,0,1]
i=2 (-1): duplicate anchor → skip
i=3 (0):  0+1+2=3 >0 r-- ... done
res = [[-1,-1,2], [-1,0,1]]
```

### Hard — Trapping Rain Water (LC 42)

**Problem:** Given elevation map `height[]`, compute total trapped water.

**Recognition trace:** Water above bar i = `min(maxLeft, maxRight) - height[i]`. Naive: precompute both max arrays (O(n) space). Two-pointer insight: **whichever side has the smaller current max is the binding constraint** — process that side, because its water level is already decided regardless of what's between the pointers.

**Java:**
```java
public int trap(int[] height) {
    int l = 0, r = height.length - 1;
    int maxL = 0, maxR = 0, water = 0;
    while (l < r) {
        if (height[l] < height[r]) {
            // left max is the limiting wall for position l — right side is
            // guaranteed to have something >= height[l], so min() is maxL.
            maxL = Math.max(maxL, height[l]);
            water += maxL - height[l];       // 0 if this bar IS the new max
            l++;
        } else {
            maxR = Math.max(maxR, height[r]);
            water += maxR - height[r];
            r--;
        }
    }
    return water;
}
```

**Python:**
```python
def trap(height: list[int]) -> int:
    l, r = 0, len(height) - 1
    max_l = max_r = water = 0
    while l < r:
        if height[l] < height[r]:
            max_l = max(max_l, height[l])
            water += max_l - height[l]   # bounded by left wall; right wall is taller
            l += 1
        else:
            max_r = max(max_r, height[r])
            water += max_r - height[r]
            r -= 1
    return water
```

**Trace** for `height = [0,1,0,2,1,0,1,3,2,1,2,1]` (first steps):
```
l=0(h0) r=11(h1): 0<1 → maxL=0, +0, l=1
l=1(h1) r=11(h1): 1≥1? no, 1<1 false → right: maxR=1, +0, r=10
l=1(h1) r=10(h2): 1<2 → maxL=1, +0, l=2
l=2(h0) r=10:     0<2 → maxL=1, +1  ← water trapped over index 2
... total = 6
```

## 6. Common Mistakes & Edge Cases

- Forgetting to **sort first** when the technique requires sorted input.
- Duplicate handling in kSum: skipping duplicates at the wrong level, or skipping *before* recording (missing valid triplets like `[-1,-1,2]`).
- `l <= r` vs `l < r` — pairing an element with itself.
- Inner skip loops missing the `l < r` guard → index out of bounds.
- Reader/writer: comparing `nums[read]` to `nums[read-1]` instead of the last **written** value.
- Returning values when indices were asked, or 0-indexed vs 1-indexed (LC 167 is 1-indexed!).
- Empty array / single element / all duplicates / already-all-same inputs.
- Trapping Rain Water: tie case `height[l] == height[r]` must go to exactly one branch (either is fine, both is not).

## 7. Fallback Map

- Data unsorted and sorting destroys required info (original indices) → **Arrays & Hashing** (`01`).
- Constraint is about a **contiguous window** with a size/aggregate condition → **Sliding Window** (`03`) — sliding window IS two same-direction pointers with a maintained invariant.
- Need to *search* for a value/boundary rather than pair elements → **Binary Search** (`05`).
- Pairs across two different arrays with ordering → merge-style pointers; if that fails, hashing.
- Need all pairs regardless of structure (no monotonicity to exploit) → accept O(n²) or rethink with math/counting.
