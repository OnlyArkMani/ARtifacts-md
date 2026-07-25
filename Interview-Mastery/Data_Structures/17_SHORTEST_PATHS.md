# 17_SHORTEST_PATHS

1. Introduction

What this concept is

Shortest path algorithms find the minimum-cost path between nodes in a graph given edge weights (or unweighted where cost=1). Depending on edge weight properties (non-negative, negative allowed, DAG), different algorithms apply: BFS for unweighted, Dijkstra for non-negative weights, Bellman-Ford for graphs with negative edges, and Floyd–Warshall/Johnson for all-pairs shortest paths.

Why it exists

Many practical problems involve finding optimal routes, minimum-cost communications, or least-resistance flows. Shortest path algorithms provide efficient techniques to solve these problems depending on constraints.

What problem it solves

Gives the minimal total weight path between nodes respecting edge costs.

Real-world analogy

GPS routing: find the fastest/shortest route between two locations when roads have travel times (weights).

Where used

- Navigation systems
- Network routing protocols (OSPF uses Dijkstra-like approaches)
- Resource scheduling and logistics


2. Intuition Section

Unweighted graph: BFS

- Expand equally outward from source; first time you reach target is via shortest edge-count path.

Weighted non-negative: Dijkstra

- Greedy: always finalize the unvisited node with smallest tentative distance because any alternative via other unfinalized nodes won't be shorter (proof by contradiction using non-negativity).

Negative edges: Bellman-Ford

- Relax all edges up to V-1 times; detect negative cycles if further relaxation possible.


3. Core Theory

Dijkstra's algorithm (single-source, non-negative weights)

- Initialize dist[source]=0, others = inf
- Use min-priority queue keyed by dist
- Pop node u with smallest dist, relax outgoing edges (if dist[u] + w(u,v) < dist[v], update dist[v] and push/update in pq)
- Once popped, node's dist is finalized

Complexity: O((V + E) log V) with binary heap; O(E + V log V) with Fibonacci heap (theoretical)

Bellman-Ford

- Repeat relax all edges V-1 times
- O(V*E) time
- Detect negative cycles by checking one more relaxation step

Floyd–Warshall (all-pairs)

- Dynamic programming over intermediate nodes k
- O(V^3) time, simple implementation for dense graphs or small V

Johnson's algorithm

- Reweight edges using Bellman-Ford then run Dijkstra from each node — good for sparse graphs for all-pairs


4. Java Implementation

Dijkstra (adj list with priority queue)

```java
long[] dijkstra(int src, List<List<Pair>> adj, int n) {
    long[] dist = new long[n]; Arrays.fill(dist, Long.MAX_VALUE);
    dist[src] = 0;
    PriorityQueue<Pair> pq = new PriorityQueue<>(Comparator.comparingLong(p -> p.dist));
    pq.add(new Pair(src, 0));
    while (!pq.isEmpty()) {
        Pair p = pq.poll();
        int u = p.node; long d = p.dist;
        if (d != dist[u]) continue; // stale entry
        for (Pair e : adj.get(u)) {
            int v = e.node; long w = e.dist;
            if (dist[v] > d + w) { dist[v] = d + w; pq.add(new Pair(v, dist[v])); }
        }
    }
    return dist;
}
```

Explain: PQ may contain stale entries, so check against current dist before relaxing.

Bellman-Ford (outline)

- Loop V-1 times, for each edge relax
- Check negative cycle if any edge can be relaxed afterwards


5. Python Implementation

Use heapq for Dijkstra; store (dist, node) tuples. For graphs with many edges, consider adjacency lists for memory.


6. Internal Working

Why Dijkstra needs non-negative weights

Because once a node is finalized (popped from PQ), no future path through an unvisited node can produce a shorter path; negative edge could break this property as it may later reduce distances.

Why Bellman-Ford works with negatives

It systematically propagates improvements through edges; V-1 rounds ensure any shortest simple path (≤ V-1 edges) is discovered. Additional relaxation indicates a negative cycle.


7. Time Complexity Table

| Operation / Algorithm | Complexity |
| --------------------- | ---------- |
| BFS (unweighted)      | O(V + E)   |
| Dijkstra (binary heap)| O((V + E) log V) |
| Bellman-Ford          | O(V * E)   |
| Floyd–Warshall         | O(V^3)     |


8. Space Complexity

- Dijkstra/Bellman-Ford: O(V + E) for adjacency lists and O(V) for dist arrays and PQ
- Floyd–Warshall: O(V^2) for dist matrix


9. Common Interview Questions

Beginner

- Implement BFS shortest path on unweighted graph
- Implement Dijkstra on small graphs

Intermediate

- Detect negative cycle using Bellman-Ford
- Reconstruct shortest path from parent pointers

Advanced

- All-pairs shortest path with Floyd–Warshall or Johnson depending on graph density
- K shortest paths variations


10. Common Mistakes

Mistake: not handling stale entries in PQ (push-only implementation) — always check popped distance against current best

Mistake: using Dijkstra on graph with negative weights — produce incorrect results


11. Real Interview Traps

Trap: forgetting to reconstruct path (use parent predecessor array updated during relaxation)

Trap: using adjacency matrix with Dijkstra for sparse graphs causing poor performance


12. Real World Applications

- GPS navigation
- Network routing protocols
- Shortest-cost planning in logistics


13. Common LeetCode Problems

Easy
- Network Delay Time (Dijkstra)

Medium
- Cheapest Flights Within K Stops (modified Dijkstra / BFS in layered graph)

Hard
- K shortest paths, variations requiring advanced algorithms


14. Pattern Recognition

Decision framework

- If unweighted → BFS
- If non-negative weights → Dijkstra
- If negative weights → Bellman-Ford (and detect cycles)
- All-pairs: Floyd–Warshall for small dense graphs; Johnson for sparse


15. Comparison Section

Dijkstra vs Bellman-Ford

- Dijkstra: faster, requires non-negative edges, good for large graphs
- Bellman-Ford: supports negatives, detects negative cycles, slower

BFS vs Dijkstra

- BFS is special case of Dijkstra when all edge weights equal


16. 5 Minute Revision

Shortest Paths

- BFS: unweighted graphs, O(V+E)
- Dijkstra: non-negative weights, PQ, O((V+E) log V)
- Bellman-Ford: negative edges allowed, O(V*E), detects negative cycles
- Floyd–Warshall: all-pairs, O(V^3)


---
