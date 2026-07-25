# 07_TREES_FUNDAMENTALS

1. Introduction

What this concept is

A tree is a hierarchical data structure made of nodes connected by edges, with one node designated as the root. Each node may have zero or more child nodes, and except for the root, each node has exactly one parent. Trees model hierarchical relationships naturally (file systems, organization charts, DOM in browsers).

Why it exists

Trees allow us to represent nested or hierarchical data efficiently and support fast search, insertion, and deletion patterns in specialized variants (BST, heap, trie). They provide structure for divide-and-conquer algorithms and are the foundation for many advanced data structures and indexes.

What problem it solves

Trees solve the representation of hierarchical relationships and provide efficient ways to search and manage sorted or structured data (e.g., search trees) and to perform range queries or prefix-based queries.

Real-world analogy

Think of a tree like a family tree or an organizational chart. The CEO is the root, managers are intermediate nodes, and individual contributors are leaves.

Where it is used in industry

- Filesystems and directory hierarchies
- XML/HTML DOM trees in browsers
- Database indexing (B-Trees)
- Compiler parse trees (AST)
- Network routing and overlay structures


2. Intuition Section

Visual explanation

Root
 |
 +-- Child A
 |    +-- Grandchild A1
 |    +-- Grandchild A2
 +-- Child B
      +-- Grandchild B1

ASCII diagram

    Root
     / \
   A     B
  / \     \
A1  A2    B1

Think of reading a tree top-down. Many operations (search, traversal) are recursive and process nodes by visiting children.

Key metaphors

- Parent / Child: like manager / employee
- Leaf: node with no children (individual contributor)
- Subtree: a node and all its descendants (a department under a manager)


3. Core Theory

Definitions

- Node: fundamental unit storing value and pointers to children (and optionally parent)
- Root: top node with no parent
- Edge: connection between parent and child
- Leaf: node with no children
- Height: length of the longest downward path to a leaf
- Depth: distance from root to a node
- Degree: number of children a node has

Terminology

- Binary tree: each node has at most two children (left and right)
- Ordered tree: children have a defined order
- Full/Perfect tree: special shapes with constraints on leaves and node counts

Internal working

- Trees are implemented as nodes with references (pointers) to children. Memory for nodes is allocated separately; traversal visits nodes via references.
- Traversals are DFS (preorder, inorder, postorder) and BFS (level-order).

Memory behavior

- Each node requires space for its value and child pointers. For binary tree: typically value + left pointer + right pointer = O(1) per node.
- Entire tree memory is O(n) where n is number of nodes.

Performance implications

- Traversal is O(n) since each node is visited once.
- For search, insertion, deletion: performance depends on the specific tree variant (BST vs balanced trees).


4. Java Implementation

Basic node class

```java
// name=Interview-Mastery/Data_Structures/07_TREES_FUNDAMENTALS.md
public class TreeNode<T> {
    public T val;            // stored value
    public TreeNode<T> left; // left child
    public TreeNode<T> right;// right child

    public TreeNode(T val) {
        this.val = val;
        this.left = null;
        this.right = null;
    }
}
```

Explain every line

- `public class TreeNode<T>`: a generic node so it can store any object type. Generics prevent boxing/unboxing issues and provide type safety.
- `public T val;`: stored value. Using T allows any object type. If you need primitives (int), use wrapper Integer or specialized classes.
- `left` and `right`: references to children. These are pointers; in Java they are references (object pointers under the hood) stored on the heap.
- Constructor sets value and initializes child refs to null.

Standard library usage

Java doesn't have a single "Tree" class in java.util for general trees; specific types exist (e.g., TreeMap, TreeSet) which implement balanced trees under the hood.

Interview implementation

For interview questions, you typically implement TreeNode yourself and write recursive traversals.

Production-grade usage

For production, prefer well-tested libraries or existing structures (TreeMap for ordered maps). For custom trees ensure clear invariants, balancing, and concurrency handling.


5. Python Implementation

Basic node

```python
class TreeNode:
    def __init__(self, val):
        self.val = val
        self.left = None
        self.right = None
```

Explain every line

- `class TreeNode`: simple class; Python is dynamic so no generics—but type hints can be added: def __init__(self, val: int | str) -> None:
- `self.left/right = None`: references to child nodes. Python stores objects on the heap and variables are references to objects.

Pythonic implementation

Many interview solutions use tuples or lists for small problems, but for clarity, classes are preferred. For heavy workloads, using built-in arrays and indices may be more memory efficient.

Production-grade

Use dataclasses for clarity and performance: `@dataclass` with __slots__ to reduce memory overhead.


6. Internal Working

What happens internally

- Node allocation: each node is an object in heap memory. In Java: an object header + fields. In Python: object with dict (unless __slots__ used).
- Pointers/reference: child fields store references (addresses) to other node objects.

Memory allocation

- O(n) nodes each with O(1) pointers → O(n) memory.
- For languages like C/C++, nodes could be tightly packed in memory for cache locality (e.g., arrays, memory pools).

Time complexity reasons

- Traversals visit each node once: O(n).
- Searching in an unbalanced BST can degrade to O(n); balanced trees keep O(log n).

Data movement

- Insert/delete in linked-node tree manipulates pointers; re-linking is O(1) given a discovered position, but finding the position costs search time.

Underlying implementation

- Many high-performance trees use arrays/implicit trees for static structures (heap in arrays) to get better memory locality. Dynamic trees use pointers.


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Insert    | O(h)       |
| Delete    | O(h)       |
| Search    | O(h)       |

Where h = tree height. For balanced binary trees, h = O(log n). For worst-case unbalanced binary tree, h = O(n).

Why

Because operations navigate from root to a leaf or specific node along a path of length up to height h.


8. Space Complexity

- Best case: O(1) auxiliary (in-place traversals iterative), O(n) total storage.
- Average case: O(n) total. Auxiliary O(h) for recursion stack (h = O(log n) for balanced trees).
- Worst case: O(n) auxiliary recursion stack for very unbalanced trees.


9. Common Interview Questions

Beginner

- Implement tree traversals (preorder, inorder, postorder) recursively and iteratively.
- Count nodes, compute height, check if two trees are identical.

Intermediate

- Serialize/deserialize a binary tree.
- Lowest common ancestor in binary tree and BST.

Advanced

- Convert sorted array to balanced BST.
- Implement tree balancing algorithms (AVL rotations).

Example Q: How do you serialize a binary tree? Answer: Use preorder with null markers or level-order with null placeholders. Explain pros/cons.


10. Common Mistakes

Mistake: Using recursion without considering stack depth
Why: deep trees lead to stack overflow
Problem: runtime crash in large inputs
Correct: convert to iterative solutions or increase stack/transform to tail recursion where applicable

Mistake: Confusing tree height vs depth
Why: both terms used loosely
Problem: wrong complexity reasoning
Correct: Height of a node = longest path to a leaf; depth = distance from root.


11. Real Interview Traps

Trap: Assuming balanced tree
Why: Many examples use balanced trees but inputs may be skewed
Fix: mention worst-case O(n)

Trap: Using mutable default arguments in Python node constructors (rare but possible)


12. Real World Applications

- Filesystems, AST, prefix trees, database indexes, routing tables, in-memory caches.


13. Common LeetCode Problems

Easy

- Binary Tree Inorder Traversal
- Maximum Depth of Binary Tree

Medium

- Serialize and Deserialize Binary Tree
- Lowest Common Ancestor of a Binary Tree

Hard

- Binary Tree Maximum Path Sum
- Recover Binary Search Tree


14. Pattern Recognition

When to use trees

- Question involves hierarchy, parent-child relations, nested structures
- You need ordered range queries (use BST or segment trees)
- Prefix queries → Trie

Decision framework

- Need ordered data with fast search → BST/ balanced tree
- Need prefix operations → Trie
- Need range queries → Segment/Fenwick trees


15. Comparison Section

Tree vs Linked List

- Tree: hierarchical, multi-branch; Linked list: linear
- Tree offers logarithmic search (balanced) vs O(n) for unsorted list

Binary Tree vs Binary Search Tree

- Binary tree: no ordering invariant
- BST: left < node < right, enabling search behavior


16. 5 Minute Revision

Tree

- Hierarchical nodes with root, children, leaves
- Traversals: preorder, inorder, postorder, level-order
- Operations cost O(h)
- Use tries for prefix, segment trees for ranges


---

