# 25_TREE_ALGORITHMS_AND_TRANSFORMATIONS

1. Introduction

What this concept is

Advanced tree algorithms include tree centroid decomposition, heavy-light decomposition (HLD), tree DP, lowest common ancestor (LCA) via binary lifting, Euler tours, and tree isomorphism checks. These techniques transform trees to support fast path queries, dynamic updates, and divide-and-conquer strategies.

Why it exists

Many problems on trees require path queries (sum, max) between nodes, updates on paths/subtrees, or repeated LCA queries. Naive traversal per query costs O(n) — decomposition and preprocessing reduce per-query costs to O(log n) or O(1) after O(n log n) preprocessing.

What problem it solves

- Efficient path/subtree queries and updates (HLD, segment trees on Euler tour)
- Repeated LCA queries in O(1) (RMQ over Euler tour) or O(log n) (binary lifting)
- Solve problems on tree metrics using divide-and-conquer (centroid decomposition)

Real-world analogy

Breaking a company organization (tree) into departments centering around managers (centroids) to process department-level queries efficiently.

Where it is used in industry

- Game engines and spatial partitioning, network analysis, and any hierarchical metric queries requiring fast repeated queries


2. Intuition Section

Euler tour + segment tree

- Flatten tree by recording entry and exit times — subtree of node corresponds to a contiguous range. Segment tree on this range supports subtree updates/queries.

Heavy-Light Decomposition (HLD)

- Split tree into heavy paths; represent each path as contiguous segment so a path query between u and v decomposes into O(log n) segments.

Centroid decomposition

- Recursively split tree at centroids to balance subproblems; used when queries involve distances to sets or path counts with multiplicative combinations.


3. Core Theory

Binary lifting for LCA

- Precompute up[v][k] = 2^k-th ancestor of v. Lifting to same depth and binary search ancestors to find LCA in O(log n).

Euler Tour & RMQ

- Do Euler tour storing first occurrence of each vertex and depth sequence; LCA reduces to RMQ on depths between occurrences; with sparse table RMQ returns O(1).

HLD decomposition

- For each node, designate child with largest subtree as heavy, others light. Each heavy chain converted to array; path query decomposed to O(log n) segments corresponding to chains.

Centroid decomposition

- Find centroid (removing it leaves subtrees ≤ n/2), recursively process subtrees and answer queries by combining information across centroid levels.


4. Java Implementation (Binary Lifting LCA skeleton)

```java
class LCA {
    int LOG; int n; int[] depth; int[][] up;
    List<List<Integer>> g;
    LCA(int n){ this.n=n; LOG = 32 - Integer.numberOfLeadingZeros(n); up = new int[LOG][n]; depth = new int[n]; g = new ArrayList<>(); for(int i=0;i<n;i++) g.add(new ArrayList<>());}    
    void dfs(int v,int p){ up[0][v]=p<0?v:p; for(int u:g.get(v)) if(u!=p){ depth[u]=depth[v]+1; dfs(u,v);} }
    void build(int root){ dfs(root,-1); for(int k=1;k<LOG;k++) for(int v=0; v<n; v++) up[k][v] = up[k-1][ up[k-1][v] ]; }
    int lca(int a,int b){ if(depth[a]<depth[b]) {int t=a;a=b;b=t;} int diff = depth[a]-depth[b]; for(int k=0;k<LOG;k++) if(((diff>>k)&1)==1) a = up[k][a]; if(a==b) return a; for(int k=LOG-1;k>=0;k--) if(up[k][a]!=up[k][b]) { a = up[k][a]; b = up[k][b]; } return up[0][a]; }
}
```

Explain: preprocessing O(n log n), queries O(log n); edge cases when root ancestor refers to itself handled.


5. Python Implementation (Euler tour + RMQ outline)

- Euler tour produce arrays euler[], depth[], firstOcc[]; build sparse table on depth; LCA(a,b) = argmin depth in euler[first[a] .. first[b]]

Concise code omitted for brevity but describe steps and complexity.


6. Internal Working

Why decomposition helps

- Transform tree path problems into array segment queries enabling segment tree or Fenwick usage
- Avoid repeated traversal by precomputing structural delegation

Memory and time tradeoffs

- HLD and Euler tours need O(n) extra arrays; binary lifting needs O(n log n) memory for ancestor table


7. Time Complexity Table

| Technique | Preprocess | Query |
| --------- | ---------- | ----- |
| Binary lifting LCA | O(n log n) | O(log n) |
| Euler tour + RMQ | O(n) + O(n log n) for sparse table | O(1) |
| HLD path query | O(n) preprocess | O(log^2 n) worst (or O(log n) with balanced paths) |
| Centroid decomposition | O(n log n) | O(log n) per query depending on problem |


8. Space Complexity

- Ancestor tables O(n log n)
- Euler arrays O(n)
- HLD arrays O(n)


9. Common Interview Questions

- Compute LCA in O(1) or O(log n)
- Use HLD to support path updates and queries
- Count pairs within distance K using centroid decomposition


10. Common Mistakes

Mistake: incorrect handling of parent of root in binary lifting causing infinite loops
Fix: set up[0][root]=root and guard while lifting

Mistake: forgetting to reset visited arrays in centroid recursion


11. Real Interview Traps

Trap: choosing HLD when Euler+RMQ is simpler and sufficient (tradeoffs depend on update needs)

Trap: expecting O(1) LCA without RMQ/sparse table setup


12. Real World Applications

- Fast network queries over hierarchical topologies
- Game tree range queries (path updates for buffs/effects in game engines)


13. Common LeetCode Problems

- Lowest Common Ancestor of a Binary Tree (simple) vs LCA in multiple queries (use binary lifting)
- Tree path sum queries (HLD typical in CP)


14. Pattern Recognition

- If repeated path queries with updates → HLD
- If static tree and many LCA queries → Euler tour + RMQ


15. 5 Minute Revision

- Binary lifting O(log n), Euler RMQ O(1) after preprocess, HLD transforms path to O(log n) segments, centroid decomposes tree for divide-and-conquer


---
