# 11 — Graphs (BFS, DFS, Union-Find, Topological Sort, Dijkstra)

## 1. Pattern Overview

A graph is nodes + edges; everything else is bookkeeping. Five algorithms cover essentially all interview graph problems — the skill is **picking the right one**, which follows almost mechanically from the question asked:

| Question the problem asks | Algorithm | Complexity |
|---|---|---|
| "Is X reachable / explore everything / count components" | **DFS** | O(V + E) |
| "**Shortest** path, edges unweighted / fewest steps" | **BFS** | O(V + E) |
| "Are these connected? (many dynamic queries) / merge groups" | **Union-Find** | ~O(α) per op |
| "Order tasks with prerequisites / detect cycle in a DAG" | **Topological sort** | O(V + E) |
| "Shortest path with **weighted** (non-negative) edges" | **Dijkstra** | O(E log V) |

Universal rule #1: **track visited** — graphs have cycles; tree code without a visited set loops forever.
Universal rule #2: grids ARE graphs — cell = node, 4-neighbors = edges. Half of all "graph" interview questions are grids.

**Representation:** adjacency list `Map<node, List<neighbor>>` (or `List<List<Integer>>`). Matrix only when the graph is dense or given that way.

## 2. Recognition Triggers

- Explicit: "nodes/edges", "cities/roads", "courses/prerequisites", "accounts/friendships".
- Grid + "islands / regions / flood fill / spread" → grid DFS/BFS.
- **"Minimum number of steps/moves"** (word ladders, knight moves, rotten oranges, locks) → **BFS** — unweighted shortest path, no exceptions.
- "Spreads simultaneously from multiple sources" (rot, fire, walls-and-gates) → **multi-source BFS** (seed queue with all sources).
- "Prerequisites / build order / must come before" → **topological sort**; "can you finish?" = cycle detection.
- "Number of connected components", "accounts merge", "redundant connection", queries of "are A and B connected?" arriving over time → **Union-Find**.
- "Cheapest/fastest path with weights" (network delay, flight costs) → **Dijkstra** (non-negative); K-stops constraint → Bellman-Ford / BFS-on-layers.
- "Minimum cost to connect all points" → MST (Prim = Dijkstra-like with heap; Kruskal = Union-Find + sorted edges).
- Implicit graphs: states as nodes (word → word by one letter change; lock combos) — if the problem has "states and transitions," build the graph mentally and BFS it.
- Constraint smell: V, E ≤ 10^5 → linear-ish O(V + E) expected; weighted + 10^4 nodes → O(E log V) Dijkstra.

## 3. Template Code

### DFS — components (Python)
```python
def count_components(n: int, edges: list[list[int]]) -> int:
    adj = [[] for _ in range(n)]
    for a, b in edges:
        adj[a].append(b)
        adj[b].append(a)

    seen = set()
    def dfs(u: int) -> None:
        for v in adj[u]:
            if v not in seen:
                seen.add(v)
                dfs(v)

    count = 0
    for u in range(n):
        if u not in seen:          # unseen node = a brand-new component
            seen.add(u)
            dfs(u)
            count += 1
    return count
```

### BFS — shortest steps on a grid (Java)
```java
// BFS visits nodes in increasing distance order — that's WHY it finds shortest paths.
// Mark visited when ENQUEUEING (not dequeuing) or nodes enter the queue twice.
public int shortestPath(int[][] grid) {
    int n = grid.length, m = grid[0].length;
    int[][] DIRS = {{1,0},{-1,0},{0,1},{0,-1}};
    Queue<int[]> q = new ArrayDeque<>();
    boolean[][] seen = new boolean[n][m];
    q.offer(new int[]{0, 0});
    seen[0][0] = true;
    int steps = 0;
    while (!q.isEmpty()) {
        for (int sz = q.size(); sz > 0; sz--) {   // layer by layer = distance by distance
            int[] cur = q.poll();
            if (cur[0] == n - 1 && cur[1] == m - 1) return steps;
            for (int[] d : DIRS) {
                int r = cur[0] + d[0], c = cur[1] + d[1];
                if (r >= 0 && r < n && c >= 0 && c < m && !seen[r][c] && grid[r][c] == 0) {
                    seen[r][c] = true;            // mark NOW, at enqueue time
                    q.offer(new int[]{r, c});
                }
            }
        }
        steps++;
    }
    return -1;
}
```

### Union-Find with path compression + union by size (Python)
```python
class DSU:
    def __init__(self, n: int):
        self.parent = list(range(n))   # each node starts as its own root
        self.size = [1] * n

    def find(self, x: int) -> int:
        while self.parent[x] != x:
            self.parent[x] = self.parent[self.parent[x]]  # path halving: flatten as we go
            x = self.parent[x]
        return x

    def union(self, a: int, b: int) -> bool:
        ra, rb = self.find(a), self.find(b)
        if ra == rb:
            return False               # already connected — useful cycle signal!
        if self.size[ra] < self.size[rb]:
            ra, rb = rb, ra            # attach smaller tree under larger → stays shallow
        self.parent[rb] = ra
        self.size[ra] += self.size[rb]
        return True
```

### Topological sort — Kahn's BFS (Java)
```java
// Repeatedly peel off nodes with no remaining prerequisites.
// If we can't peel everything → a cycle is holding the leftovers hostage.
public int[] topoSort(int n, int[][] edges) {   // edge [a,b]: a must come before b
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[n];
    for (int[] e : edges) { adj.get(e[0]).add(e[1]); indegree[e[1]]++; }

    Queue<Integer> q = new ArrayDeque<>();
    for (int i = 0; i < n; i++) if (indegree[i] == 0) q.offer(i);  // no prereqs → ready

    int[] order = new int[n];
    int idx = 0;
    while (!q.isEmpty()) {
        int u = q.poll();
        order[idx++] = u;
        for (int v : adj.get(u))
            if (--indegree[v] == 0) q.offer(v);   // last prereq done → v is ready
    }
    return idx == n ? order : new int[0];         // idx < n → cycle
}
```

### Dijkstra (Python)
```python
import heapq

def dijkstra(n: int, adj: dict[int, list[tuple[int, int]]], src: int) -> list[float]:
    # adj[u] = [(v, weight), ...]. Non-negative weights ONLY.
    dist = [float("inf")] * n
    dist[src] = 0
    heap = [(0, src)]                     # (distance, node)
    while heap:
        d, u = heapq.heappop(heap)
        if d > dist[u]:                   # stale entry (we found better already) → skip
            continue
        for v, w in adj[u]:
            nd = d + w
            if nd < dist[v]:              # relaxation: found a better route to v
                dist[v] = nd
                heapq.heappush(heap, (nd, v))   # lazy: push duplicate, skip stale later
    return dist
```

## 4. Language-Specific Gotchas

- **Python recursion limit:** grid DFS on a 300×300 grid can exceed 1000 frames → `sys.setrecursionlimit(10**6)` or convert to an explicit stack / BFS.
- **Java:** no default-dict — `computeIfAbsent(u, k -> new ArrayList<>())` is the adjacency-building idiom. Python: `defaultdict(list)`.
- **Java PriorityQueue has no decrease-key** → lazy deletion (push duplicates, skip stale) is standard in both languages; Python's `heapq` same.
- Visited timing: BFS marks at **enqueue**; Dijkstra "marks" via the `d > dist[u]` staleness check at **pop**. Mixing these up breaks correctness or blows up the queue.
- Grid directions array: define once (`DIRS`), don't hand-write 4 near-identical blocks.
- Python `deque` for BFS (never `list.pop(0)`); Java `ArrayDeque` (never `LinkedList` for perf, though it works).
- Union-Find in Java: plain `int[] parent` arrays — fastest and cleanest; resist building a class if a couple arrays suffice under time pressure.
- Node keys: Python dicts take tuples `(r, c)` directly; Java needs encoding (`r * cols + c`) or a nested array — the int-encoding is less error-prone than `Objects.hash`.

## 5. Worked Examples

### Easy — Number of Islands (LC 200)

**Problem:** Grid of `'1'` (land) / `'0'` (water). Count islands (4-directionally connected land).

**Recognition trace:** grid + "connected regions" → flood fill. Each unvisited land cell starts a new island; DFS sinks the whole island so it's never counted again.

**Java:**
```java
public int numIslands(char[][] grid) {
    int count = 0;
    for (int r = 0; r < grid.length; r++)
        for (int c = 0; c < grid[0].length; c++)
            if (grid[r][c] == '1') {
                count++;              // new island discovered
                sink(grid, r, c);     // erase it entirely
            }
    return count;
}
private void sink(char[][] g, int r, int c) {
    if (r < 0 || r >= g.length || c < 0 || c >= g[0].length || g[r][c] != '1') return;
    g[r][c] = '0';                    // mark visited by mutating the grid
    sink(g, r + 1, c); sink(g, r - 1, c); sink(g, r, c + 1); sink(g, r, c - 1);
}
```

**Python:**
```python
def num_islands(grid: list[list[str]]) -> int:
    rows, cols = len(grid), len(grid[0])

    def sink(r: int, c: int) -> None:
        if not (0 <= r < rows and 0 <= c < cols) or grid[r][c] != "1":
            return
        grid[r][c] = "0"              # visited = sunk
        sink(r + 1, c); sink(r - 1, c); sink(r, c + 1); sink(r, c - 1)

    count = 0
    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == "1":
                count += 1
                sink(r, c)
    return count
```

**Trace:**
```
1 1 0 0        scan hits (0,0): island #1, sink → 0 0 0 0
1 0 0 1                                          0 0 0 1
0 0 1 1        scan hits (1,3): island #2, sink → ...(2,2),(2,3) connected → sunk
final count = 2
```

### Medium — Course Schedule (LC 207)

**Problem:** `numCourses` and prerequisite pairs `[a, b]` (take b before a). Can all courses be finished?

**Recognition trace:** "prerequisites" → dependency DAG → finishable ⟺ **no cycle** → topological sort; if the peeled count < n, a cycle exists.

**Java:** the topo template above, returning `idx == n`.

**Python:**
```python
from collections import deque

def can_finish(num_courses: int, prerequisites: list[list[int]]) -> bool:
    adj = [[] for _ in range(num_courses)]
    indegree = [0] * num_courses
    for a, b in prerequisites:        # b → a  (b unlocks a)
        adj[b].append(a)
        indegree[a] += 1

    q = deque(i for i in range(num_courses) if indegree[i] == 0)
    taken = 0
    while q:
        u = q.popleft()
        taken += 1
        for v in adj[u]:
            indegree[v] -= 1
            if indegree[v] == 0:      # all prereqs of v now done
                q.append(v)
    return taken == num_courses       # leftover courses = trapped in a cycle
```

**Trace** for `n = 4`, prereqs `[[1,0],[2,0],[3,1],[3,2]]` (diamond):
```
0 → 1 → 3          indegree: 0:0  1:1  2:1  3:2
 \→ 2 ↗
queue [0] → take 0 → 1,2 drop to 0 → queue [1,2]
take 1 → 3 drops to 1 | take 2 → 3 drops to 0 → queue [3]
take 3 → taken = 4 = n → True
(add edge 3→0 and node 0 never re-enters: taken stalls at 0 < 4 → False)
```

### Hard — Word Ladder (LC 127)

**Problem:** Transform `beginWord` → `endWord`, one letter per step, every intermediate in `wordList`. Return the length of the shortest chain (0 if impossible).

**Recognition trace:** "shortest transformation sequence" → **shortest path** → BFS. The graph is implicit: words = nodes, edge = one-letter difference. Naive edge-building is O(n² · L); the wildcard-bucket trick (`h*t` matches hot/hit/hat) makes neighbor lookup O(L · 26) or O(L) with buckets.

**Java:**
```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return 0;
    Queue<String> q = new ArrayDeque<>();
    q.offer(beginWord);
    Set<String> seen = new HashSet<>();
    seen.add(beginWord);
    int steps = 1;                                   // counts words, incl. beginWord
    while (!q.isEmpty()) {
        for (int sz = q.size(); sz > 0; sz--) {
            String w = q.poll();
            if (w.equals(endWord)) return steps;
            char[] cs = w.toCharArray();
            for (int i = 0; i < cs.length; i++) {    // try all one-letter mutations
                char orig = cs[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    if (c == orig) continue;
                    cs[i] = c;
                    String next = new String(cs);
                    if (dict.contains(next) && seen.add(next))  // add() returns false if present
                        q.offer(next);
                }
                cs[i] = orig;                        // restore before next position
            }
        }
        steps++;
    }
    return 0;
}
```

**Python:**
```python
from collections import deque
import string

def ladder_length(begin: str, end: str, word_list: list[str]) -> int:
    dict_ = set(word_list)
    if end not in dict_:
        return 0
    q = deque([begin])
    seen = {begin}
    steps = 1
    while q:
        for _ in range(len(q)):                  # one BFS layer = one mutation step
            w = q.popleft()
            if w == end:
                return steps
            for i in range(len(w)):
                for c in string.ascii_lowercase: # generate all 1-letter neighbors
                    nxt = w[:i] + c + w[i + 1:]
                    if nxt in dict_ and nxt not in seen:
                        seen.add(nxt)            # mark at enqueue
                        q.append(nxt)
        steps += 1
    return 0
```

**Trace** for `hit → cog`, dict `[hot, dot, dog, lot, log, cog]`:
```
layer 1: hit
layer 2: hot                    (h*t: only hot)
layer 3: dot, lot
layer 4: dog, log
layer 5: cog  ← reached → 5
      hit ─ hot ─ dot ─ dog ─ cog
                └ lot ─ log ──┘        BFS guarantees first arrival = shortest
```

## 6. Common Mistakes & Edge Cases

- No visited set → infinite loop. Marking visited at dequeue instead of enqueue → exponential queue blowup (still "correct," but TLEs).
- Directed vs undirected: adding edges one-way when the problem is mutual (or both ways when it isn't).
- Topological sort: reversing edge direction (`[a,b]` — which one is the prerequisite? Read twice).
- Dijkstra with negative weights → silently wrong (use Bellman-Ford); forgetting the stale-entry skip → TLE.
- Union-Find without path compression → O(n) finds → TLE on 10^5 queries.
- Grid: row/col swapped in bounds checks; diagonal neighbors included/excluded wrongly.
- Multi-source BFS: initializing distances per-source and running BFS n times (O(n·V)) instead of seeding all sources at once (O(V)).
- Disconnected graphs: forgetting the outer loop over all nodes (components, topo).
- Word Ladder: counting edges vs nodes (off-by-one on `steps`).
- Self-loops and parallel edges breaking naive cycle detection.

## 7. Fallback Map

- Weighted shortest path with **negative** edges → Bellman-Ford; "at most K stops" → BFS by layers / Bellman-Ford with K rounds.
- All-pairs shortest paths, small V (≤ 400) → Floyd-Warshall (three nested loops).
- "Minimum cost to connect everything" → MST: Kruskal (**Union-Find** + sort) or Prim (**heap**).
- Shortest path where "cost" = maximize the minimum edge / minimize the maximum → **binary search on answer** (`05`) + BFS/DFS feasibility, or modified Dijkstra.
- Counting paths (not finding them) in a DAG → **DP over topological order** (`12`).
- Grid enumeration of all paths (not shortest) → **Backtracking** (`10`).
- Strongly connected components, bridges, articulation points → Tarjan's — know they exist; rarely required to code.
