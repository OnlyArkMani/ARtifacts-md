# 03 — Sliding Window

## 1. Pattern Overview

A special case of same-direction two pointers: maintain a **contiguous window** `[l, r]` over the input plus some **state describing the window** (a sum, a char-count map, a max). Grow `r` to include elements; when the window breaks the rule, shrink from `l` until it's valid again.

Why it works: instead of recomputing every subarray from scratch (O(n²) subarrays × O(n) each), you **incrementally update state** — each element enters the window once and leaves once → O(n) total. The pattern is valid whenever the window property is **monotonic**: if a window is invalid, every *larger* window containing it is also invalid (so shrinking is the only sensible fix, and you never need to re-expand what you gave up).

Two flavors:
- **Fixed size window:** slide a size-k window; update state by adding `r`, removing `l`.
- **Variable size window:** expand always, shrink while invalid; record best valid window.

**Complexity:** O(n) time (amortized — the `l` pointer only moves forward), O(k) space for window state (often O(1) or O(26)).

## 2. Recognition Triggers

- The words **"contiguous subarray"** or **"substring"** — the #1 signal.
- "Longest/shortest substring/subarray that satisfies X" → variable window.
- "Max/min sum of subarray of size k", "average of every window" → fixed window.
- "At most K distinct / at most K replacements / at most K zeros" → variable window, shrink when the "budget" is exceeded.
- "Minimum window containing all of X" → variable window, shrink while *valid* (inverted: record while shrinking).
- Stream of data + "over the last k items" → fixed window.
- Constraint smell: n up to 10^5–10^6 and question is about substrings/subarrays → O(n²) enumeration is dead; sliding window is the intended O(n).
- **Anti-trigger:** if the array has negative numbers and you're windowing on *sum* with a target, monotonicity breaks → use prefix sums + hashmap (`01`) instead.
- "Exactly K" tricks: `exactly(K) = atMost(K) - atMost(K-1)`.

## 3. Template Code

### Variable-size window (Java)
```java
// Canonical shape: for each r, shrink until valid, then record.
// Invariant AFTER the while-loop: window [l..r] is valid.
public int longestWindow(int[] nums) {
    int l = 0, best = 0;
    // windowState: whatever you need to test validity in O(1)
    for (int r = 0; r < nums.length; r++) {
        add(nums[r]);                       // 1. window grows: absorb nums[r]
        while (windowInvalid()) {           // 2. restore the invariant
            remove(nums[l]);                //    evict from the left
            l++;
        }
        best = Math.max(best, r - l + 1);   // 3. window valid: record answer
    }
    return best;
}
```

### Variable-size window (Python)
```python
def longest_window(nums: list[int]) -> int:
    l = 0
    best = 0
    # state = ...                        e.g. Counter(), running sum
    for r, x in enumerate(nums):
        # add x to state
        while window_invalid():          # shrink until the rule holds again
            # remove nums[l] from state
            l += 1
        best = max(best, r - l + 1)      # every (l, r) here is a valid window
    return best
```

### Fixed-size window (Python — same shape in Java)
```python
def max_sum_size_k(nums: list[int], k: int) -> int:
    window = sum(nums[:k])
    best = window
    for r in range(k, len(nums)):
        window += nums[r] - nums[r - k]   # one in, one out — O(1) update
        best = max(best, window)
    return best
```

### "Minimum window" variant — shrink while VALID (Python)
```python
def min_window_shape(s: str) -> int:
    l = 0
    best = float("inf")
    for r, c in enumerate(s):
        # add c
        while window_valid():             # NOTE: inverted vs. longest-window
            best = min(best, r - l + 1)   # record BEFORE breaking validity
            # remove s[l]
            l += 1
    return best
```

## 4. Language-Specific Gotchas

- **Python `Counter`:** `counts[c] -= 1` leaves zero-count keys in the dict — `len(counts)` then lies about distinct count. Either `del counts[c]` when it hits 0, or track distinct separately.
- **Java:** use `int[26]`/`int[128]` for char windows — much faster and less error-prone than `HashMap<Character,Integer>`. Python's `Counter` equality (`window == need`) is O(26) and handy but Java map equality needs care with zero-count entries.
- **Java Integer boxing** again: comparing map values with `==` breaks silently.
- Window length is `r - l + 1` (inclusive both ends). Writing `r - l` is the single most common bug in this pattern — pick one convention and never deviate.
- Python: `while` loop with `l` as closure state — no issue; Java: remember `l` must be declared outside the for-loop.
- Deque-based windows (max-in-window): Java `ArrayDeque`, Python `collections.deque`; both need "pop from back while smaller" logic — see LC 239.

## 5. Worked Examples

### Easy — Best Time to Buy and Sell Stock (LC 121)

**Problem:** `prices[i]` is the stock price on day i. Max profit from one buy then one sell.

**Recognition trace:** "Buy before sell" over a sequence → window between the running minimum (buy) and today (sell). Equivalent framing: keep the cheapest price seen so far as `l`; the "window" never needs to shrink more than jumping `l` to the new minimum.

**Java:**
```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, best = 0;
    for (int p : prices) {
        minPrice = Math.min(minPrice, p);      // best buy day so far
        best = Math.max(best, p - minPrice);   // sell today against best buy
    }
    return best;
}
```

**Python:**
```python
def max_profit(prices: list[int]) -> int:
    min_price = float("inf")
    best = 0
    for p in prices:
        min_price = min(min_price, p)   # cheapest buy up to today
        best = max(best, p - min_price) # profit if we sell today
    return best
```

**Trace** for `[7,1,5,3,6,4]`:
```
p=7: min=7 best=0 | p=1: min=1 best=0 | p=5: min=1 best=4
p=3: min=1 best=4 | p=6: min=1 best=5 | p=4: min=1 best=5  → 5
```

### Medium — Longest Substring Without Repeating Characters (LC 3)

**Problem:** Length of the longest substring of `s` with all distinct characters.

**Recognition trace:** "Longest substring" + a validity rule (no repeats) → variable window. State: set (or last-seen-index map) of chars in window. Invalid when the incoming char is already present → shrink.

**Java:**
```java
public int lengthOfLongestSubstring(String s) {
    Set<Character> window = new HashSet<>();
    int l = 0, best = 0;
    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        while (window.contains(c)) {     // duplicate → evict from left until gone
            window.remove(s.charAt(l++));
        }
        window.add(c);
        best = Math.max(best, r - l + 1);
    }
    return best;
}
```

**Python:**
```python
def length_of_longest_substring(s: str) -> int:
    window = set()
    l = 0
    best = 0
    for r, c in enumerate(s):
        while c in window:          # the duplicate must be inside [l..r-1]
            window.remove(s[l])
            l += 1
        window.add(c)
        best = max(best, r - l + 1)
    return best
```

**Trace** for `s = "abcabcbb"`:
```
r=0 a: [a]        l=0 best=1
r=1 b: [ab]       l=0 best=2
r=2 c: [abc]      l=0 best=3
r=3 a: dup → evict a, l=1 → [bca]  best=3
r=4 b: dup → evict b, l=2 → [cab]  best=3
r=5 c: dup → evict c, l=3 → [abc]  best=3
r=6 b: dup → evict a,b l=5 → [cb]  best=3
r=7 b: dup → evict c,b l=7 → [b]   best=3   → 3
window as [l────r] over  a b c a b c b b
```

### Hard — Minimum Window Substring (LC 76)

**Problem:** Return the smallest substring of `s` containing every character of `t` (with multiplicity).

**Recognition trace:** "Minimum window containing all of X" → variable window, inverted: expand until valid, then **shrink while still valid**, recording each time. State: `need` counts from t, plus `have`/`needCount` satisfied-character counters so validity checks are O(1) instead of O(26).

**Java:**
```java
public String minWindow(String s, String t) {
    if (t.length() > s.length()) return "";
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;
    int required = t.length();           // chars still missing (with multiplicity)
    int l = 0, bestLen = Integer.MAX_VALUE, bestL = 0;

    for (int r = 0; r < s.length(); r++) {
        if (need[s.charAt(r)]-- > 0) required--;  // this char was still needed
        while (required == 0) {                    // valid → try to shrink
            if (r - l + 1 < bestLen) { bestLen = r - l + 1; bestL = l; }
            if (++need[s.charAt(l)] > 0) required++; // evicting a needed char breaks it
            l++;
        }
    }
    return bestLen == Integer.MAX_VALUE ? "" : s.substring(bestL, bestL + bestLen);
}
```

**Python:**
```python
from collections import Counter

def min_window(s: str, t: str) -> str:
    if len(t) > len(s):
        return ""
    need = Counter(t)
    required = len(t)                 # total chars (with multiplicity) still missing
    l = 0
    best = (float("inf"), 0)          # (length, start)

    for r, c in enumerate(s):
        if need[c] > 0:               # c was genuinely needed
            required -= 1
        need[c] -= 1                  # may go negative = "surplus" of c in window
        while required == 0:          # window covers t → shrink from left
            if r - l + 1 < best[0]:
                best = (r - l + 1, l)
            need[s[l]] += 1
            if need[s[l]] > 0:        # we just evicted a char we truly needed
                required += 1
            l += 1
    return "" if best[0] == float("inf") else s[best[1]: best[1] + best[0]]
```

**Trace** for `s = "ADOBECODEBANC"`, `t = "ABC"`:
```
expand → A D O B E C   required hits 0 at r=5   window [0..5] "ADOBEC" len 6 ★
shrink: evict A → required=1, l=1
expand → O D E B A     required 0 at r=10 (A)   window [1..10] shrink →
  evict D,O,B,E,C? : evicting keeps validity until C leaves at l=5 →
  best window seen while shrinking: [5..10] "CODEBA" len 6
expand → N C           required 0 at r=12       shrink to [9..12] "BANC" len 4 ★★
answer: "BANC"
```

## 6. Common Mistakes & Edge Cases

- `r - l` vs `r - l + 1` for window length.
- Shrinking with `if` instead of `while` — a single eviction may not restore validity.
- Minimum-window problems: recording the answer *outside* the shrink loop (records invalid/oversized windows).
- Counter going negative in Python and being misread as "missing" — decide whether negative means *surplus* and stay consistent.
- Zero-count keys inflating `len(counter)`.
- `t` longer than `s`; empty `t`; window never valid (return "" / 0 — know which).
- Negative numbers with a sum-target window → monotonicity broken, pattern silently wrong (use prefix sums).
- Fixed window: off-by-one when initializing the first window (sum of `nums[0..k-1]`, then start loop at `r = k`).

## 7. Fallback Map

- Subarray sum with **negative numbers** or "count subarrays summing to k" → **prefix sum + hashmap** (`01`).
- Not contiguous ("subsequence")? Sliding window is wrong → **DP** (`12`/`13`) or greedy.
- Need max/min *within* each window efficiently → **monotonic deque** (hybrid with `04` ideas).
- Pairs from both ends / sorted data → plain **Two Pointers** (`02`).
- Window over answers-space instead of index-space ("min largest sum splitting array") → **Binary search on the answer** (`05`).
