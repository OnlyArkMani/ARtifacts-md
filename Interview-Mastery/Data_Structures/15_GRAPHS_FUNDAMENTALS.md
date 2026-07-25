# 15_GRAPHS_FUNDAMENTALS

1. Introduction

What this concept is

A Graph is an abstract data structure that models relationships between objects. It consists of vertices (nodes) and edges (connections). Graphs can be directed or undirected, weighted or unweighted, and may include cycles or be acyclic.

Why it exists

Graphs model any pairwise relationships: social networks, road maps, web links, dependency graphs, etc. They are essential for representing complex networks where relationships, not hierarchy, matter.

What problem it solves

Graphs provide a unified way to model connectivity, paths, flows, cycles, reachability, and ordering under dependencies.

Real-world analogy

- Cities (vertices) connected by roads (edges)
- People (vertices) connected by friendships (edges)

Where it is used in industry

- Social networks, recommendation systems
- Maps and GPS routing
- Dependency resolution in build tools and package managers
- Knowledge graphs and semantic web


2. Intuition Section

ASCII: small undirected unweighted graph

A -- B
| \  |
C -- D

Edges: (A,B),(A,C),(A,D),(B,D),(C,D)

Think of walking the graph — BFS explores closer nodes first, DFS dives deep along a branch.


3. Core Theory

Basic definitions

- Vertex (V): node
- Edge (E): connection between two vertices
- Degree: number of edges incident to a vertex (out-degree and in-degree for directed graphs)
- Path: sequence of edges connecting vertices
- Cycle: path that begins and ends at same vertex
- Connected component: maximal set of vertices where each pair is mutually reachable (in undirected graphs)

Graph representations

- Adjacency list: for each vertex, list of neighbors — efficient for sparse graphs (O(V + E) memory)
- Adjacency matrix: VxV matrix where cell indicates presence/weight of edge — O(V^2) memory, fast edge checks
- Edge list: list of edges — simple and compact for some algorithms

Weighted vs Unweighted

- Weighted edges have costs; algorithms like Dijkstra operate on non-negative weights, Bellman-Ford handles negatives.

Directed vs Undirected

- Directed edges have orientation and affect reachability and cycle detection (e.g., for topological sort)

Common graph problems

- Traversal (BFS/DFS)
- Shortest path (single-source, all-pairs)
- Minimum spanning tree (MST)
- Strongly connected components (SCC)
- Topological sorting


4. Java Implementation

Adjacency list using lists

```java
class Graph {
    int n;
    List<List<Integer>> adj;
    Graph(int n) {
        this.n = n;
        adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    }
    void addEdge(int u, int v) { adj.get(u).add(v); }
}
```

Explain: zero-based nodes, adjacency list is efficient for sparse graphs.

Edge weights

- Use List<List<Pair<Integer,Integer>>> or separate arrays for weights

Representation tradeoffs

- Use adjacency matrix when n ≤ few thousands and you need constant-time edge checks or matrix operations; adjacency list for large sparse graphs.


5. Python Implementation

Adjacency list using dict/list

```python
from collections import defaultdict
class Graph:
    def __init__(self):
        self.adj = defaultdict(list)
    def add_edge(self, u, v):
        self.adj[u].append(v)
```

Use dict for arbitrary labeled vertices. For weighted edges store tuples (v, w).


6. Internal Working

Why BFS and DFS visit all edges

- BFS/DFS complexity O(V+E) because each vertex is discovered once and each edge examined at most once in adjacency list representation.

Memory locality

- Adjacency matrix uses contiguous memory but wastes space in sparse graphs; adjacency list uses more pointers/references but is compact for sparse graphs.

Graph traversals and recursion

- DFS recursion depth can reach O(V) - watch for stack limits; iterative stack-based DFS mitigates this.


7. Time Complexity Table

| Operation | Complexity (Adj List) |
| --------- | --------------------- |
| Add Edge  | O(1)                  |
| Remove Edge | O(deg(v))           |
| Has Edge  | O(deg(v))             |
| Traverse (BFS/DFS) | O(V + E)     |

Why: adjacency list stores neighbors; traversals visit each vertex and each edge once.


8. Space Complexity

- Adjacency list: O(V + E)
- Adjacency matrix: O(V^2)


9. Common Interview Questions

Beginner

- Implement BFS/DFS
- Check if a graph is bipartite

Intermediate

- Detect cycle in directed graph
- Topological sort and build ordering

Advanced

- Implement SCCs (Kosaraju/Tarjan)
- Minimum cut / max flow (Edmonds-Karp, Dinic)


10. Common Mistakes

Mistake: not marking visited nodes and causing infinite loops in cyclic graphs
Fix: always mark visited upon enqueue/discovery for BFS, upon push/visit for DFS

Mistake: choosing adjacency matrix for large sparse graphs leading to memory blowup


11. Real Interview Traps

Trap: confusion between directed and undirected cycle detection algorithms — they differ in approach (undirected uses parent check, directed uses recursion stack)

Trap: mis-using global visited arrays across calls, leading to incorrect results for multi-component graphs


12. Real World Applications

- Social graph analysis, recommendations
- Routing and navigation (graphs with weights)
- Dependency resolution and build systems


13. Common LeetCode Problems

Easy
- Number of Islands (graph via grid BFS)
- Flood Fill

Medium
- Course Schedule (detect cycle / topological sort)
- Clone Graph

Hard
- Word Ladder II (BFS + backtracking to reconstruct paths)


14. Pattern Recognition

When to model as graph

- Problem states relationships/pairs, connectivity, reachability, or transitions between states
- Grid problems (convert cells to nodes)

Decision framework

- Use BFS for shortest path in unweighted graphs
- Use Dijkstra for non-negative weighted shortest path
- Use DP/state graph for puzzle/board problems with moderate state spaces


15. Comparison Section

Graph vs Tree

- Tree is a special graph (connected, acyclic). Graphs allow cycles and multiple components.

Adjacency List vs Matrix

- Use list for sparse graphs, matrix for dense graphs or simple constant-time edge checks.


16. 5 Minute Revision

Graphs

- Model: vertices + edges, directed/undirected, weighted/unweighted
- Representations: adjacency list (O(V+E)), adjacency matrix (O(V^2))
- Traversals: BFS/DFS O(V+E)
- Shortest path: BFS (unweighted), Dijkstra (non-negative weights), Bellman-Ford (negative weights), Floyd-Warshall (all pairs)


---
