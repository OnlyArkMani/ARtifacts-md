# 20_SLIDING_WINDOW

1. Introduction

What this concept is

Sliding window is an algorithmic pattern to process contiguous subarrays or substrings by maintaining a window [left, right] that slides over the input. It enables solving many problems in linear time by expanding and contracting the window while maintaining an invariant.

Why it exists

To avoid nested loops when dealing with problems about contiguous sequences, e.g., longest substring with at most k distinct characters, maximum subarray of fixed length, and minimum window substring. Sliding window gives O(n) solutions where naive O(n^2) approaches exist.

What problem it solves

Efficiently compute properties over contiguous subarrays/substrings, whether fixed-size windows (move one step each) or variable-size windows (expand until invariant violated, then contract)

Real-world analogy

Reading a book with a fixed-size magnifier that you slide along the page, or a camera viewport scanning a panorama.

Where it is used in industry

- Text processing and search (minimum window match)
- Signal processing with moving averages (fixed-size window)
- Streaming analytics (running sums or sliding aggregates)


2. Intuition Section

Two flavors

- Fixed-size window: move window by 1 step each time and compute/update aggregate
- Variable-size window: extend right until condition broken, then move left until restored. Useful for constraints like "at most k distinct" or sum ≤ S.

ASCII variable window for longest subarray with sum ≤ S

[ l ...... r ]
expand r while sum ≤ S
if sum > S: move l until sum ≤ S


3. Core Theory

Invariant maintenance

- Maintain count/state for elements inside window (e.g., frequency map)
- Update answer when invariant satisfied (max/min length/value)

Typical implementations

- Use frequency map (dict) to count characters; maintain distinct count
- For sums use running sum and subtract on left contraction

Complexity

- Each element enters and leaves window at most once → O(n)
- Auxiliary space depends on alphabet or value domain (e.g., O(k) distinct characters)


4. Java Implementation

Longest substring with at most k distinct characters

```java
public int lengthOfLongestKDistinct(String s, int k) {
    Map<Character, Integer> freq = new HashMap<>();
    int l = 0, maxLen = 0;
    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        freq.put(c, freq.getOrDefault(c, 0) + 1);
        while (freq.size() > k) {
            char cl = s.charAt(l++);
            freq.put(cl, freq.get(cl) - 1);
            if (freq.get(cl) == 0) freq.remove(cl);
        }
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
```

Explain: expand r to grow window; shrink l until distinct count ≤ k.

Fixed-size window example: moving average

```java
double[] movingAverage(int[] a, int w) {
    int n = a.length; double[] res = new double[n-w+1];
    long sum = 0;
    for (int i = 0; i < n; i++) {
        sum += a[i];
        if (i >= w) sum -= a[i-w];
        if (i >= w-1) res[i-w+1] = sum/(double)w;
    }
    return res;
}
```


5. Python Implementation

Minimum window substring (classic)

```python
from collections import Counter

def min_window(s, t):
    need = Counter(t)
    missing = len(t)
    l = start = end = 0
    for r, ch in enumerate(s, 1):
        if need[ch] > 0:
            missing -= 1
        need[ch] -= 1
        if missing == 0:
            while l < r and need[s[l]] < 0:
                need[s[l]] += 1
                l += 1
            if end == 0 or r - l < end - start:
                start, end = l, r
            need[s[l]] += 1
            missing += 1
            l += 1
    return s[start:end]
```

Explain: maintain needed counts and missing counter; once all required chars present, shrink window greedily.


6. Internal Working

Why linear

- Each character is processed twice at most: once when r includes it, once when l excludes it — hence O(n). Frequency updates are O(1) amortized.

Space

- Frequency map size depends on alphabet distinctness (O(k) if constraint k), or O(min(n, alphabet)) worst-case.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Fixed window (move by 1) | O(n) |
| Variable window (expand/contract) | O(n) |
| Space | O(k) where k is distinct elements in window |


8. Space Complexity

- O(k) for frequency counters; O(1) for numeric windows


9. Common Interview Questions

Beginner

- Sliding window for maximum sum subarray of size k

Intermediate

- Minimum window substring
- Longest substring without repeating characters

Advanced

- Sliding window with additional constraints (multiple conditions combined), or streaming variations with limited memory


10. Common Mistakes

Mistake: forgetting to update counts correctly on contraction leading to off-by-one
Fix: carefully update the data structure and test small examples

Mistake: using substring concatenation inside loop leading to O(n^2) string copying
Fix: use indices and return slice once done


11. Real Interview Traps

Trap: using sliding window on problems where property is not monotonic — window approach may not apply

Trap: misunderstanding "subarray" vs "subset" — sliding window only handles contiguous sequences


12. Real World Applications

- Streaming analytics and moving averages
- Text search engines and substring matching


13. Common LeetCode Problems

Easy
- Maximum average subarray I (fixed size)
- Longest substring without repeating characters

Medium
- Minimum window substring
- Subarray sum equals k (uses hash + prefix sums or sliding window in case of positives only)

Hard
- Complex streaming sliding-window problems with multiple conditions


14. Pattern Recognition

When to use sliding window

- Problem asks about contiguous subsequences
- You can maintain invariants with counts/sums and expand/contract to preserve them

Decision checklist

- Is condition monotonic as you expand? → sliding window likely works
- Are values positive (for sum-based windows)? → simplifies to easier contraction decisions


15. Comparison Section

Sliding Window vs Two-Pointers

- Sliding window is a type of two-pointer technique focused on contiguous windows and invariants. Two-pointers covers a broader set including opposite ends and merge-like problems.


16. 5 Minute Revision

Sliding Window

- Maintain window [l, r], expand/right and contract/left
- Good for contiguous sequence problems, O(n) time
- Use freq maps for character constraints and sum counters for numeric constraints


---
