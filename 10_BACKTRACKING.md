# 10 — Backtracking

## 1. Pattern Overview

Backtracking is **structured brute force**: explore every candidate solution by building it one choice at a time, and the moment a partial solution can't possibly work, abandon it (**prune**) and undo the last choice (**backtrack**).

The universal skeleton — burn this in:

```
def backtrack(state):
    if goal_reached(state):   record answer;  return
    for choice in choices(state):
        if not valid(choice): continue        # prune
        apply(choice)                          # choose
        backtrack(state)                       # explore
        undo(choice)                           # un-choose  ← the defining move
```

Why it works: the choices form a **decision tree**; DFS visits every leaf (complete candidate), and the undo step means all branches share one mutable state object instead of copying it — that's the efficiency trick. Pruning cuts entire subtrees that provably contain no answers.

**Complexity:** exponential by nature — O(2^n) subsets, O(n!) permutations, O(k^n) general. Space O(n) for the recursion path. **This is why n ≤ 20 is the tell.** Your job isn't to beat the exponential; it's to not do worse than the count of actual answers.

## 2. Recognition Triggers

- **n ≤ 20** (or ≤ 12 for permutations, ≤ 9-ish for grids) → exponential intended → backtracking.
- "Return **all** ..." — all subsets, all permutations, all combinations, all paths, all valid X. Enumerate-everything = backtracking.
- "Generate all valid configurations" (parentheses, N-Queens placements, sudoku).
- "Partition into ..." (palindrome partitioning, k equal subsets).
- "Word search in a grid" (single word) → DFS + un-mark = backtracking.
- "Combination sum": choose numbers (with/without reuse) hitting a target.
- Constraint satisfaction: place items subject to mutual restrictions.
- **Anti-triggers:** "count the number of ways" with n large → **DP** (`12`) — you need the count, not the objects. "Find any one solution / optimal value" with n large → DP or greedy. Backtracking is for when the *output itself* is exponential.

## 3. Template Code

### Subsets / combinations skeleton (Java)
```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> res = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), res);
    return res;
}
// 'start' enforces order (i only picks from i onward) → no duplicate sets like {2,1}/{1,2}
private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> res) {
    res.add(new ArrayList<>(path));            // COPY — path keeps mutating after this
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);                     // choose
        backtrack(nums, i + 1, path, res);     // explore (i+1: each element used once)
        path.remove(path.size() - 1);          // un-choose
    }
}
```

### Subsets / combinations skeleton (Python)
```python
def subsets(nums: list[int]) -> list[list[int]]:
    res = []
    path = []

    def backtrack(start: int) -> None:
        res.append(path[:])          # snapshot; appending `path` itself aliases!
        for i in range(start, len(nums)):
            path.append(nums[i])     # choose
            backtrack(i + 1)         # explore from the next index
            path.pop()               # un-choose — state restored for the sibling
    backtrack(0)
    return res
```

### Permutations skeleton (Python)
```python
def permutations(nums: list[int]) -> list[list[int]]:
    res, path = [], []
    used = [False] * len(nums)          # order matters now → no 'start'; track usage

    def backtrack() -> None:
        if len(path) == len(nums):      # complete permutation
            res.append(path[:])
            return
        for i in range(len(nums)):
            if used[i]:
                continue
            used[i] = True              # choose
            path.append(nums[i])
            backtrack()
            path.pop()                  # un-choose BOTH pieces of state
            used[i] = False
    backtrack()
    return res
```

**Dedup with sorted input (duplicates in nums):** `if i > start and nums[i] == nums[i-1]: continue` — skip a duplicate *at the same tree level* (it would rebuild an identical subtree), while still allowing duplicates *along a path*.

## 4. Language-Specific Gotchas

- **Python `res.append(path)` without copying** — the #1 bug. Every entry ends up aliasing the same (finally empty) list. Use `path[:]` or `list(path)`.
- **Java equivalent:** `res.add(path)` — same aliasing bug; `new ArrayList<>(path)`.
- Java `List.remove(int)` vs `remove(Object)`: `path.remove(path.size()-1)` removes by index — with `List<Integer>` beware `remove(Integer.valueOf(x))` confusion.
- Python recursion limit (default 1000): fine for n ≤ 20 backtracking, but grid DFS on large boards can hit it.
- Python closures: `nonlocal` needed only for rebinding scalars; mutating `path`/`res` needs nothing.
- Grid problems: mark visited by mutating the board (`'#'`) then restore — cheaper than a visited set in both languages, but **must** restore.
- Java: prefer `StringBuilder` with explicit `deleteCharAt` for string building; Python: build a list of chars, `''.join` at the leaves.

## 5. Worked Examples

### Easy — Subsets (LC 78)

**Problem:** Return all subsets of distinct `nums`.

**Recognition trace:** "all subsets" + n ≤ 10 → enumerate the decision tree; `start` index prevents reorderings. The template above is the solution.

**Decision-tree trace** for `[1, 2, 3]`:
```
                       []
          ┌────────────┼────────────┐
        [1]           [2]          [3]
      ┌───┴───┐        │
   [1,2]   [1,3]    [2,3]
      │
  [1,2,3]
Every node is recorded (subsets), not just leaves.
res = [], [1], [1,2], [1,2,3], [1,3], [2], [2,3], [3]   (2³ = 8 ✓)
```

### Medium — Combination Sum (LC 39)

**Problem:** Distinct candidates, target; return all unique combinations summing to target. A number may be reused unlimited times.

**Recognition trace:** "all combinations summing to target" → backtracking. Reuse allowed → recurse with `i`, not `i + 1`. Prune when the running sum exceeds target (sort first to break early).

**Java:**
```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates);                       // enables early break on overshoot
    List<List<Integer>> res = new ArrayList<>();
    backtrack(candidates, 0, target, new ArrayList<>(), res);
    return res;
}
private void backtrack(int[] cand, int start, int remain,
                       List<Integer> path, List<List<Integer>> res) {
    if (remain == 0) {                             // exact hit → record
        res.add(new ArrayList<>(path));
        return;
    }
    for (int i = start; i < cand.length; i++) {
        if (cand[i] > remain) break;               // sorted → all later ones too big
        path.add(cand[i]);
        backtrack(cand, i, remain - cand[i], path, res); // i (not i+1): reuse allowed
        path.remove(path.size() - 1);
    }
}
```

**Python:**
```python
def combination_sum(candidates: list[int], target: int) -> list[list[int]]:
    candidates.sort()
    res, path = [], []

    def backtrack(start: int, remain: int) -> None:
        if remain == 0:
            res.append(path[:])
            return
        for i in range(start, len(candidates)):
            c = candidates[i]
            if c > remain:            # prune: sorted, so every later c is too big
                break
            path.append(c)
            backtrack(i, remain - c)  # same i → element may repeat
            path.pop()
    backtrack(0, target)
    return res
```

**Trace** for `candidates = [2, 3, 6, 7]`, `target = 7`:
```
[]r7 ─2→ [2]r5 ─2→ [2,2]r3 ─2→ [2,2,2]r1 ─2→ 2>1 break; 3>1 break        dead
                          └3→ [2,2,3]r0 ✓
              └3→ [2,3]r2 ─3→ 3>2 break                                   dead
     ─3→ [3]r4 ─3→ [3,3]r1 → dead
     ─6→ [6]r1 → dead
     ─7→ [7]r0 ✓
res = [[2,2,3], [7]]
```

### Hard — N-Queens (LC 51)

**Problem:** Place n queens on an n×n board so none attack each other; return all boards.

**Recognition trace:** classic constraint satisfaction, n ≤ 9 → backtracking row by row. Key efficiency: represent attacks as **sets** — columns, diagonals (`r - c` constant), anti-diagonals (`r + c` constant) — making validity O(1) instead of rescanning the board.

**Java:**
```java
public class Solution {
    private final List<List<String>> res = new ArrayList<>();
    private boolean[] cols, diag, anti;   // attacked lines
    private int[] queenCol;               // queenCol[r] = column of queen in row r
    private int n;

    public List<List<String>> solveNQueens(int n) {
        this.n = n;
        cols = new boolean[n];
        diag = new boolean[2 * n];        // index r - c + n (shifted non-negative)
        anti = new boolean[2 * n];        // index r + c
        queenCol = new int[n];
        place(0);
        return res;
    }
    private void place(int r) {
        if (r == n) { res.add(render()); return; }     // all rows filled
        for (int c = 0; c < n; c++) {
            if (cols[c] || diag[r - c + n] || anti[r + c]) continue;  // attacked → prune
            cols[c] = diag[r - c + n] = anti[r + c] = true;           // choose
            queenCol[r] = c;
            place(r + 1);                                              // explore
            cols[c] = diag[r - c + n] = anti[r + c] = false;          // un-choose
        }
    }
    private List<String> render() {
        List<String> board = new ArrayList<>();
        for (int r = 0; r < n; r++) {
            char[] row = new char[n];
            Arrays.fill(row, '.');
            row[queenCol[r]] = 'Q';
            board.add(new String(row));
        }
        return board;
    }
}
```

**Python:**
```python
def solve_n_queens(n: int) -> list[list[str]]:
    res = []
    queen_col = [0] * n
    cols, diag, anti = set(), set(), set()   # O(1) attack checks

    def place(r: int) -> None:
        if r == n:
            res.append(["." * c + "Q" + "." * (n - c - 1) for c in queen_col])
            return
        for c in range(n):
            if c in cols or (r - c) in diag or (r + c) in anti:
                continue                       # square under attack → prune
            cols.add(c); diag.add(r - c); anti.add(r + c)
            queen_col[r] = c
            place(r + 1)
            cols.remove(c); diag.remove(r - c); anti.remove(r + c)
    place(0)
    return res
```

**Trace** for n = 4 (rows top-down; `.`=empty, `Q`=queen, `x`=pruned column):
```
r0: try c0 → r1: c0,c1 attacked; try c2 → r2: all 4 attacked → BACKTRACK
              try c3 → r2: c1 only → r3: all attacked → BACKTRACK ... r0 c0 exhausted
r0: try c1 → r1: c3 → r2: c0 → r3: c2 ✓   solution 1:
    . Q . .
    . . . Q
    Q . . .
    . . Q .
r0: try c2 → (mirror) → solution 2
r0: c3 → dead-ends → total 2 solutions
```

## 6. Common Mistakes & Edge Cases

- Recording `path` without copying (aliasing) — the universal bug.
- Forgetting to undo **all** state pieces (e.g., popping `path` but not clearing `used[i]`).
- Duplicates: skipping at the wrong place. Rule: sort, then skip `nums[i] == nums[i-1]` only when `i > start` (same level), never when it's the first use at this level.
- Combination vs permutation confusion: `start` index (combinations, order-free) vs `used[]` (permutations, order matters).
- Reuse-allowed problems: recursing with `i + 1` instead of `i` (or vice versa).
- Missing pruning entirely — correct but TLE. Sort + break on overshoot is usually the expected prune.
- Grid DFS: not restoring the visited mark; walking off-board without bounds checks.
- Base case order: check goal *before* iterating choices, or you'll miss/duplicate answers.
- Empty input; target 0; k = 0; board 1×1.

## 7. Fallback Map

- "**Count** ways" / "best value", n large → **DP** (`12`, `13`) — same decision tree, but overlapping subproblems collapse via memoization.
- Only need ONE valid solution → backtracking still fine but add aggressive pruning/heuristics; or reduce to graph search.
- Choices at each step don't interact (no shared constraints) → straight recursion/math (product rule), no undo needed.
- n ≤ 20 but 2^n still too slow with heavy per-leaf work → **meet in the middle** (split, enumerate halves, combine).
- Optimization with monotone structure → **Greedy** (`14`) or **Binary search on answer** (`05`).
- Grid shortest path (not enumeration) → **BFS** (`11`), never backtracking.
