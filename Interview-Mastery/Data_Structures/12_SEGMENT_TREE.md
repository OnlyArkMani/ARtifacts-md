# 12_SEGMENT_TREE

1. Introduction

What this concept is

A Segment Tree is a binary tree data structure used for answering range queries and performing updates on intervals of an array, typically in O(log n) per operation. Each node represents an interval (segment) of the array and stores aggregated information (sum, min, max, gcd, etc.) about that interval.

Why it exists

To provide fast queries and updates on contiguous ranges of a static or dynamic array where naive range recomputation is too slow (O(n) per query) and prefix-sum tricks are insufficient for arbitrary updates.

What problem it solves

Efficiently compute aggregate functions (sum, min, max) over ranges and support point updates or range updates with lazy propagation.

Real-world analogy

Think of a tournament scoreboard: each node stores results for a range of teams; updates to an individual team's score propagate up the tree, and queries about a range aggregate data from a small number of nodes.

Where used

- Competitive programming (range queries)
- Databases and analytics for range aggregation
- Graphics and geometry queries


2. Intuition Section

Imagine you have an array A of length n. Build a binary tree where the root represents A[0..n-1], its left child A[0..mid], right A[mid+1..n-1], recursively down to single elements at leaves.

ASCII for n=8

                 [0..7]
                /      \
           [0..3]      [4..7]
           /   \        /   \
        [0..1][2..3] [4..5][6..7]
        /\    /\      ...
      [0][1]

To query sum on [l..r], collect O(log n) nodes that exactly cover the interval.


3. Core Theory

Construction

Build tree recursively in O(n). Each node stores aggregate of children: node.value = combine(left.value, right.value)

Query

Range query runs recursively and aggregates values from nodes whose segments lie fully inside query range; partial overlaps descend.

Update

Point update: update leaf and update ancestors O(log n).
Range update: use lazy propagation to defer updates to children until necessary, storing a lazy value per node.

Lazy propagation

When an update affects whole node segment, update node value and mark lazy tag. Push tag to children only when needed (on subsequent queries/updates), ensuring amortized O(log n).

Space

Segment tree often stored in array of size 4*n for safe allocation.


4. Java Implementation

Simple segment tree for sum

```java
class SegmentTree {
    int n;
    long[] tree;

    SegmentTree(int[] arr) {
        n = arr.length;
        tree = new long[4*n];
        build(1, 0, n-1, arr);
    }

    void build(int idx, int l, int r, int[] arr) {
        if (l == r) { tree[idx] = arr[l]; return; }
        int mid = (l + r) >>> 1;
        build(idx<<1, l, mid, arr);
        build(idx<<1|1, mid+1, r, arr);
        tree[idx] = tree[idx<<1] + tree[idx<<1|1];
    }

    long query(int idx, int l, int r, int ql, int qr) {
        if (ql > r || qr < l) return 0; // neutral for sum
        if (ql <= l && r <= qr) return tree[idx];
        int mid = (l + r) >>> 1;
        return query(idx<<1, l, mid, ql, qr) + query(idx<<1|1, mid+1, r, ql, qr);
    }

    void update(int idx, int l, int r, int pos, int val) {
        if (l == r) { tree[idx] = val; return; }
        int mid = (l + r) >>> 1;
        if (pos <= mid) update(idx<<1, l, mid, pos, val);
        else update(idx<<1|1, mid+1, r, pos, val);
        tree[idx] = tree[idx<<1] + tree[idx<<1|1];
    }
}
```

Explain: index arithmetic uses 1-based root; children idx<<1, idx<<1|1. Build O(n), query/update O(log n).


5. Python Implementation

Simple python array-based implementation (recursive). Use lists sized 4*n.


6. Internal Working

Why O(log n)

A query breaks interval to at most O(log n) disjoint segments corresponding to tree nodes. Each node visited reduces segment size by ~2.

Why 4*n array

Safe bounding: worst-case number of nodes ≤ 4*n for full binary partition representation.

Lazy propagation details

Store pending update per node; when visiting child, push pending update to children and clear current node's lazy. Ensures each update or query visits O(log n) nodes.

Memory and locality

Array-based storage gives good cache behavior; node indices map compactly in memory.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Build     | O(n)       |
| Query     | O(log n)   |
| Update    | O(log n)   |

With lazy range updates still O(log n) per update (amortized) and O(log n) per query.


8. Space Complexity

- Total: O(n) actual data, tree array O(4n) overhead
- Auxiliary: O(log n) recursion


9. Common Interview Questions

- Range sum query with point updates
- Range add with range sum query using lazy propagation
- Range min/max queries


10. Common Mistakes

Mistake: not handling mid calculation properly leading to infinite recursion
Fix: use (l+r)>>>1 or l + (r-l)/2

Mistake: using tree size 2*n instead of 4*n and getting index out of bounds for non-power-of-two sizes


11. Real Interview Traps

Trap: forgetting neutral element (0 for sum, -inf for max) when out-of-range in query
Trap: using recursion without tail conditions causing stack overflow for large n; consider iterative segment trees or BIT for simpler operations


12. Real World Applications

- Time series range queries
- Game development for range effect computations
- Any system requiring frequent range aggregation and updates


13. Common LeetCode Problems

- Range Sum Query - Mutable (segment tree or BIT)
- Range Sum Query 2D (2D segment tree or BIT)


14. Pattern Recognition

Use segment tree when:
- You need arbitrary range queries (not just prefix)
- You need both updates and queries intermixed
- Array size is moderate (up to 1e5 typical in CP)

Decision tree

- If only prefix sums and point updates → Fenwick (BIT)
- If range updates or non-invertible aggregates → segment tree


15. Comparison Section

Fenwick vs Segment

- Fenwick (BIT): simpler, lower constant, supports prefix sums and point updates; harder for range updates without trickery
- Segment Tree: supports arbitrary range queries and range updates with lazy propagation; more flexible but heavier


16. 5 Minute Revision

Segment Tree

- Binary tree over array segments
- Build O(n), queries/updates O(log n)
- Use lazy propagation for range updates
- Use array of size ~4n


---
