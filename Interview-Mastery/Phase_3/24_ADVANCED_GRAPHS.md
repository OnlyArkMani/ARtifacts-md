# 24_ADVANCED_GRAPHS

1. Introduction

What this concept is

Advanced graph algorithms extend basic traversal and shortest-path techniques to solve problems such as flow, matching, connectivity under constraints, planarity, and dynamic graphs. They include maximum flow (Edmonds–Karp, Dinic), minimum cut, bipartite matching (Hopcroft–Karp), strongly connected components (Tarjan), bridges and articulation points, and advanced shortest-path variants.

Why it exists

Real-world networks and complex optimization problems require algorithms beyond BFS/DFS and Dijkstra. Advanced graph algorithms enable solving problems in routing, resource allocation, scheduling, and network reliability with provable guarantees.

What problem it solves

- Compute maximum throughput in networks (max flow)
- Find minimum cut separating two sets
- Find optimal matchings in bipartite graphs
- Analyze robustness (bridges, articulation points) and SCCs

Real-world analogy

Water flow through pipes (max flow), pairing drivers to riders (matching), analyzing critical roads whose failure splits a network (bridges).

Where it is used in industry

- Network design and traffic engineering
- Resource allocation and marketplaces
- Computational biology (matching problems)
- Compiler optimizations and program analysis (SCCs)


2. Intuition Section

Max flow (concept)

Think of each edge having a capacity. Flow pushes from source to sink without exceeding capacities. Augmenting paths increase flow until no admissible path remains.

Dinic idea

- Build level graph via BFS from source restricting edges with residual capacity
- Use blocking flow via DFS repeatedly; each BFS increases distance layering and limits augmentations per layer

Hopcroft–Karp for bipartite matching

- Use alternating BFS to find multiple shortest augmenting paths in parallel, speed-up over naive augmenting path method


3. Core Theory

Residual network

- For each edge u->v with capacity c and current flow f, residual capacity forward = c-f and backward = f. Augmenting paths exist in residual graph.

Max-flow min-cut theorem

- The value of the maximum flow equals the capacity of the minimum s-t cut.

Algorithms overview

- Edmonds–Karp: BFS to find shortest augmenting path each iteration; O(V*E^2) worst-case
- Dinic: BFS layering + multiple DFS to find blocking flow; O(E*sqrt(V)) for unit networks or O(V^2 * E) worst-case but practical fast
- Push–relabel (Goldberg–Tarjan): locally maintains preflows and pushes/excess; good for dense graphs
- Hopcroft–Karp: O(E*sqrt(V)) for bipartite maximum matching

SCCs and Tarjan

- Use DFS ordering to compute lowlink values and identify strongly connected components in O(V+E)

Bridges & articulation points

- Use DFS discovery times and lowlink to find edges/vertices that separate components when removed


4. Java Implementation (Dinic skeleton)

```java
// name=Interview-Mastery/Phase_3/24_ADVANCED_GRAPHS.md
static class Edge { int to; long cap; int rev; Edge(int t, long c, int r){to=t;cap=c;rev=r;} }

static class Dinic {
    int N;
    List<List<Edge>> g;
    int[] level, it;
    Dinic(int n){N=n; g = new ArrayList<>(); for (int i=0;i<n;i++) g.add(new ArrayList<>()); level=new int[n]; it=new int[n];}
    void addEdge(int u,int v,long c){ g.get(u).add(new Edge(v,c,g.get(v).size())); g.get(v).add(new Edge(u,0,g.get(u).size()-1)); }
    boolean bfs(int s,int t){ Arrays.fill(level, -1); Queue<Integer>q=new ArrayDeque<>(); level[s]=0; q.add(s); while(!q.isEmpty()){int u=q.poll(); for(Edge e:g.get(u)) if(e.cap>0 && level[e.to]<0){ level[e.to]=level[u]+1; q.add(e.to);} } return level[t]>=0; }
    long dfs(int u,int t,long f){ if(u==t) return f; for(int i=it[u]; i<g.get(u).size(); i++,it[u]++){ Edge e=g.get(u).get(i); if(e.cap>0 && level[e.to]==level[u]+1){ long ret=dfs(e.to,t,Math.min(f,e.cap)); if(ret>0){ e.cap-=ret; g.get(e.to).get(e.rev).cap += ret; return ret; } } } return 0; }
    long maxflow(int s,int t){ long flow=0; while(bfs(s,t)){ Arrays.fill(it,0); long f; while((f=dfs(s,t,Long.MAX_VALUE))>0) flow+=f; } return flow; }
}
```

Explain: residual edges, layering, and blocking flows. Practical notes: use long for capacities when sums may exceed int.


5. Python Implementation (Tarjan SCC and Hopcroft–Karp sketch)

Tarjan SCC (concise)

```python
def tarjan_scc(n, adj):
    sys.setrecursionlimit(1<<25)
    id = 0; ids=[-1]*n; low=[0]*n; onstack=[False]*n; stk=[]; sccs=[]
    def dfs(v):
        nonlocal id
        ids[v]=low[v]=id; id+=1; stk.append(v); onstack[v]=True
        for w in adj[v]:
            if ids[w]==-1:
                dfs(w); low[v]=min(low[v], low[w])
            elif onstack[w]:
                low[v]=min(low[v], ids[w])
        if low[v]==ids[v]:
            comp=[]
            while True:
                w=stk.pop(); onstack[w]=False; comp.append(w)
                if w==v: break
            sccs.append(comp)
    for i in range(n):
        if ids[i]==-1: dfs(i)
    return sccs
```

Hopcroft–Karp is longer; in interviews outline BFS layering over unmatched left nodes then DFS to find augmenting paths. Use adjacency lists and pair arrays.


6. Internal Working

Performance considerations

- Dinic suits sparse graphs common in CP; push-relabel can be faster on dense graphs in practice.
- Use adjacency vectors and avoid expensive object overhead in tight loops.
- For unit-capacity graphs, Dinic runs in O(min(V^{2/3}, sqrt(E)) E ) specialized bounds — but practical performance is often more relevant.

Numerical issues

- Capacity overflow: use 64-bit integer types when summing capacities; watch for sentinel values.


7. Time Complexity Table

| Algorithm | Complexity |
| --------- | ---------- |
| Edmonds–Karp | O(V * E^2) |
| Dinic | O(E * sqrt(V)) typical for unit graphs; O(V^2 * E) worst-case |
| Push–Relabel | O(V^3) worst, but practical fast|
| Hopcroft–Karp | O(E * sqrt(V)) |
| Tarjan SCC | O(V + E) |


8. Space Complexity

- Adjacency lists + residual edges ≈ O(E)
- Extra arrays for level, iterators, pairings O(V)


9. Common Interview Questions

- Implement Dinic or outline Edmonds–Karp for max-flow
- Find bridges/articulation points in a graph
- Compute SCCs and condense graph to DAG
- Bipartite matching (Hopcroft–Karp)


10. Common Mistakes

Mistake: forgetting to add reverse edge with zero capacity on addEdge — breaks residual graph
Fix: always add both forward and reverse entries

Mistake: not resetting iter pointers between BFS layers in Dinic


11. Real Interview Traps

Trap: trying to implement full push–relabel from scratch under time pressure — better to present Dinic or Edmonds–Karp and discuss tradeoffs

Trap: not considering multi-edge or parallel edges when modeling flows


12. Real World Applications

- Network traffic engineering, bipartite job matching, image segmentation via min-cut, computational biology matchings


13. Common LeetCode Problems

Medium/Hard
- Max Area of Island variants (graph traversal)
- Max Flow problems (rare on LC but common in CP)


14. Pattern Recognition

- Use flow when assignment/throughput constraints exist and can be modeled as capacities and demands
- Use SCC/bridge algorithms when analyzing connectivity robustness or component condensation


15. 5 Minute Revision

Advanced Graphs

- Residual networks, augmenting paths, blocking flow
- Dinic: BFS layering + DFS blocking flow
- Tarjan: lowlink and SCC detection
- Bridges/articulation via lowlink


---
