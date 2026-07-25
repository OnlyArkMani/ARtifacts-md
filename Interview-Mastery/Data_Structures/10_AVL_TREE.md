# 10_AVL_TREE

1. Introduction

What it is

An AVL tree is a self-balancing binary search tree where the heights of two child subtrees of any node differ by at most one. If after insertion or deletion the balance property is violated, rotations are performed to restore it.

Why it exists

To keep binary search tree operations guaranteed at O(log n) worst-case by maintaining tree height logarithmic in the number of nodes.

What problem it solves

Avoids degeneration of BST into a linear structure for pathological insertion orders (e.g., sorted input), guaranteeing logarithmic operations.

Real-world analogy

Think of keeping a bookshelf balanced: after adding or removing books you occasionally rearrange shelves (rotations) so the left and right sides aren't excessively lopsided.

Where used

Historically used in libraries and systems that need fast guaranteed operations. Today red-black trees are more common in language libraries, but AVL trees are still valuable when faster lookups are desired at the cost of more rotations on inserts/deletes.


2. Intuition Section

ASCII

Before rotation (left heavy):

    z
   / \
  y   T4
 / \
 x  T3
/ \
T1 T2

Right rotation at z moves y up, z becomes y's right child.

After rotation:

    y
   / \
  x   z
 /\   /\
T1 T2 T3 T4

Goal: restore balance so subtree heights differ by at most 1.


3. Core Theory

Definitions

- Balance factor of node = height(left) - height(right)
- AVL invariant: balance factor ∈ {-1,0,1}

Rotations

- Right rotation (RR)
- Left rotation (LL)
- Left-Right (LR) double rotation
- Right-Left (RL) double rotation

When to rotate

- After insertion or deletion, walk up from changed node to root, update heights and check balance. When |balance| > 1, perform appropriate rotation.

Height maintenance

Each node stores height (often) and is updated during upward traversal in O(1) per node changed.


4. Java Implementation

Node with height

```java
class AVLNode {
    int val;
    AVLNode left, right;
    int height;
    AVLNode(int v) { val = v; height = 1; }
}
```

Utility: getHeight

```java
int height(AVLNode n) { return n == null ? 0 : n.height; }
int getBalance(AVLNode n) { return n == null ? 0 : height(n.left) - height(n.right); }
```

Right rotation

```java
AVLNode rightRotate(AVLNode z) {
    AVLNode y = z.left;
    AVLNode T3 = y.right;

    y.right = z;
    z.left = T3;

    z.height = Math.max(height(z.left), height(z.right)) + 1;
    y.height = Math.max(height(y.left), height(y.right)) + 1;

    return y; // new root
}
```

Explain lines

- Save pointers, perform pointer rewiring, update heights (bottom-up), return new root for the subtree.

Insert with balancing (outline)

- Insert like BST, then update heights, check balance, perform rotations based on cases (LL, RR, LR, RL).


5. Python Implementation

(omitted full code for brevity in this batch) Provide same structure: node with height, rotate functions, insert balancing.


6. Internal Working

Why rotations fix height

Rotations reassign subtree roots without breaking BST invariant and reduce the height of the taller subtree while increasing the shorter side slightly, bringing balance factors within [-1,1].

Cost

- Each rotation is O(1)
- Insert/delete cost includes path traversal O(h) plus O(1) rotations per unbalanced node, so still O(log n)

Memory

- Each node stores height int; auxiliary O(n).


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(log n)   |
| Delete    | O(log n)   |
| Search    | O(log n)   |

Why: height is O(log n) due to balancing.


8. Space Complexity

- Total: O(n)
- Auxiliary: O(log n) recursion stack for balanced trees


9. Common Interview Questions

- Implement AVL insert with rotations
- Prove height bound: h < 1.44 * log2(n+2) - 0.328 (sketch proof via Fibonacci-like recurrence)


10. Common Mistakes

Mistake: forgetting to update heights after rotation
Why it happens: mixing pointer rewiring and height updates
Problem: later balance checks wrong
Correct approach: always update children heights before parent when rewiring

Mistake: incorrect rotation case detection (mixing up LR vs RL)


11. Real Interview Traps

Trap: expecting fewer rotations than needed; some sequences require multiple rotations on consecutive inserts.

Trap: confusing height with balance factor sign conventions.


12. Real World Applications

- Scenarios requiring strict lookup guarantees where RB tree's slightly weaker guarantee and cheaper writes aren’t sufficient. E.g., certain database indexing and in-memory ordered sets.


13. Common LeetCode Problems

- Implement AVL tree insertion (rare on LeetCode; more common on educational platforms)
- Self-balancing BST problems


14. Pattern Recognition

Use AVL when you need strictly faster lookups than red-black typically provides (AVL has tighter balance) at the cost of more rotations during inserts/deletes.


15. Comparison Section

AVL vs Red-Black

- AVL: stricter balance, faster lookups, more rotations on updates
- Red-Black: looser balance, faster updates, preferred in standard libraries (TreeMap)


16. 5 Minute Revision

AVL

- Self-balancing BST, balance factor in {-1,0,1}
- Rotations: LL, RR, LR, RL
- Operations O(log n)


---

