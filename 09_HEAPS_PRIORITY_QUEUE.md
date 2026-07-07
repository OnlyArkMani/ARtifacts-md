# 09 — Heaps / Priority Queue

## 1. Pattern Overview

A heap answers one question repeatedly and fast: **"what's the current min (or max)?"** — O(1) to peek, O(log n) to insert or remove, no full sorting ever performed.

Mental model: a partially-sorted binary tree (stored as an array) where each parent ≤ its children (min-heap). It's *lazy sorting*: you pay log n per operation instead of n log n upfront, which wins whenever you only need the extremes, the data arrives as a stream, or you'd re-sort repeatedly.

The three killer use-cases:

- **Top-K:** keep a heap of size K of the *opposite* polarity (min-heap for top-K largest — the root is the "weakest member," evicted when someone better arrives). O(n log k) beats sorting's O(n log n) and keeps memory at K for streams.
- **K-way merge:** heap holds one candidate per source; pop the best, push its successor. (Merge K lists, kth smallest in sorted matrix.)
- **Scheduling/simulation by priority:** always process the currently-most-urgent item next (Dijkstra `11`, task scheduling, meeting rooms).

**Complexity:** push/pop O(log n), peek O(1), build-heap from array O(n) (not n log n — worth saying). Top-K: O(n log k).

## 2. Recognition Triggers

- **"K"** in the problem: "kth largest", "top K frequent", "K closest points" → heap of size K.
- "Stream" / "data arriving one at a time" + order statistics → heap (you can't sort a stream).
- "Median from a stream" → **two heaps** (max-heap lower half, min-heap upper half).
- "Merge K sorted X" → K-way merge heap.
- "Most frequent / highest priority next", CPU/task scheduling, "cooldown" → max-heap + possibly a queue for cooldowns.
- Repeatedly need min AND the data changes between queries → heap beats re-sorting.
- "Minimum cost to connect/combine" (sticks, ropes, Huffman-style) → greedily combine two smallest → min-heap.
- Constraint smell: n = 10^5, k small; O(n log k) or O(n log n) fine → heap. If K = few and n huge, heap-of-K is the memory-safe answer interviewers want.
- **Anti-trigger:** need *arbitrary* lookups/deletions or "kth" only once with mutation allowed → quickselect (average O(n)) may beat it; sorted static data → binary search.

## 3. Template Code

### Top-K largest via min-heap (Java)
```java
// WHY min-heap for "largest": the root is the SMALLEST of the current top-K —
// the gatekeeper. Any newcomer must beat the gatekeeper to enter.
public int[] topK(int[] nums, int k) {
    PriorityQueue<Integer> heap = new PriorityQueue<>();   // min-heap by default
    for (int x : nums) {
        heap.offer(x);
        if (heap.size() > k) heap.poll();   // evict the weakest — O(log k)
    }
    int[] res = new int[k];
    for (int i = k - 1; i >= 0; i--) res[i] = heap.poll(); // ascending pops
    return res;
}
```

### Top-K largest via min-heap (Python)
```python
import heapq

def top_k(nums: list[int], k: int) -> list[int]:
    heap = []                          # heapq is ALWAYS a min-heap
    for x in nums:
        heapq.heappush(heap, x)
        if len(heap) > k:
            heapq.heappop(heap)        # smallest of the top-k leaves first
    return sorted(heap, reverse=True)
    # one-liner alternative: heapq.nlargest(k, nums)
```

### K-way merge (Python)
```python
import heapq

def merge_k_sorted(lists: list[list[int]]) -> list[int]:
    # Entry: (value, source_index, position) — heap needs a total order;
    # the index tiebreaks equal values so comparison never reaches unorderables.
    heap = [(lst[0], i, 0) for i, lst in enumerate(lists) if lst]
    heapq.heapify(heap)                # O(k) build
    out = []
    while heap:
        val, src, pos = heapq.heappop(heap)   # global minimum across all lists
        out.append(val)
        if pos + 1 < len(lists[src]):          # successor from the same source
            heapq.heappush(heap, (lists[src][pos + 1], src, pos + 1))
    return out
```

### Max-heap idioms
```java
// Java: flip the comparator
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
PriorityQueue<int[]> byDist = new PriorityQueue<>((a, b) -> a[0] - b[0]); // custom
```
```python
# Python: negate on the way in, negate on the way out
heapq.heappush(heap, -x)
biggest = -heapq.heappop(heap)
```

## 4. Language-Specific Gotchas

- **Python `heapq` is min-only** — negate values for max-heap; for tuples, negate the sort key: `(-count, word)`.
- **Python tuple comparison fallthrough:** `(5, obj1) vs (5, obj2)` compares the objects → `TypeError` if unorderable. Insert a unique counter as tiebreaker: `(priority, id, obj)`.
- **Java comparator overflow:** `(a, b) -> a - b` overflows on extreme ints → use `Integer.compare(a, b)`.
- **Java `PriorityQueue.remove(Object)` is O(n)** — no decrease-key. For Dijkstra-style updates, push duplicates and skip stale entries on pop ("lazy deletion").
- Python: `heapq.heapify(list)` is O(n) in-place; building by n pushes is O(n log n) — use heapify when you have all data.
- Python: `heap[0]` peeks; Java: `peek()`. Neither iterates in sorted order — **iterating a heap gives arbitrary order**, a classic mistake.
- Java records/tuples: use `int[]` or small classes; comparator must be supplied.

## 5. Worked Examples

### Easy — Kth Largest Element in a Stream (LC 703)

**Problem:** Class with `add(val)` returning the kth largest value seen so far.

**Recognition trace:** stream + kth largest → maintain min-heap of exactly K elements; its root *is* the kth largest at all times.

**Java:**
```java
class KthLargest {
    private final PriorityQueue<Integer> heap = new PriorityQueue<>();
    private final int k;

    public KthLargest(int k, int[] nums) {
        this.k = k;
        for (int x : nums) add(x);
    }
    public int add(int val) {
        heap.offer(val);
        if (heap.size() > k) heap.poll();  // keep only the K best
        return heap.peek();                // root = weakest of the best = kth largest
    }
}
```

**Python:**
```python
import heapq

class KthLargest:
    def __init__(self, k: int, nums: list[int]):
        self.k = k
        self.heap = nums[:]
        heapq.heapify(self.heap)              # O(n)
        while len(self.heap) > k:
            heapq.heappop(self.heap)

    def add(self, val: int) -> int:
        heapq.heappush(self.heap, val)
        if len(self.heap) > self.k:
            heapq.heappop(self.heap)
        return self.heap[0]                   # kth largest = min of the top-k
```

**Trace** for `k=3`, init `[4,5,8,2]`, then `add(3), add(5), add(10)`:
```
after init, keep 3 best: heap {4,5,8}  root=4
add 3 : {3,4,5,8}→evict 3 → {4,5,8}   return 4
add 5 : {4,5,5,8}→evict 4 → {5,5,8}   return 5
add 10: {5,5,8,10}→evict 5 → {5,8,10} return 5
```

### Medium — K Closest Points to Origin (LC 973)

**Problem:** Return the k points closest to (0,0).

**Recognition trace:** "K closest" → top-K by distance → **max-heap of size K** (root = farthest of the kept, evicted by anyone closer). Note squared distance — no sqrt needed for comparison.

**Java:**
```java
public int[][] kClosest(int[][] points, int k) {
    // max-heap by distance: farthest of the current K sits at the root
    PriorityQueue<int[]> heap = new PriorityQueue<>(
        (a, b) -> Long.compare(dist(b), dist(a)));      // reversed = max-heap
    for (int[] p : points) {
        heap.offer(p);
        if (heap.size() > k) heap.poll();               // drop the farthest
    }
    return heap.toArray(new int[k][]);
}
private long dist(int[] p) { return (long) p[0] * p[0] + (long) p[1] * p[1]; }
```

**Python:**
```python
import heapq

def k_closest(points: list[list[int]], k: int) -> list[list[int]]:
    heap = []                                  # entries: (-dist, x, y) → max-heap
    for x, y in points:
        d = x * x + y * y                      # sqrt unnecessary for ordering
        heapq.heappush(heap, (-d, x, y))
        if len(heap) > k:
            heapq.heappop(heap)                # pops the LARGEST distance (most negative)
    return [[x, y] for _, x, y in heap]
```

**Trace** for `points = [[1,3],[-2,2],[5,8],[0,1]]`, `k = 2`:
```
(1,3)  d=10 : heap {10}
(-2,2) d=8  : heap {10, 8}          full
(5,8)  d=89 : push → {89,10,8} → evict 89 → {10, 8}
(0,1)  d=1  : push → {10,8,1} → evict 10 → {8, 1}
answer: [(-2,2), (0,1)]
```

### Hard — Find Median from Data Stream (LC 295)

**Problem:** `addNum(num)` and `findMedian()` over a growing stream.

**Recognition trace:** running median → **two heaps**: max-heap `lo` (smaller half), min-heap `hi` (larger half), sizes balanced within 1. Median = `lo`'s root (odd count) or average of both roots. Every add is O(log n), median O(1).

**Java:**
```java
class MedianFinder {
    private final PriorityQueue<Integer> lo =           // smaller half, max on top
        new PriorityQueue<>(Comparator.reverseOrder());
    private final PriorityQueue<Integer> hi =           // larger half, min on top
        new PriorityQueue<>();

    public void addNum(int num) {
        lo.offer(num);                 // 1. tentatively join the lower half
        hi.offer(lo.poll());           // 2. push lower-half max up → halves stay ordered
        if (hi.size() > lo.size())     // 3. rebalance: lo may equal or exceed hi by 1
            lo.offer(hi.poll());
    }
    public double findMedian() {
        return lo.size() > hi.size() ? lo.peek()
                                     : (lo.peek() + hi.peek()) / 2.0;
    }
}
```

**Python:**
```python
import heapq

class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap via negation: smaller half
        self.hi = []   # min-heap: larger half

    def add_num(self, num: int) -> None:
        heapq.heappush(self.lo, -num)                    # join lower half
        heapq.heappush(self.hi, -heapq.heappop(self.lo)) # ship lo's max to hi
        if len(self.hi) > len(self.lo):                  # keep lo >= hi in size
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def find_median(self) -> float:
        if len(self.lo) > len(self.hi):
            return float(-self.lo[0])
        return (-self.lo[0] + self.hi[0]) / 2
```

**Trace** adding `5, 2, 8, 1` (each add: join lo → ship lo's max to hi → rebalance):
```
add 5: lo{5} → ship 5 → hi{5} → rebalance (hi bigger) → lo{5} hi{}     median 5
add 2: lo{5,2} → ship 5 → hi{5} → sizes 1,1 ok → lo{2} hi{5}          median 3.5
add 8: lo{8,2} → ship 8 → hi{5,8} → hi bigger → lo{5,2} hi{8}         median 5
add 1: lo{5,2,1} → ship 5 → hi{5,8} → sizes 2,2 ok → lo{2,1} hi{5,8}  median 3.5
invariant: every lo element ≤ every hi element; sizes differ by ≤ 1
```

## 6. Common Mistakes & Edge Cases

- Wrong heap polarity: top-K **largest** needs a **min**-heap (and vice versa). If your evictions feel backwards, they are.
- Python: forgetting to negate on *both* push and pop for max-heaps.
- Java: `a - b` comparator overflow; also mutating objects after insertion (heap order silently corrupts).
- Iterating heap contents expecting sorted order.
- Two-heap median: adding directly to one heap without the push-through step → halves' ordering invariant breaks.
- Lazy deletion (Dijkstra, sliding window median): forgetting to skip stale entries on pop.
- k > n; empty heap peek; duplicate values (fine for heaps, but tuple tiebreakers may be needed in Python).
- Using a heap when a single pass with min/max tracking suffices (k=1!).

## 7. Fallback Map

- Kth element, one-shot, array mutable → **quickselect**: O(n) average, worth mentioning as the follow-up answer.
- Top-K frequent with bounded counts → **bucket sort** (`01`) is O(n) flat.
- Static sorted data → **binary search** (`05`); no heap needed for fixed data.
- Need arbitrary deletion/lookup + ordering → balanced BST (Java `TreeMap`; Python `sortedcontainers.SortedList`).
- Priority + graph distances → **Dijkstra** (`11`) — the heap is the engine inside it.
- Max in a sliding window → **monotonic deque** (`03`/`04`), O(n) beats heap's O(n log n).
