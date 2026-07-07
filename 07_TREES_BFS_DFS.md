# 07 — Trees (DFS & BFS)

## 1. Pattern Overview

Trees are recursion made visible. Nearly every tree problem is one of two traversal strategies plus a decision about **what information flows where**:

- **DFS (recursion):** go deep first. The key design question: does information flow **top-down** (pass state as parameters: "what's the max allowed value on this path?") or **bottom-up** (return computed values from children: "what's the height of your subtree?"). Hard tree problems usually flow both ways at once.
- **BFS (queue):** process level by level. Anything phrased "per level", "nearest", "minimum depth" wants BFS.
- **BST bonus structure:** in-order traversal visits values in sorted order; the BST property lets you discard half the tree per step (binary search, structurally).

Why DFS recursion works so cleanly: a tree problem on node `n` is the *same problem* on `n.left` and `n.right` plus O(1) combination logic. Define what your function returns, trust it on children, combine.

**Complexity:** traversals are O(n) time. Space O(h) for DFS (recursion stack, h = height: log n balanced, n degenerate), O(w) for BFS (queue, w = max width, up to n/2).

## 2. Recognition Triggers

- "Level by level", "level order", "zigzag levels", "right side view", "minimum depth" → **BFS**.
- "Depth / height / diameter / balanced" → **bottom-up DFS** (children report up).
- "Path from root", "path sum", "valid within bounds" → **top-down DFS** (parents pass constraints down).
- "Validate BST", "kth smallest in BST" → **in-order traversal** (sorted order) or bounds-passing DFS.
- "Lowest common ancestor" → DFS returning found-status; BST version: walk from root using ordering.
- "Serialize/deserialize" → preorder DFS with null markers, or BFS.
- Any "count/collect all nodes satisfying X" → either traversal; pick DFS for less code.
- "Diameter", "max path sum", "longest X path" → bottom-up DFS where **the value you return ≠ the value you track globally** (return best single arm, track best combined-through-node).
- Constraint smell: n up to 10^4–10^5 nodes → O(n) single traversal expected; if the tree is a BST and you're not exploiting sorted order, you're missing the point.

## 3. Template Code

### Bottom-up DFS (Java)
```java
// Shape for: height, diameter, balance, subtree sums, max path sum.
// Children compute answers, parent combines them.
public int dfs(TreeNode node) {
    if (node == null) return 0;          // base case = identity value for the combine
    int left = dfs(node.left);           // trust recursion: left subtree fully solved
    int right = dfs(node.right);
    // combine step: often ALSO updates a global/instance "best" with left+right+node
    return 1 + Math.max(left, right);    // e.g., height
}
```

### Top-down DFS (Python)
```python
# Shape for: path sums, BST validation, "is this path still legal?" problems.
# Parent passes accumulated state down; leaves decide.
def dfs(node, state) -> bool:
    if not node:
        return base_case_answer
    state = update(state, node.val)       # fold this node into the path state
    if is_leaf(node) :
        return check(state)               # decision happens at the leaf
    return dfs(node.left, state) or dfs(node.right, state)
```

### BFS by level (Java)
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> res = new ArrayList<>();
    if (root == null) return res;
    Queue<TreeNode> q = new ArrayDeque<>();
    q.offer(root);
    while (!q.isEmpty()) {
        int size = q.size();                 // freeze size: exactly one level's nodes
        List<Integer> level = new ArrayList<>();
        for (int i = 0; i < size; i++) {     // drain the level, enqueue the next
            TreeNode cur = q.poll();
            level.add(cur.val);
            if (cur.left != null) q.offer(cur.left);
            if (cur.right != null) q.offer(cur.right);
        }
        res.add(level);
    }
    return res;
}
```

### BFS by level (Python)
```python
from collections import deque

def level_order(root):
    if not root:
        return []
    res, q = [], deque([root])
    while q:
        level = []
        for _ in range(len(q)):        # len(q) NOW = this level's node count
            node = q.popleft()
            level.append(node.val)
            if node.left:
                q.append(node.left)
            if node.right:
                q.append(node.right)
        res.append(level)
    return res
```

## 4. Language-Specific Gotchas

- **Python recursion limit** is 1000 by default — a degenerate tree of 10^4 nodes crashes. `sys.setrecursionlimit(10**6)` or go iterative. Java's stack is bigger but also finite (~10^4–10^5 frames).
- **Java:** no nonlocal closures over primitives — use an instance field, a one-element array `int[] best`, or return richer objects for "global best" DFS. **Python:** `nonlocal best` inside a nested function is the idiom.
- **Python:** default mutable args (`def dfs(node, path=[])`) is a classic footgun — the list persists across calls.
- BST validation bounds: Java `Integer.MIN_VALUE/MAX_VALUE` fails when node values hit those extremes — use `Long` or `null` bounds. Python: `float('-inf')/float('inf')` just works.
- Java `Queue` interface: `offer/poll/peek` (null-returning) vs `add/remove/element` (throwing) — pick one family.
- Python BFS: always `deque`, never `list.pop(0)` (O(n) per pop).

## 5. Worked Examples

### Easy — Maximum Depth of Binary Tree (LC 104)

**Problem:** Return the tree's maximum depth.

**Recognition trace:** "depth" → bottom-up DFS, three lines. Definition: my depth = 1 + max of children's depths; empty tree = 0.

**Java:**
```java
public int maxDepth(TreeNode root) {
    if (root == null) return 0;
    return 1 + Math.max(maxDepth(root.left), maxDepth(root.right));
}
```

**Python:**
```python
def max_depth(root) -> int:
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

**Trace** for tree `[3, 9, 20, null, null, 15, 7]`:
```
        3
       / \
      9  20
         / \
        15  7
dfs(9)=1  (children both 0)
dfs(15)=1  dfs(7)=1 → dfs(20)=1+max(1,1)=2
dfs(3)=1+max(1,2)=3
```

### Medium — Validate Binary Search Tree (LC 98)

**Problem:** Return true if the tree is a valid BST (left subtree < node < right subtree, strictly).

**Recognition trace:** The trap: checking only `child vs parent` is insufficient — a grandchild can violate a grandparent's bound. Correct model: **every node lives in an interval** narrowed by its ancestors → top-down DFS passing `(lo, hi)`.

**Java:**
```java
public boolean isValidBST(TreeNode root) {
    return valid(root, null, null);        // null bounds = unbounded (avoids MIN/MAX trap)
}
private boolean valid(TreeNode n, Integer lo, Integer hi) {
    if (n == null) return true;
    if ((lo != null && n.val <= lo) || (hi != null && n.val >= hi)) return false;
    return valid(n.left, lo, n.val)        // left: everything must be < n.val
        && valid(n.right, n.val, hi);      // right: everything must be > n.val
}
```

**Python:**
```python
def is_valid_bst(root) -> bool:
    def valid(node, lo, hi):
        if not node:
            return True
        if not (lo < node.val < hi):       # node must fit its ancestor-imposed window
            return False
        return (valid(node.left, lo, node.val) and
                valid(node.right, node.val, hi))
    return valid(root, float("-inf"), float("inf"))
```

**Trace** for `[5, 1, 8, null, null, 6, 9]` — subtle case:
```
        5 (−inf, +inf) ✓
       / \
      1   8 (5, +inf) ✓
         / \
        6   9
6 gets window (5, 8) ✓ ;  9 gets window (8, +inf) ✓ → valid
Now swap 6 → 4:  4 gets window (5, 8) ✗  — parent check alone (4 < 8 ✓) would MISS this.
```

### Hard — Binary Tree Maximum Path Sum (LC 124)

**Problem:** A path is any node sequence connected by edges (need not pass the root, can't revisit). Return the maximum path sum.

**Recognition trace:** "best path anywhere in tree" → bottom-up DFS with the **two-value trick**: for each node, (a) the best *downward arm* it can offer its parent = `node + max(leftArm, rightArm, 0)`; (b) the best *complete path through it* = `node + leftArm⁺ + rightArm⁺` — track (b) globally, return (a). Negative arms are clamped to 0 (better to not take them).

**Java:**
```java
public class Solution {
    private int best = Integer.MIN_VALUE;

    public int maxPathSum(TreeNode root) {
        arm(root);
        return best;
    }
    // returns: max sum of a path that starts at node and goes DOWN one side only
    private int arm(TreeNode node) {
        if (node == null) return 0;
        int left = Math.max(arm(node.left), 0);   // clamp: negative arm = skip it
        int right = Math.max(arm(node.right), 0);
        best = Math.max(best, node.val + left + right); // path bending THROUGH node
        return node.val + Math.max(left, right);  // parent may extend only one arm
    }
}
```

**Python:**
```python
def max_path_sum(root) -> int:
    best = float("-inf")

    def arm(node) -> int:
        nonlocal best
        if not node:
            return 0
        left = max(arm(node.left), 0)          # drop negative contributions
        right = max(arm(node.right), 0)
        best = max(best, node.val + left + right)  # candidate: bend through here
        return node.val + max(left, right)         # offer parent the better arm

    arm(root)
    return best
```

**Trace** for `[-10, 9, 20, null, null, 15, 7]`:
```
        -10
        /  \
       9    20
            / \
          15   7
arm(9)=9        best=9
arm(15)=15      best=15
arm(7)=7        best=15
arm(20): left=15 right=7 → best=max(15, 20+15+7)=42 ★  return 20+15=35
arm(-10): left=9 right=35 → best=max(42, -10+9+35)=42  return -10+35=25
answer 42  (path 15 → 20 → 7)
```

## 6. Common Mistakes & Edge Cases

- BST validation via parent-only comparison (misses grandparent violations) or using `<=` where strict `<` is required (duplicate values).
- Confusing "what I return" with "what I track": diameter/max-path problems return an *arm* but record a *bend*. Mixing them gives wrong answers on exactly the tricky test cases.
- BFS without freezing `size = q.size()` before the inner loop → levels bleed together.
- Null root. Single node. Skewed tree (recursion depth!). All-negative values (max path sum: can't return 0 as base "best").
- Java: `Integer.MIN_VALUE` bounds failing on extreme node values.
- Forgetting that in-order of a BST is sorted — many "kth smallest / closest value / recover BST" problems are one in-order pass.
- Modifying a list passed down the recursion without backtracking it (`path.add` → must `path.remove` after; or pass copies and pay O(n²)).

## 7. Fallback Map

- Graph, not tree (cycles, multiple parents)? → **Graphs** (`11`) — add a `visited` set; tree code without one infinite-loops.
- Prefix/string structure on a tree → **Trie** (`08`).
- Need repeated "kth / min / max" as tree changes → **Heap** (`09`) or balanced BST (TreeMap/SortedList).
- Counting subtree shapes/paths with memoization on structure → tree DP (still bottom-up DFS, richer return type).
- "All paths / all combinations of nodes" with exponential output → **Backtracking** (`10`) on the tree.
- Very wide flat trees blowing BFS memory or deep trees blowing DFS stack → switch traversal, or DFS iteratively with an explicit stack (`04`).
