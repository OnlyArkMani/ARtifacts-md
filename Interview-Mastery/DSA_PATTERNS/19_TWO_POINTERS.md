# 19_TWO_POINTERS

1. Introduction

What this concept is

Two-pointers is an algorithmic technique that uses two indices (pointers) moving through a data structure — typically an array or linked list — to solve problems efficiently in linear time while often using constant extra space.

Why it exists

It reduces the need for nested loops by coordinating two pointers to scan the data structure in one pass or in linear passes, enabling O(n) solutions to problems that naive approaches solve in O(n^2).

What problem it solves

Common problems: removing duplicates, partitioning, finding pairs with given sum in sorted arrays, sliding-window-like tasks, and merging sorted arrays.

Real-world analogy

Two people walking along a beach from either end looking for meeting points or treasures. One moves forward while the other might move backward depending on conditions.

Where it is used in industry

- Array processing, streaming algorithms
- Two-sum on sorted arrays, merging steps of merge sort, removing duplicates in-place


2. Intuition Section

Basic patterns

- Fast and slow pointer (tortoise and hare): find cycle in linked lists
- Left and right pointers on sorted arrays: find pair with sum k by moving pointers inward
- Window boundaries: maintain window [l, r] with invariants

ASCII: two-sum on sorted array

arr: [1,3,4,6,8]
 l             r
1             8
sum = 9
if sum>target -> move r--, else l++


3. Core Theory

Common two-pointer patterns

- Opposite ends on sorted array: move inward based on comparison
- Slow-fast pointers: fast moves k steps ahead; used for cycle detection and to find kth from end
- Window expansion/contraction: one pointer expands window, other contracts when invariant violated
- Merge-like pointers: merging two sorted arrays by comparing current elements

Complexity

- Usually O(n) time and O(1) auxiliary space (ignoring output storage).


4. Java Implementation

Two-sum on sorted array

```java
int[] twoSumSorted(int[] a, int target) {
    int l = 0, r = a.length - 1;
    while (l < r) {
        int s = a[l] + a[r];
        if (s == target) return new int[]{l, r};
        if (s < target) l++; else r--;
    }
    return null;
}
```

Explain: sorted property allows single-pass solution; moving pointers reduces search space.

Slow-fast cycle detection

```java
boolean hasCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

Explain: fast moves two steps; if cycle exists they meet.


5. Python Implementation

Two-sum sorted

```python
def two_sum_sorted(a, target):
    l, r = 0, len(a)-1
    while l < r:
        s = a[l] + a[r]
        if s == target: return (l, r)
        if s < target: l += 1
        else: r -= 1
    return None
```

Kth from end (fast/slow)

```python
def kth_from_end(head, k):
    fast = slow = head
    for _ in range(k):
        if not fast: return None
        fast = fast.next
    while fast:
        slow = slow.next
        fast = fast.next
    return slow
```


6. Internal Working

Why it’s linear

Each pointer moves monotonically in one direction (or bounded number of times), so total moves ≤ c*n.

Memory

Typically O(1) auxiliary memory because pointers are indices or references.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Pair-sum (sorted) | O(n) |
| Remove duplicates in-place | O(n) |
| Kth from end (linked list) | O(n) |


8. Space Complexity

- O(1) aux in most two-pointer patterns


9. Common Interview Questions

Beginner

- Two-sum for sorted arrays
- Remove duplicates from sorted array in-place

Intermediate

- Container with most water (two-pointer optimization)
- Partitioning by pivot (Dutch National Flag variants)

Advanced

- Use two-pointers to solve complex string/array windowing with multiple constraints (e.g., min window substring variants)


10. Common Mistakes

Mistake: moving the wrong pointer on equality or off-by-one in window boundaries
Fix: carefully write invariants and examples; dry-run on small arrays

Mistake: assuming unsorted arrays can be solved with basic two-pointers — sorting changes indices; for original indices use hash map or two-phase approach


11. Real Interview Traps

Trap: for linked lists, not checking nulls properly before accessing next/next.next -> NullPointerException

Trap: confusing two-pointer window patterns vs slow-fast; both use two pointers but different semantics


12. Real World Applications

- Streaming median/quantile approximations (variants)
- In-place array transformations, deduplication, merging


13. Common LeetCode Problems

Easy
- Remove Duplicates from Sorted Array
- Two Sum II - Input array is sorted

Medium
- Container With Most Water
- Minimum Window Substring (advanced sliding window + two-pointer ideas)

Hard
- Triplet and k-sum variants optimized with two-pointers after sorting


14. Pattern Recognition

Signs to use two-pointers

- Input is sorted or can be sorted cheaply and you need pair/partition
- Problem asks for k-from-end or cycle detection
- In-place rearrangement or streaming context where O(1) aux is desired

Decision checklist

- Are indices monotonic? → two-pointers likely useful
- Do you need the original order/indices? → be careful with sorting


15. Comparison Section

Two-pointers vs Hash table

- Two-pointers often faster with O(1) space but requires sorted input or monotonic properties
- Hash table gives O(n) time and O(n) space for unsorted two-sum preserving original indices

Two-pointers vs Sliding Window

- Sliding window is a specific two-pointer pattern where the left pointer contracts and right expands to maintain invariant


16. 5 Minute Revision

Two-pointers

- Use two indices to scan from ends or maintain window
- Common uses: sorted pair-sum, remove-duplicates, kth-from-end, cycle detection
- Time O(n), Aux O(1)


---
