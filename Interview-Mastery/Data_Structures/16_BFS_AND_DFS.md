# 16_BFS_AND_DFS

1. Introduction

What this concept is

BFS (Breadth-First Search) and DFS (Depth-First Search) are fundamental graph traversal algorithms used to explore nodes and edges in a graph. BFS explores in layers (by distance from start) while DFS explores by diving deep into one branch before backtracking.

Why they exist

To systematically visit all nodes reachable from a source, enabling solutions to reachability, shortest paths in unweighted graphs, cycle detection, topological sorting, and component discovery.

What problem they solve

- BFS: finds shortest path in unweighted graphs, layer-order traversal
- DFS: useful for exploring connectivity, detecting cycles, generating topological orders, and solving backtracking/state-space search

Real-world analogy

- BFS: ripples spreading out from a stone dropped in water (wavefronts)
- DFS: maze solving by following a path until dead end, then backtracking

Where used

- Network protocols (shortest hop count)
- Puzzle solvers, compilers (dependency resolution), web crawlers


2. Intuition Section

ASCII BFS from node A

A
| \
B  C
|   |
D   E

BFS order: A, B, C, D, E (level by level)

DFS order (one possible): A, B, D, C, E (dive then backtrack)


3. Core Theory

BFS

- Uses queue
- Mark source visited then enqueue
- While queue not empty: dequeue v, examine neighbors u; if not visited mark and enqueue
- BFS tree records shortest path distances (number of edges)

DFS

- Uses stack (explicit) or recursion
- Visit node, mark visited, recursively visit each neighbor
- Useful for discovering back edges, computing discovery/finish times, and topological sort (postorder)

Properties

- BFS finds shortest path in edges from source in unweighted graphs
- DFS can detect cycles and produce topological order in DAGs


4. Java Implementation

BFS

```java
public List<Integer> bfs(Graph g, int s) {
    boolean[] vis = new boolean[g.n];
    List<Integer> order = new ArrayList<>();
    Queue<Integer> q = new ArrayDeque<>();
    q.add(s); vis[s] = true;
    while (!q.isEmpty()) {
        int v = q.poll(); order.add(v);
        for (int u : g.adj.get(v)) if (!vis[u]) { vis[u] = true; q.add(u); }
    }
    return order;
}
```

DFS recursive

```java
void dfs(int v, boolean[] vis, List<Integer> order) {
    vis[v] = true; order.add(v);
    for (int u : g.adj.get(v)) if (!vis[u]) dfs(u, vis, order);
}
```

Iterative DFS (stack)

```java
List<Integer> dfsIter(Graph g, int s) {
    boolean[] vis = new boolean[g.n];
    List<Integer> order = new ArrayList<>();
    Deque<Integer> st = new ArrayDeque<>();
    st.push(s);
    while (!st.isEmpty()) {
        int v = st.pop();
        if (vis[v]) continue;
        vis[v] = true; order.add(v);
        for (int u : g.adj.get(v)) if (!vis[u]) st.push(u);
    }
    return order;
}
```

Explain: iterative DFS may produce different order depending on neighbor iteration order.


5. Python Implementation

BFS

```python
from collections import deque

def bfs(adj, s):
    vis = set([s])
    q = deque([s])
    order = []
    while q:
        v = q.popleft(); order.append(v)
        for u in adj[v]:
            if u not in vis:
                vis.add(u); q.append(u)
    return order
```

DFS recursive

```python
def dfs(v, adj, vis, order):
    vis.add(v); order.append(v)
    for u in adj[v]:
        if u not in vis:
            dfs(u, adj, vis, order)
```


6. Internal Working

Why O(V+E)

- Each vertex is visited once; each adjacency list is traversed once across the traversal, so total work across edges is O(E).

Visited marking

- For BFS mark when enqueuing to avoid multiple enqueue; for DFS mark when pushing or entering to avoid redundant stack entries.

Space

- BFS queue holds at most O(width) nodes (could be O(V) in worst-case)
- DFS recursion stack depth O(V) worst-case; iterative stack similar


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| BFS       | O(V + E)   |
| DFS       | O(V + E)   |
| Shortest path (unweighted using BFS) | O(V + E) |


8. Space Complexity

- O(V) additional storage for visited array, order list, and queue/stack


9. Common Interview Questions

Beginner

- Implement BFS/DFS
- Level order traversal on trees (BFS)

Intermediate

- Shortest path in unweighted graph (BFS)
- Cycle detection using DFS

Advanced

- Iterative deepening DFS for memory-limited searches
- Tarjan's SCC (uses DFS-time and lowlink numbers)


10. Common Mistakes

Mistake: marking visited when dequeued vs enqueued in BFS; this can cause multiple enqueues and O(E) extra work
Correct: mark when enqueuing

Mistake: using recursion without guard for deep graphs leading to stack overflow
Fix: use iterative versions when V large or increase recursion limit carefully


11. Real Interview Traps

Trap: mixing BFS and Dijkstra semantics — BFS gives shortest path only in unweighted graphs

Trap: assuming DFS is faster than BFS; both are O(V+E); choice depends on problem needs


12. Real World Applications

- Pathfinding in games (BFS for grid movement), web crawling, AI search algorithms


13. Common LeetCode Problems

Easy
- Binary Tree Level Order Traversal (BFS)
- Flood Fill

Medium
- Course Schedule (use DFS for cycle detection / topological sort)

Hard
- Word Ladder II (BFS to build graph then backtrack to build paths)


14. Pattern Recognition

When to use BFS

- You need shortest path in number of edges
- You need level-order traversal

When to use DFS

- You need to explore entire component, detect cycles, or compute postorder/topological order


15. Comparison Section

BFS vs DFS

- BFS: queue, finds shortest paths in unweighted graphs, memory may be large due to frontier width
- DFS: recursion/stack, memory proportional to path depth, useful for topological sort and backtracking


16. 5 Minute Revision

BFS & DFS

- Both traverse O(V+E)
- BFS uses queue, good for shortest unweighted paths
- DFS uses recursion/stack, good for cycle detection and topological sort


---
