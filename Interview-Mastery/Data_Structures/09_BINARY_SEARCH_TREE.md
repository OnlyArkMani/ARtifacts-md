# 09_BINARY_SEARCH_TREE

1. Introduction

What it is

A Binary Search Tree (BST) is a binary tree with an ordering invariant: for any node, values in the left subtree are less than the node's value and values in the right subtree are greater (or greater-or-equal depending on convention). This invariant enables efficient search, insertion, and deletion.

Why it exists

BSTs allow ordered data to be stored in a structure that supports average-case O(log n) search and updates, giving both tree-like structure and ordering benefits like in sorted arrays but with faster dynamic updates.

What problem it solves

BSTs provide fast lookup, retrieval of min/max, predecessor/successor queries, and in-order traversal yields sorted order without additional sorting.

Real-world analogy

Think of an alphabetized phonebook split recursively: choose a pivot name at root, left side before pivot, right side after pivot.

Where used

- Databases (as a conceptual structure; production uses balanced variants)
- Ordered maps/sets
- In-memory indexing and range queries


2. Intuition Section

ASCII

        8
       / \
      3   10
     / \    \
    1   6    14
       / \   /
      4  7  13

Search for 7: compare with 8 → go left; compare with 3 → go right; compare with 6 → go right → found.

This reduces comparisons logarithmically if tree is balanced.


3. Core Theory

Definitions

- BST invariant: left < node < right
- Inorder traversal yields sorted order
- Height determines complexity

Operations

- Search: start at root, compare, move left or right
- Insert: find leaf position, attach new node
- Delete: three cases: leaf, one-child, two-children (replace with successor or predecessor)

Property: average depth is O(log n) for random inserts; worst-case O(n) for sorted inserts.


4. Java Implementation

Search

```java
public TreeNode search(TreeNode root, int key) {
    TreeNode cur = root;
    while (cur != null) {
        if (key == cur.val) return cur;
        cur = key < cur.val ? cur.left : cur.right;
    }
    return null;
}
```

Explain: iterative, avoids recursion overhead, moves left/right based on comparison.

Insert

```java
public TreeNode insert(TreeNode root, int key) {
    if (root == null) return new TreeNode(key);
    TreeNode cur = root, parent = null;
    while (cur != null) {
        parent = cur;
        if (key < cur.val) cur = cur.left;
        else cur = cur.right;
    }
    if (key < parent.val) parent.left = new TreeNode(key);
    else parent.right = new TreeNode(key);
    return root;
}
```

Delete (brief)

- If node has two children: find successor (leftmost of right subtree), copy value, delete successor
- If zero/one child: replace node with child


5. Python Implementation

Recursive search/insert

```python
def search(root, key):
    if not root or root.val == key: return root
    return search(root.left, key) if key < root.val else search(root.right, key)

def insert(root, key):
    if not root: return TreeNode(key)
    if key < root.val:
        root.left = insert(root.left, key)
    else:
        root.right = insert(root.right, key)
    return root
```

Explain recursion and base cases.


6. Internal Working

Why O(h)

Operations follow a path from root to a node/leaf, length at most height h.

Why balanced matters

Balanced trees (AVL, red-black) keep h = O(log n). An unbalanced tree degenerates into a linked list.

Memory

O(n) nodes, each storing two child refs and value.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(h)       |
| Delete    | O(h)       |
| Search    | O(h)       |

With h = O(log n) for balanced; worst-case O(n).


8. Space Complexity

- Total: O(n)
- Aux: O(h) recursion; iterative O(1)


9. Common Interview Questions

- Validate BST (is a binary tree a BST?), handle duplicate keys conventions
- Find kth smallest / largest (use inorder)
- Convert sorted array to BST

Q: How to validate a BST? Answer: use range limits during traversal (pass min/max to recursion) or inorder check for strictly increasing sequence.


10. Common Mistakes

Mistake: using naive inorder check that fails for trees with equal keys in incorrect places.
Correct: handle duplicates with strictness rule or defined policy.

Mistake: Using parent pointer incorrectly when deleting root


11. Real Interview Traps

Trap: Deleting node by copying values but forgetting to unlink successor properly.

Trap: Using floating-point keys leading to precision comparison pitfalls.


12. Real World Applications

- Implementation of ordered sets/maps (TreeMap uses red-black trees)
- Range queries in in-memory data structures


13. Common LeetCode Problems

Easy
- Validate Binary Search Tree
- Search in a BST

Medium
- Kth Smallest Element in a BST
- BST Iterator

Hard
- Recover Binary Search Tree
- Serialize/Deserialize with BST assumptions


14. Pattern Recognition

Use BST when data must be kept ordered and frequent insert/delete operations happen and average-case O(log n) is acceptable; prefer balanced variants for guarantees.


15. Comparison

BST vs HashMap

- BST: ordered, supports range queries, O(log n) search in balanced case
- HashMap: unordered, average O(1) search, no ordered traversal without extra work

BST vs Balanced BST (AVL/RB)

- BST no balance guarantees; balanced variants maintain O(log n) operations with extra rotations/cost.


16. 5 Minute Revision

BST

- Invariant: left < node < right
- Search/insert/delete O(h)
- Balanced variants required for worst-case guarantees


---

