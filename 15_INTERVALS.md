# 15 — Intervals

## 1. Pattern Overview

Interval problems look diverse (merge, insert, meeting rooms, arrows) but are one idea repeated: **sort, then sweep, tracking a tiny amount of state** (the current merged interval, the last end time, a heap of active end times).

Why sorting unlocks everything: once intervals are sorted by start, **overlap becomes a purely local question** — interval `i+1` can only overlap the *current merged block*, never something two blocks back. Sorting converts a quadratic all-pairs comparison into a linear neighbor comparison.

The two facts to internalize:

- **Overlap test:** `a` and `b` overlap ⟺ `a.start <= b.end && b.start <= a.end`. After sorting by start, this simplifies to `next.start <= cur.end`.
- **Merge:** merged = `[min(starts), max(ends)]` — and with sorted starts, just `cur.end = max(cur.end, next.end)`.

Second tool: **sweep line** — treat starts as +1 events and ends as −1 events, sort events, track a running counter. "Max simultaneous intervals" (meeting rooms) falls out immediately.

**Complexity:** O(n log n) for the sort, O(n) sweep. Heap variants O(n log n).

## 2. Recognition Triggers

- The input is pairs `[start, end]`: meetings, jobs, ranges, bookings → this playbook.
- "Merge overlapping X" → sort by start + linear merge.
- "Insert an interval into a sorted list" → three phases: before / overlapping (merge) / after.
- "Minimum rooms / platforms / resources needed" → max simultaneous overlap → sweep line or min-heap of end times.
- "Can attend all meetings?" → sort + check any `next.start < cur.end`.
- "Remove minimum intervals to make non-overlapping" / "max non-overlapping set" → greedy by **end** time (`14`).
- "Burst balloons with arrows" style (stab all intervals with fewest points) → greedy by end time.
- "Employee free time", "gaps between merged blocks" → merge, then read the gaps.
- Constraint smell: n ≤ 10^5 intervals → O(n log n) sort-based; queries against intervals → sort + binary search (`05`) or heap by end (`09`).

## 3. Template Code

### Merge sweep (Java)
```java
// Invariant: 'cur' is the merged block still growing; anything before it is final.
public int[][] merge(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0])); // by start
    List<int[]> out = new ArrayList<>();
    int[] cur = intervals[0].clone();
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= cur[1]) {               // overlaps (or touches) cur
            cur[1] = Math.max(cur[1], intervals[i][1]); // absorb — note the max!
        } else {
            out.add(cur);                               // gap → cur is finished
            cur = intervals[i].clone();
        }
    }
    out.add(cur);                                       // don't forget the last block
    return out.toArray(new int[0][]);
}
```

### Merge sweep (Python)
```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    intervals.sort(key=lambda iv: iv[0])       # by start
    out = [intervals[0][:]]
    for start, end in intervals[1:]:
        if start <= out[-1][1]:                # overlaps the growing block
            out[-1][1] = max(out[-1][1], end)  # extend; max guards nested intervals
        else:
            out.append([start, end])           # disjoint → new block
    return out
```

### Sweep line — max simultaneous (Python)
```python
def max_overlap(intervals: list[list[int]]) -> int:
    events = []
    for s, e in intervals:
        events.append((s, 1))       # meeting starts: +1 room
        events.append((e, -1))      # meeting ends: room frees
    # Sort by time; on ties, ends BEFORE starts (end at t frees a room for a start at t)
    events.sort(key=lambda ev: (ev[0], ev[1]))
    rooms = best = 0
    for _, delta in events:
        rooms += delta
        best = max(best, rooms)
    return best
```

## 4. Language-Specific Gotchas

- **Tie-breaking is a correctness issue, not style:** if `[1,3]` and `[3,5]` don't count as overlapping (usual for meetings), ends must sort before starts at equal times — Python `(t, delta)` works because −1 < +1; in Java write the comparator explicitly.
- **Java comparator overflow:** `a[0] - b[0]` → use `Integer.compare`.
- Python `intervals.sort()` mutates the input — fine on LeetCode, mention it in interviews.
- Copying: Java `clone()` vs aliasing the input rows; Python `intervals[0][:]` — mutating `out[-1]` must not corrupt the input if it's still needed.
- Java: building `List<int[]>` then `toArray(new int[0][])` — the idiom is worth memorizing; hand-rolling arrays mid-merge is bug-prone.
- Python heapq for meeting rooms II: heap of end times; `heap[0]` = earliest-freeing room.

## 5. Worked Examples

### Easy — Meeting Rooms (LC 252)

**Problem:** Can one person attend all meetings (no overlaps)?

**Recognition trace:** overlap existence check → sort by start; any `next.start < cur.end` → clash.

**Java:**
```java
public boolean canAttendMeetings(int[][] intervals) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));
    for (int i = 1; i < intervals.length; i++)
        if (intervals[i][0] < intervals[i - 1][1])  // starts before previous ends
            return false;
    return true;
}
```

**Python:**
```python
def can_attend_meetings(intervals: list[list[int]]) -> bool:
    intervals.sort(key=lambda iv: iv[0])
    return all(intervals[i][0] >= intervals[i - 1][1]
               for i in range(1, len(intervals)))
```

**Trace** for `[[0,30],[5,10],[15,20]]`:
```
sorted: [0,30] [5,10] [15,20]
  [0──────────────30]
       [5───10]           5 < 30 → clash → False
```

### Medium — Merge Intervals (LC 56)

**Problem:** Merge all overlapping intervals.

**Recognition trace:** "merge overlapping" → the definitional sort + sweep; template above is the solution.

**Trace** for `[[1,3],[2,6],[8,10],[15,18]]`:
```
sorted:  [1,3] [2,6] [8,10] [15,18]
cur=[1,3]:  2 ≤ 3 → absorb → cur=[1,6]
            8 > 6 → emit [1,6], cur=[8,10]
            15 > 10 → emit [8,10], cur=[15,18] → emit
1───3
  2─────6        →  1───────6   8──10   15──18
         8──10
              15──18
```

### Hard — Minimum Interval to Include Each Query (LC 1851)

**Problem:** Given intervals and queries, for each query q return the **size of the smallest** interval containing q (−1 if none).

**Recognition trace:** many queries × many intervals → per-query scan is O(n·q), dead at 10^5 each. Classic offline trick: **sort both**, sweep queries in ascending order, add intervals as their starts come into range (push `(size, end)` onto a min-heap by size), pop heap-top intervals that already ended. Heap root = smallest interval still covering q.

**Java:**
```java
public int[] minInterval(int[][] intervals, int[] queries) {
    Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));  // by start
    int[] qSorted = queries.clone();
    Arrays.sort(qSorted);
    Map<Integer, Integer> ans = new HashMap<>();          // query value -> answer
    PriorityQueue<int[]> heap =                            // [size, end], smallest size on top
        new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));
    int i = 0;
    for (int q : qSorted) {
        while (i < intervals.length && intervals[i][0] <= q) {   // interval now "open"
            heap.offer(new int[]{intervals[i][1] - intervals[i][0] + 1, intervals[i][1]});
            i++;
        }
        while (!heap.isEmpty() && heap.peek()[1] < q)     // already closed → useless forever
            heap.poll();                                   // (queries only grow) 
        ans.putIfAbsent(q, heap.isEmpty() ? -1 : heap.peek()[0]);
    }
    int[] res = new int[queries.length];
    for (int j = 0; j < queries.length; j++) res[j] = ans.get(queries[j]);
    return res;
}
```

**Python:**
```python
import heapq

def min_interval(intervals: list[list[int]], queries: list[int]) -> list[int]:
    intervals.sort()                      # by start
    heap = []                             # (size, end) — min-heap by size
    ans = {}
    i = 0
    for q in sorted(set(queries)):        # process queries left → right
        while i < len(intervals) and intervals[i][0] <= q:
            s, e = intervals[i]
            heapq.heappush(heap, (e - s + 1, e))   # interval is now a candidate
            i += 1
        while heap and heap[0][1] < q:    # ended before q — and before all later q too
            heapq.heappop(heap)
        ans[q] = heap[0][0] if heap else -1
    return [ans[q] for q in queries]      # restore original query order
```

**Trace** for `intervals = [[1,4],[2,4],[3,6],[4,4]]`, `queries = [2, 3, 4, 5]`:
```
sorted intervals: [1,4](sz4) [2,4](sz3) [3,6](sz4) [4,4](sz1)

q=2: open [1,4],[2,4]           heap: (3,[2,4]) (4,[1,4])        → ans 3
q=3: open [3,6]                 heap: (3,..) (4,..) (4,[3,6])    → ans 3
q=4: open [4,4]                 heap: (1,[4,4]) (3,..) (4,..) (4,..) → ans 1
q=5: heap top (1, end=4) < 5 → pop; (3, end=4) < 5 → pop; (4,[1,4])? end 4 < 5 → pop
     remaining (4,[3,6]) end 6 ≥ 5                                → ans 4
result: [3, 3, 1, 4]
Why sorting queries is safe: an interval ending before q ends before every later q → pops are permanent.
```

## 6. Common Mistakes & Edge Cases

- Forgetting `max(cur.end, next.end)` when absorbing — nested intervals (`[1,10], [2,3]`) shrink the block if you just assign.
- Emitting the last merged block: the final `out.add(cur)` after the loop is forgotten constantly.
- Touching endpoints: `[1,2]` and `[2,3]` — overlapping or not? Problem-dependent; pick `<=` vs `<` consciously, and match the sweep-line tie order to it.
- Sorting by end when the algorithm needs start (merging) or by start when it needs end (activity selection `14`) — know which and why.
- Mutating input rows that the caller (or your own later logic) still reads.
- Sweep line: processing all events at time t in arbitrary order instead of ends-first.
- Single interval; fully nested chains; all identical intervals; queries outside every interval.
- Java: `Integer.compare` vs subtraction overflow with coordinates up to 10^9.

## 7. Fallback Map

- "Max non-overlapping / min removals / min arrows" → **Greedy by end time** (`14`) — same data, different question.
- "Min rooms at once" → sweep line here, or **min-heap of end times** (`09`) — equivalent; heap version generalizes to "which room".
- Point-in-interval queries, online (can't sort queries) → sorted starts + **binary search** (`05`), or an interval tree (mention, don't build).
- Intervals + weights ("max profit non-overlapping jobs") → sort + **DP with binary search** (weighted interval scheduling — `12` + `05`).
- 2D versions (rectangles, skyline) → sweep line + heap/multiset: the Skyline Problem (LC 218) is this file's ideas at the next difficulty level.
- Counting overlaps over huge coordinate ranges → coordinate compression + difference array (`01` prefix-sum thinking).
