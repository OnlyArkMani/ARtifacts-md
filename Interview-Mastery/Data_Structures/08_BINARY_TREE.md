# 08_BINARY_TREE

1. Introduction

What this concept is

A binary tree is a tree data structure where each node has at most two children, commonly referred to as left and right. It's a foundational data structure used for many algorithms and higher-level structures such as binary search trees, heaps, and expression trees.

Why it exists

To model hierarchical data with a fixed branching factor of two, simplifying algorithms (especially recursive ones) and enabling ordered variations like BST and heap.

What problem it solves

Binary trees provide a simple structured way to store hierarchical data where nodes can be processed with divide-and-conquer. They balance the branching complexity and are easy to traverse recursively.

Real-world analogy

Think of a tournament bracket where each match leads to two players and winners advance, forming a binary structure.

Where used in industry

- Expression parsing (AST)
- Heaps for priority queues
- Binary indexed algorithms in compilers and interpreters


2. Intuition Section

ASCII diagram

      1
     / \
    2   3
   / \   \
  4   5   6

Think of each node as splitting the problem into left and right halves. Recursion processes a node and its left and right subproblems.


3. Core Theory

Key properties

- Full binary tree: every node has 0 or 2 children
- Complete binary tree: all levels except possibly the last are completely filled, and nodes are as left as possible
- Perfect binary tree: all internal nodes have two children and all leaves are at same depth

Traversals

- Preorder (node, left, right)
- Inorder (left, node, right)
- Postorder (left, right, node)
- Level order (BFS)

Use-cases of traversals

- Inorder on BST yields sorted order
- Postorder useful to delete/free nodes or evaluate expression trees


4. Java Implementation

Basic traversal templates

```java
public void inorder(TreeNode root) {
    if (root == null) return;
    inorder(root.left);
    System.out.print(root.val + " ");
    inorder(root.right);
}
```

Explain lines

- Base case checks null and returns — prevents recursion overflow.
- Recursively traverse left subtree, process node, then right subtree.

Iterative inorder using stack

```java
public void inorderIter(TreeNode root) {
    Deque<TreeNode> stack = new ArrayDeque<>();
    TreeNode cur = root;
    while (cur != null || !stack.isEmpty()) {
        while (cur != null) { stack.push(cur); cur = cur.left; }
        cur = stack.pop();
        System.out.print(cur.val + " ");
        cur = cur.right;
    }
}
```

Explain: stack simulates recursion; push lefts, pop process, then go right.


5. Python Implementation

Recursive

```python
def inorder(root):
    if not root:
        return []
    return inorder(root.left) + [root.val] + inorder(root.right)
```

Explain: returns list concatenation which is fine for small trees but creates many temporary lists; better to use generator or accumulator.

Generator version

```python
def inorder_gen(node):
    if not node:
        return
    yield from inorder_gen(node.left)
    yield node.val
    yield from inorder_gen(node.right)
```


6. Internal Working

Memory layout

- Linked-node: each node stored separately on heap; pointers connect them
- Array-based (for complete trees): nodes stored in array level-order; index arithmetic (left = 2*i+1, right = 2*i+2) for zero-based

Why use array representation

- For complete binary trees and heaps, array gives excellent cache locality and O(1) parent/child access by index.

Recursion stack

- Recursive traversals use O(h) call stack. Iterative stack simulates this explicitly.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(n) worst for general binary tree (to find position), O(log n) if heap/complete maintained |
| Delete    | O(n) worst |
| Search    | O(n) |

Why: general binary tree has no ordering invariant; operations often require full traversal.


8. Space Complexity

- Total: O(n)
- Auxiliary: O(h) recursion/stack


9. Common Interview Questions

- Implement traversals (iterative and recursive)
- Check if two trees are mirrors
- Tree flattening to linked list


10. Common Mistakes

Mistake: Using list concatenation in Python recursion (O(n^2) in some patterns)
Correct: use generators or pass accumulator list

Mistake: Forgetting base-case null check in recursion


11. Real Interview Traps

Trap: Interpreting "binary tree" as BST implicitly. Always clarify if there's an ordering invariant.


12. Real World Applications

- Heaps (priority queue)
- Expression trees
- File system binary decompositions for algorithms


13. Common LeetCode Problems

Easy
- Maximum Depth of Binary Tree
- Symmetric Tree

Medium
- Binary Tree Level Order Traversal
- Construct Binary Tree from Preorder and Inorder

Hard
- Binary Tree Maximum Path Sum


14. Pattern Recognition

Use general binary tree when: structure is hierarchical but no ordering guarantee is required. Ask clarifying questions.


15. Comparison

Binary Tree vs Array

- Binary tree (node refs) flexible, dynamic size
- Array (implicit tree) efficient for complete trees/heap


16. 5 Minute Revision

Binary Tree

- Each node ≤ 2 children
- Traversals: preorder/inorder/postorder/level-order
- General binary tree ops O(n)


---

