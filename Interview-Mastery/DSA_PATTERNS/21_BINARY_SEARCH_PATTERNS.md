# 21_BINARY_SEARCH_PATTERNS

1. Introduction

What this concept is

Binary search is a divide-and-conquer algorithm to find an element or boundary in a sorted (or monotonic) array in O(log n) time. Beyond simple search, binary search patterns are used to find boundaries, optimize monotonic functions, or search in answer-space for numeric problems.

Why it exists

To provide logarithmic-time search for sorted datasets and to extend to problems where results are monotonic in a parameter (searching over the answer domain rather than indices).

What problem it solves

Efficient lookup, threshold finding (first/last occurrence), and parametric search (find minimal feasible value satisfying predicate) where monotonicity holds.

Real-world analogy

Searching a book's index for a term: split the pages in half, choose side where term may appear, repeat. Or tuning a parameter until a constraint flips from false to true using binary halving.

Where it is used in industry

- Database indexing and B-tree searches
- Parametric search in optimization, threshold finding in system tuning
- Algorithmic contest problems searching minimal feasible K


2. Intuition Section

Classic array binary search to find target

- Compare mid value; if equals, done; else pick left or right half. Each step halves the search space.

Boundary search example: find first index where arr[i] ≥ target (lower_bound)

- If arr[mid] < target → move left = mid+1
- Else → move right = mid
- Loop until left == right → left is answer


3. Core Theory

Variants

- Exact search: find if an element exists
- Lower bound: first element ≥ target
- Upper bound: first element > target
- Binary search on answer: when checking feasibility is monotonic in candidate value

Invariant design

- Maintain left and right bounds so the invariant narrows to correct answer
- Choose inclusive/exclusive ranges carefully to avoid infinite loops or off-by-one

Complexity

- Each step halves search space → O(log n) iterations


4. Java Implementation

Classic binary search

```java
int binarySearch(int[] a, int target) {
    int l = 0, r = a.length - 1;
    while (l <= r) {
        int mid = l + (r - l)/2;
        if (a[mid] == target) return mid;
        else if (a[mid] < target) l = mid + 1;
        else r = mid - 1;
    }
    return -1;
}
```

Lower bound

```java
int lowerBound(int[] a, int target) {
    int l = 0, r = a.length; // note r = n
    while (l < r) {
        int mid = l + (r - l)/2;
        if (a[mid] < target) l = mid + 1;
        else r = mid;
    }
    return l; // could be n (not found)
}
```

Binary search on answer (parametric)

- Example: minimize maximum subarray sum with k partitions — check predicate(mid) = can we split into ≤k pieces with sum ≤ mid?


5. Python Implementation

Use bisect module for lower/upper bounds, but understanding manual implementation is crucial in interviews.

Manual lower_bound

```python
def lower_bound(a, target):
    l, r = 0, len(a)
    while l < r:
        mid = (l + r)//2
        if a[mid] < target:
            l = mid + 1
        else:
            r = mid
    return l
```


6. Internal Working

Why halving works

Binary search relies on monotonic ordering to discard one half safely because the predicate's truth in one half excludes the other.

Precision issues

- For floating-point binary search, run fixed number of iterations or compare with epsilon; careful design avoids infinite loops.

Off-by-one pitfalls

- Inclusive/exclusive boundaries must be chosen to maintain invariants; tests on small arrays and edge cases required.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Exact search | O(log n) |
| Lower/upper bound | O(log n) |
| Binary search on answer (with predicate cost T) | O(log R * T) where R is search range |


8. Space Complexity

- O(1) auxiliary for iterative implementations


9. Common Interview Questions

Beginner

- Implement binary search
- Implement lower_bound/upper_bound

Intermediate

- Binary search on rotated sorted array (find pivot then search)
- Find first bad version (classic LeetCode problem)

Advanced

- Parametric search: minimize maximum load, find smallest radius, find minimum number of days etc.


10. Common Mistakes

Mistake: using (l+r)/2 causing overflow in languages with bounded integers
Fix: use l + (r-l)/2 or safe midpoint computation

Mistake: wrong loop condition (l <= r vs l < r) mismatch with inclusive/exclusive choices


11. Real Interview Traps

Trap: binary searching on non-monotonic functions — binary search invalid

Trap: forgetting edge-case where result is beyond array bounds (returning n vs -1)


12. Real World Applications

- Index lookups (B-trees), performance tuning using binary search on parameter space, quantile estimation by binary searching percentile thresholds


13. Common LeetCode Problems

Easy
- Binary Search
- First Bad Version

Medium
- Search in Rotated Sorted Array
- Find Peak Element (binary search on local maxima)

Hard
- Aggressive Cows (allocate stalls with minimum distance) — binary search on answer


14. Pattern Recognition

When to use binary search

- Input sorted or monotonic predicate exists
- Seeking boundary/threshold rather than exact value

Decision checklist

- Is predicate monotonic? → binary search on answer
- Is array sorted? → index binary search


15. Comparison Section

Binary Search vs Linear Search

- Binary: O(log n) but requires sorted data
- Linear: O(n) but works on unsorted

Binary search vs Hashing

- Hashing gives average O(1) existence checks but no ordering or boundary queries


16. 5 Minute Revision

Binary Search Patterns

- Use for exact search, lower/upper bounds, and parametric search
- Watch inclusive/exclusive boundaries and overflow
- Time O(log n), O(1) aux


---
