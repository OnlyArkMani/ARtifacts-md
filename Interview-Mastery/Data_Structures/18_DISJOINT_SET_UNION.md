# 18_DISJOINT_SET_UNION

1. Introduction

What this concept is

Disjoint Set Union (DSU), also called Union-Find, is a data structure that tracks a set of elements partitioned into a number of non-overlapping (disjoint) subsets. It supports two primary operations: Find (determine which subset a particular element belongs to) and Union (merge two subsets).

Why it exists

DSU exists to efficiently manage connectivity queries over a dynamic collection of elements — particularly when the only modifications are unions of sets. It provides near-constant-time queries for connectivity and is essential in graph algorithms and connectivity tracking.

What problem it solves

DSU solves the problem of maintaining components under unions and answering connectivity queries quickly. Instead of scanning elements, DSU uses parent pointers and optional heuristics to keep trees shallow.

Real-world analogy

Think of a social network where people form friend groups. Initially each person is alone; as friendships form (union), groups merge. DSU answers: "Are Alice and Bob in the same friend group?" quickly.

Where it is used in industry

- Kruskal’s algorithm for Minimum Spanning Tree (MST)
- Dynamic connectivity problems
- Image segmentation and union operations in computer vision
- Network connectivity and clustering


2. Intuition Section

Visual

Start: each element is its own set (rooted tree of one node):

1   2   3   4

Union(1,2):

  1
  |
  2   3   4

Union(3,4):

  1    3
  |    |
  2    4

Union(1,3):

    1
   / \
  2   3
       \
        4

Path compression during "find" flattens trees by pointing nodes directly to their roots.

ASCII path compression

Before: 4 -> 3 -> 1
After find(4): 4 -> 1 (direct)


3. Core Theory

Operations

- find(x): returns representative (root) of element x's set
- union(x, y): merges sets containing x and y (no-op if already same set)

Implementation choices (two heuristics)

- Union by Rank/Size: attach smaller tree under root of larger tree to keep depth small
- Path Compression: during find, make visited nodes point directly to root

If both heuristics are used, amortized time per operation is nearly inverse-Ackermann function α(n), which is effectively constant for all practical n.

Complexity

- With union by rank + path compression: amortized O(α(n)) per operation (very close to O(1))


4. Java Implementation

Basic DSU with path compression and union by size

```java
class DSU {
    private int[] parent;
    private int[] size; // size[root] stores subtree size

    public DSU(int n) {
        parent = new int[n];
        size = new int[n];
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }
    }

    public int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }

    public boolean union(int x, int y) {
        int rx = find(x), ry = find(y);
        if (rx == ry) return false;
        // union by size: attach smaller to larger
        if (size[rx] < size[ry]) {
            parent[rx] = ry;
            size[ry] += size[rx];
        } else {
            parent[ry] = rx;
            size[rx] += size[ry];
        }
        return true;
    }
}
```

Explain every line

- `parent` array holds immediate parent; root nodes have parent[x] == x.
- `size` tracks subtree size for union-by-size heuristic.
- `find` compresses path by recursion and updating parent[x] to root.
- `union` finds roots and merges smaller into larger to keep depth small.


5. Python Implementation

```python
class DSU:
    def __init__(self, n):
        self.parent = list(range(n))
        self.size = [1]*n

    def find(self, x):
        while self.parent[x] != x:
            # path halving: point x to its grandparent
            self.parent[x] = self.parent[self.parent[x]]
            x = self.parent[x]
        return x

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry: return False
        if self.size[rx] < self.size[ry]: rx, ry = ry, rx
        self.parent[ry] = rx
        self.size[rx] += self.size[ry]
        return True
```

Explain: path halving reduces path length by pointing a node to its grandparent in the find loop (iterative alternative to full recursion compression).


6. Internal Working

What happens during operations

- Union finds roots and attaches one tree under another, changing only one parent pointer at the root.
- Path compression, during find, rewires several nodes to point directly to root, drastically reducing future find costs.

Why near-constant time

The inverse Ackermann function grows extremely slowly. For any conceivable n (n < 2^65536 or even astronomical ranges), α(n) ≤ 5. Thus DSU operations are effectively constant-time in practice.

Memory

- O(n) arrays for parent and size/rank.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| MakeSet (init) | O(n) |
| Find      | Amortized O(α(n)) |
| Union     | Amortized O(α(n)) |


8. Space Complexity

- O(n) for parent/size/rank arrays
- Auxiliary O(1) per operation


9. Common Interview Questions

Beginner

- Implement DSU and use it for connected components

Intermediate

- Use DSU in Kruskal to build MST
- Use DSU with rollback (persistent DSU) for offline dynamic connectivity

Advanced

- DSU with parity or bipartiteness checking (store extra metadata like parity bit)
- DSU on tree: compressing virtual tree or lowlink/bridge-finding techniques using DSU for offline queries


10. Common Mistakes

Mistake: forgetting path compression or union-by-rank leading to linear-time sequences
Why: naive union attaches arbitrarily and find walks long chains
Fix: always use at least one heuristic; preferably both

Mistake: using recursion for find without tail recursion protection in deep chains (stack overflow)


11. Real Interview Traps

Trap: assuming DSU supports split/delete of sets — DSU is not designed for deletions. Deletions require complex data structures or offline techniques.

Trap: forgetting to reinitialize DSU for multiple test cases or queries in problems — stateful arrays persist.


12. Real World Applications

- Kruskal's MST, dynamic connectivity, clustering algorithms, image processing


13. Common LeetCode Problems

Easy
- Number of Connected Components in an Undirected Graph

Medium
- Accounts Merge (use DSU on email identities)

Hard
- Offline dynamic connectivity problems or queries answerable with DSU rollback


14. Pattern Recognition

When to use DSU

- You need to maintain groups under merge operations and answer whether two items are connected
- Kruskal-like algorithms over edges sorted by weight

Decision checklist

- Are operations mostly unions and finds? → DSU
- Do you need deletes? → DSU not ideal; consider other approaches


15. Comparison Section

DSU vs BFS/DFS for connectivity

- DSU: incremental merges with near-constant query time; very efficient when unions are many and need repeated connectivity queries
- BFS/DFS: used when graph is static and you can run a traversal per query; expensive for many queries


16. 5 Minute Revision

DSU (Union-Find)

- Tracks disjoint sets with parent pointers
- Use union-by-size/rank and path compression for near-constant time (α(n))
- Ideal for Kruskal and dynamic connectivity when only unions occur


---
