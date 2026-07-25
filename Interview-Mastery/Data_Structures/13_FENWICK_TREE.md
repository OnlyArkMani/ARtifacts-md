# 13_FENWICK_TREE

1. Introduction

What this concept is

A Fenwick Tree, also known as Binary Indexed Tree (BIT), is a compact data structure that supports prefix-sum queries and point updates in O(log n) time with O(n) space. It leverages binary decomposition of indices to store partial sums.

Why it exists

To provide a simpler and more space-efficient alternative to segment trees for prefix-based queries and updates, often with a lower constant factor and easier implementation.

What problem it solves

Efficiently computes prefix aggregates (sum, xor) and supports updating individual elements, which is useful when queries ask for prefix sums or differences between prefixes.

Real-world analogy

Picture a ledger where each page stores the sum of several previous pages. To compute a prefix sum, you look at a logarithmic number of ledger entries determined by the binary representation of the index.

Where used

- Competitive programming
- Frequency tables and cumulative sums
- In databases and analytics for incremental updates where prefix queries dominate


2. Intuition Section

Index decomposition

Indices have binary representation. A BIT stores partial sums for ranges sized as powers of two. For index i, BIT[i] stores sum from (i - lsb(i) + 1) to i where lsb(i) = i & -i.

Example (1-based indices):

i   binary  lsb  BIT range
1   001     1    [1..1]
2   010     2    [1..2]
3   011     1    [3..3]
4   100     4    [1..4]

To get prefix sum up to i, keep subtracting lsb(i) and accumulate BIT[i]. To update position i by delta, add delta to BIT indices obtained by repeatedly adding lsb(i).

ASCII flow for query(6)

6 -> add BIT[6], i -= lsb(6)=2 => 4
4 -> add BIT[4], i -= 4 => 0 stop


3. Core Theory

Operations

- pointUpdate(i, delta): while i <= n: BIT[i] += delta; i += lsb(i)
- prefixSum(i): res=0; while i>0: res += BIT[i]; i -= lsb(i)

Why it works

BIT stores overlapping ranges of sizes powers of two. Using binary indexing you can decompose any prefix into O(log n) stored ranges.

Variants

- Range update, point query via difference trick
- Range update, range query via two BITs


4. Java Implementation

```java
class Fenwick {
    int n;
    long[] bit;
    Fenwick(int n) { this.n = n; bit = new long[n+1]; }
    void add(int idx, long delta) {
        for (; idx <= n; idx += idx & -idx) bit[idx] += delta;
    }
    long sum(int idx) {
        long res = 0;
        for (; idx > 0; idx -= idx & -idx) res += bit[idx];
        return res;
    }
    long rangeSum(int l, int r) { return sum(r) - sum(l-1); }
}
```

Explain: 1-based indices are common; using idx & -idx computes LSB.


5. Python Implementation

```python
class Fenwick:
    def __init__(self, n):
        self.n = n
        self.bit = [0]*(n+1)
    def add(self, idx, delta):
        while idx <= self.n:
            self.bit[idx] += delta
            idx += idx & -idx
    def sum(self, idx):
        res = 0
        while idx > 0:
            res += self.bit[idx]
            idx -= idx & -idx
        return res
    def range_sum(self, l, r):
        return self.sum(r) - self.sum(l-1)
```


6. Internal Working

Binary decomposition

The BIT structure is built on powers-of-two spans. Each index points to previous indices in a linked fashion determined by lsb increments/decrements.

Space efficiency

Uses O(n) space with single array and lower constants than segment trees.

Why O(log n)

Each update or sum moves by adding/subtracting lsb(i), which clears one set bit per step, so at most O(log n) steps.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Update    | O(log n)   |
| Query     | O(log n)   |
| RangeSum  | O(log n)   |


8. Space Complexity

- Total: O(n)
- Aux: O(1)


9. Common Interview Questions

- Implement BIT for prefix sums
- Use BIT to count inversions in array (process from right to left using frequency BIT)
- Support range add and range sum using two BITs


10. Common Mistakes

Mistake: using 0-based indices directly
Fix: either convert to 1-based or carefully adapt lsb logic for 0-based

Mistake: forgetting to use long/64-bit for large sums


11. Real Interview Traps

Trap: thinking BIT supports arbitrary associative functions beyond sums/xor that are not invertible. BIT relies on invertibility for range queries using prefix sums.

Trap: confusing range-update techniques — two common patterns exist (difference-array trick vs two BITs) and you must pick the right one for the problem.


12. Real World Applications

- Counting inversions
- Frequency accumulations and cumulative histograms
- Dynamic prefix analytics


13. Common LeetCode Problems

- Count of Smaller Numbers After Self (use BIT on coordinate-compressed values)
- Range Sum Query - Mutable (alternative uses)


14. Pattern Recognition

Use BIT when:
- You need prefix sums and point updates
- You want lightweight and fast implementation
- Problem size is large and constants matter


15. Comparison Section

Fenwick vs Segment

- BIT: simpler, smaller memory footprint, only built-in prefix operations
- Segment tree: more flexible (arbitrary ranges, non-invertible ops, lazy propagation)


16. 5 Minute Revision

Fenwick Tree (BIT)

- Supports prefixSum and pointUpdate in O(log n)
- Uses lsb(i) = i & -i to traverse significant index jumps
- Use 1-based indices for simplicity


---
