# 13 — 2D Dynamic Programming

## 1. Pattern Overview

Same DP discipline as 1D (`12`), but the state needs **two independent dimensions**. The three archetypes cover nearly everything:

- **Two sequences:** `dp[i][j]` = answer for `first i chars of A` × `first j chars of B` (LCS, edit distance, regex matching). Recurrence asks: "do the current characters match, and what's the last edit?"
- **Grid walk:** `dp[r][c]` = answer reaching cell (r,c) from an origin with restricted moves (unique paths, min path sum). Recurrence: combine the cells you could have come from.
- **Interval DP:** `dp[i][j]` = answer for subarray `i..j` (burst balloons, palindromes). Fill by increasing interval *length* — small intervals first, because big ones are built from them.

Why 2D: when two positions advance independently (one per string; row and column; both endpoints of an interval), one index can't capture the subproblem. The state must name *everything the future depends on* — no more, no less.

**Complexity:** usually O(m·n) time/space; space compresses to O(min(m,n)) when each row depends only on the previous row. Interval DP is O(n²) states × O(n) split points = O(n³).

## 2. Recognition Triggers

- **Two strings/arrays compared**: "edit distance", "common subsequence/substring", "interleaving", "wildcard/regex matching", "distinct subsequences" → two-sequence DP, reflexively.
- Grid + "number of paths / min cost path / max collectable" moving right/down (or similar restricted moves) → grid DP. (Contrast: arbitrary movement → BFS/Dijkstra `11`, not DP.)
- "Palindromic substring/subsequence" → interval DP or expand-around-center.
- Operations on a range that split it ("burst balloons", "merge stones", matrix-chain) → interval DP, fill by length.
- Take-or-skip across **two** resources (knapsack with weight: item index × capacity) → 2D knapsack table (often compressible to 1D).
- Constraint smell: two inputs with m, n ≤ 1000–5000 → O(m·n) table intended. n ≤ 500 with an inner split loop → O(n³) interval DP intended.
- Brute-force recursion has exactly two changing parameters → memoize on both = 2D DP found you.

## 3. Template Code

### Two-sequence template — LCS shape (Java)
```java
// dp[i][j] = answer for A's first i chars vs B's first j chars.
// Size (m+1)×(n+1): row/col 0 = empty prefix = base case, avoids all bounds checks.
public int twoSeqDP(String a, String b) {
    int m = a.length(), n = b.length();
    int[][] dp = new int[m + 1][n + 1];        // dp[0][*] and dp[*][0] = 0 (empty vs anything)
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (a.charAt(i - 1) == b.charAt(j - 1))   // note the -1: dp is 1-indexed
                dp[i][j] = dp[i - 1][j - 1] + 1;      // match extends the diagonal
            else
                dp[i][j] = Math.max(dp[i - 1][j],     // drop a's char
                                    dp[i][j - 1]);    // drop b's char
        }
    }
    return dp[m][n];
}
```

### Two-sequence template (Python)
```python
def two_seq_dp(a: str, b: str) -> int:
    m, n = len(a), len(b)
    dp = [[0] * (n + 1) for _ in range(m + 1)]   # comprehension — NOT [[0]*(n+1)]*(m+1)
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if a[i - 1] == b[j - 1]:
                dp[i][j] = dp[i - 1][j - 1] + 1   # diagonal: consume both chars
            else:
                dp[i][j] = max(dp[i - 1][j], dp[i][j - 1])
    return dp[m][n]
```

### Interval DP template (Python)
```python
def interval_dp(nums: list[int]) -> int:
    n = len(nums)
    dp = [[0] * n for _ in range(n)]      # dp[i][j] = answer for interval i..j
    for length in range(2, n + 1):        # LENGTH outer — deps are shorter intervals
        for i in range(0, n - length + 1):
            j = i + length - 1
            for k in range(i, j + 1):     # k = split point / last operation
                dp[i][j] = max(dp[i][j], combine(dp, i, k, j))
    return dp[0][n - 1]
```

### Row compression (Python)
```python
# When dp[i][j] reads only row i-1 and row i → keep two rows (or one + a diagonal temp).
prev = [0] * (n + 1)
for i in range(1, m + 1):
    cur = [0] * (n + 1)
    for j in range(1, n + 1):
        cur[j] = ...  # uses prev[j], prev[j-1], cur[j-1]
    prev = cur
```

## 4. Language-Specific Gotchas

- **Python row-aliasing:** `[[0]*n]*m` copies *references* — writing one row writes all. The single most common Python DP bug. Always `[[0]*n for _ in range(m)]`.
- **Java:** `new int[m+1][n+1]` zero-fills correctly (no aliasing) — one place Java is safer.
- The `+1` offset: `dp[i][j]` describes prefixes of length i, j → string chars are `a[i-1]`, `b[j-1]`. Mixing these silently shifts the whole table.
- Python nested loops are slow: O(m·n) at 5·10^6 cells is fine, O(n³) at n=500 (1.25·10^8 combine steps) can TLE — keep inner loops lean, consider `lru_cache` (often faster to write, similar speed).
- Java memo of `Integer[][]` (null = unseen) vs `int[][]` + sentinel — with 0 a valid answer, null is safer.
- In-place compression direction: 0/1 knapsack over one row must iterate capacity **descending** (or you reuse this row's own updates = unbounded knapsack).

## 5. Worked Examples

### Easy — Unique Paths (LC 62)

**Problem:** m×n grid, start top-left, move only right/down, count paths to bottom-right.

**Recognition trace:** grid + "count paths" + restricted moves → grid DP. Any cell is reached from above or left: `dp[r][c] = dp[r-1][c] + dp[r][c-1]`.

**Java:**
```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];              // one row suffices: dp[c] = ways to reach (r, c)
    Arrays.fill(dp, 1);                 // first row: only one way (all rights)
    for (int r = 1; r < m; r++)
        for (int c = 1; c < n; c++)
            dp[c] += dp[c - 1];         // dp[c] (from above) + dp[c-1] (from left)
    return dp[n - 1];
}
```

**Python:**
```python
def unique_paths(m: int, n: int) -> int:
    dp = [1] * n                        # top row: 1 way everywhere
    for _ in range(1, m):
        for c in range(1, n):
            dp[c] += dp[c - 1]          # above (old dp[c]) + left (new dp[c-1])
    return dp[-1]
```

**Trace** for m=3, n=4 (full table view):
```
1   1   1   1
1   2   3   4        each cell = above + left
1   3   6  10   → 10 paths
fill order: row by row, left to right (deps: ↑ and ←, both ready)
```

### Medium — Longest Common Subsequence (LC 1143)

**Problem:** Length of the longest subsequence common to `text1` and `text2`.

**Recognition trace:** two strings + "longest common subsequence" → the definitional two-sequence DP; template above *is* the solution.

**Trace** for `text1 = "abcde"`, `text2 = "ace"`:
```
        ""  a  c  e
    ""   0  0  0  0
    a    0  1  1  1     ← 'a'='a': diag+1
    b    0  1  1  1
    c    0  1  2  2     ← 'c'='c': diag+1
    d    0  1  2  2
    e    0  1  2  3     ← 'e'='e': diag+1  → LCS = 3 ("ace")
Mismatch cells take max(↑, ←); match cells take ↖ + 1.
```

### Hard — Edit Distance (LC 72)

**Problem:** Min operations (insert, delete, replace) converting `word1` → `word2`.

**Recognition trace:** two strings + min operations → two-sequence DP. `dp[i][j]` = min ops for first i of word1 → first j of word2. Chars match → free diagonal. Else 1 + best of: replace (diag), delete from word1 (up), insert into word1 (left). Base row/col: converting to/from empty = i (or j) deletions/insertions.

**Java:**
```java
public int minDistance(String w1, String w2) {
    int m = w1.length(), n = w2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;   // delete all i chars
    for (int j = 0; j <= n; j++) dp[0][j] = j;   // insert all j chars
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (w1.charAt(i - 1) == w2.charAt(j - 1))
                dp[i][j] = dp[i - 1][j - 1];              // match: free
            else
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], // replace
                             Math.min(dp[i - 1][j],       // delete w1's char
                                      dp[i][j - 1]));     // insert w2's char
        }
    }
    return dp[m][n];
}
```

**Python:**
```python
def min_distance(w1: str, w2: str) -> int:
    m, n = len(w1), len(w2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]
    for i in range(m + 1):
        dp[i][0] = i                    # i deletions
    for j in range(n + 1):
        dp[0][j] = j                    # j insertions
    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if w1[i - 1] == w2[j - 1]:
                dp[i][j] = dp[i - 1][j - 1]
            else:
                dp[i][j] = 1 + min(dp[i - 1][j - 1],   # replace
                                   dp[i - 1][j],       # delete
                                   dp[i][j - 1])       # insert
    return dp[m][n]
```

**Trace** for `horse → ros`:
```
        ""  r  o  s
    ""   0  1  2  3
    h    1  1  2  3      h≠r: 1+min(0,1,2)=1 (replace h→r)
    o    2  2  1  2      o=o: diag ✓
    r    3  2  2  2      r=r: diag ✓
    s    4  3  3  2      s=s: diag ✓
    e    5  4  4  3      e≠s: 1+min(2,3,3... )=3 (delete e)
answer 3:  horse → rorse (replace) → rose (delete) → ros (delete)
```

## 6. Common Mistakes & Edge Cases

- Python `[[0]*n]*m` row aliasing (again — it's that common).
- The 1-index offset: reading `a[i]` where `a[i-1]` is meant — shifts the whole table one cell.
- Missing/wrong base row and column (edit distance without `dp[i][0] = i` returns nonsense).
- Interval DP filled in index order instead of **length order** — reads unfilled cells (as zeros) silently.
- Knapsack compression iterating ascending when items are single-use.
- Confusing subsequence (gaps allowed → LCS recurrence) with substring (contiguous → different recurrence: reset on mismatch, answer = max cell).
- O(n³) interval DP attempted at n = 5000 (misreading which input is small).
- Empty string inputs (base cases should handle them for free — test it).
- Returning `dp[m-1][n-1]` from an (m+1)×(n+1) table.

## 7. Fallback Map

- One sequence, one changing index → **1D DP** (`12`) — don't add dimensions you don't need.
- Grid with arbitrary movement (up/down/left/right, weights) → **BFS/Dijkstra** (`11`); DP requires acyclic dependency (right/down-style).
- Palindrome problems: expand-around-center is O(n²) time, O(1) space — often simpler than the DP table (LC 5).
- Two-string problems where greedy matching works (subsequence check `is s a subsequence of t`) → **two pointers** (`02`) in O(n); no table needed.
- State = (index, small bitmask of used items) → bitmask DP (1D over 2^n masks) — n ≤ 20 territory.
- Table too big (10^5 × 10^5) → the intended solution isn't tabular DP: look for greedy, binary search, or a smarter state.
