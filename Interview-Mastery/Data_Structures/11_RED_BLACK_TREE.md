# 11_RED_BLACK_TREE

1. Introduction

What this concept is

A Red-Black Tree (RBT) is a type of self-balancing binary search tree. It augments each node with a color attribute (red or black) and maintains a set of invariants that guarantee the tree remains approximately balanced, ensuring O(log n) worst-case time for search, insertion, and deletion.

Why it exists

To provide a balanced BST variant with amortized efficient insertions and deletions while requiring fewer rotations on average than stricter balancing schemes like AVL. Red-black trees are easier to implement in library-grade code and have been widely adopted for ordered associative containers.

What problem it solves

Prevents degenerate (linear) behavior of BSTs under worst-case insertion orders by enforcing color and black-height properties, limiting the height to O(log n).

Real-world analogy

Imagine organizing a line where every person is labeled red or black with rules on how many blacks must appear between certain positions. The coloring rules prevent long skinny chains and keep the line relatively balanced.

Where it is used in industry

- Java's TreeMap / TreeSet implementations (historically; modern implementations vary)
- C++'s std::map and std::set historically used RB trees
- Kernel data structures and many standard libraries


2. Intuition Section

Core idea: lightly enforce balance using colors and simple rules instead of strict height balancing. The colors guide where rotations and recoloring should happen during insert/delete.

ASCII: simple 3-node tree

    (B)10
     /  \
  (R)5  (R)15

Black-root property keeps root black. No red parent-child pairs allowed.


3. Core Theory

Properties (invariants)

1. Every node is either red or black.
2. The root is black.
3. All leaves (NULL/NIL) are considered black.
4. Red nodes cannot have red children (no two red nodes in a row).
5. For each node, all paths from the node to its descendant leaves have the same number of black nodes (black-height).

Consequences

- These constraints ensure that no path can be more than twice as long as any other, bounding height by O(log n).

Insert/Delete overview

- Insert: add node as red; fix violations (red parent) by rotations and recoloring using cases.
- Delete: more complex; may need to fix black-height violations via rotations and color changes.

Rotation mechanics are identical to AVL, but case-handling differs because of colors.


4. Java Implementation

Node definition

```java
class RBNode {
    int val;
    RBNode left, right, parent;
    boolean red; // true = red, false = black
    RBNode(int v) { val = v; red = true; }
}
```

Insert (conceptual)

- Standard BST insert, color new node red.
- While parent is red, consider uncle color and perform recolor/rotate as per three classic cases (mirror-symmetric handled similarly).

Pseudo: (high-level)

1. Insert node z as red.
2. while (z.parent.red) {
   if (z.parent is left child) {
       y = z.uncle();
       if (y.red) { z.parent.black(); y.black(); z.grandparent.red(); z = z.grandparent; }
       else { if (z is right child) { z = z.parent; leftRotate(z); } z.parent.black(); z.grandparent.red(); rightRotate(z.grandparent); }
   } else mirror...
}
root.black();

Explain each case in comments in the actual code when implementing.


5. Python Implementation

Lightweight node

```python
class RBNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None
        self.parent = None
        self.red = True
```

Insertion fix-up follows the same logic as Java pseudocode. Deletion is lengthy; in interviews, focus on understanding cases and rotations.


6. Internal Working

Why colors help

Colors encode slack: red links let short black-height paths be matched with taller paths that include red nodes. The black-height invariant forces overall balance while allowing flexible local imbalance solved by recoloring and at most a couple rotations.

Height bound intuition

Let bh(x) be black-height of x. Any path has at least bh blacks and at most 2*bh nodes because red nodes cannot be consecutive; from this derive height ≤ 2*log2(n+1).

Memory

Each node stores color bit and parent pointer if convenient; overhead O(1) per node.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(log n)   |
| Delete    | O(log n)   |
| Search    | O(log n)   |

Why: height is O(log n) due to invariants.


8. Space Complexity

- Total: O(n)
- Aux: O(log n) recursion stack, iterative O(1)


9. Common Interview Questions

- Implement insert with fix-up cases.
- Explain why red-black trees guarantee O(log n)
- Compare RB vs AVL — tradeoffs in rotations and lookup speed


10. Common Mistakes

Mistake: forgetting to color the root black at the end of insert fix-up.
Why: early returns or logic errors in loop
Fix: always set root.black() after fix.

Mistake: mishandling NULL leaf nodes as actual nodes instead of sentinel NILs; simplifies invariants if you use sentinel NIL nodes colored black.


11. Real Interview Traps

Trap: Overlooking mirroring of cases — every left-case has a right-case symmetric counterpart.
Trap: Trying to reason through deletion without sentinel NIL nodes leads to many corner cases.


12. Real World Applications

- Implementations of ordered maps and sets in standard libraries
- File systems and kernel trees for interval maps


13. Common LeetCode Problems

While full RB implementation questions are rare in LeetCode, concepts appear in questions about balanced tree properties and library behaviors.


14. Pattern Recognition

When asked about balanced BST guarantees or implementing library-like ordered maps, bring up RB trees. If interviewer asks about fewer rotations vs stricter balance, mention AVL vs RB tradeoffs.


15. Comparison Section

Red-Black vs AVL

- RB: fewer rotations on average, faster insertion/deletion, slightly slower lookups
- AVL: stricter balance, faster lookups, more rotations on update

Use RB for more write-intensive workloads where amortized costs matter.


16. 5 Minute Revision

Red-Black Tree

- BST with color bits and 5 invariants
- Insert: color new node red, fix with recolor/rotate
- Delete: more complex; may require multiple fix steps
- Guarantees O(log n)


---
