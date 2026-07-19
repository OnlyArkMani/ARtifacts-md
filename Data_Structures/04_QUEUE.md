# 04 — Queue: FIFO (First In, First Out) Data Structure

## Introduction

### What Is a Queue?

A **queue** is a linear data structure that follows the **FIFO (First In, First Out)** principle:
- The **first element added** is the **first element removed**
- Think of a real-world queue: first person in line is first to be served

### Real-World Analogy

**Queue at a Bank:**
- People join at the back
- People are served from the front
- First person to arrive is first to be served
- You can't jump to the front

**Printer Queue:**
- Print jobs added to queue
- Printer processes from front
- First job sent is first printed

### Why Does It Matter?

Queues power:
1. **BFS (Breadth-First Search)** — level-order tree traversal
2. **Task Scheduling** — job queues, print queues
3. **Message Passing** — between threads, processes
4. **Level-order Traversal** — trees, graphs
5. **Resource Management** — ticket systems, waiting lists

### Where Is It Used?

**Operating Systems:**
- CPU scheduling (ready queue)
- I/O queue (disk requests)

**Networking:**
- Packet queue in routers
- Request queue in web servers

**Gaming:**
- Skill queue in games like League of Legends
- Player queue in online games

**Real-time Systems:**
- Event queue in UI frameworks
- Message queue in message brokers

---

## Intuition Section

### Visual Understanding

```
Queue (Line at Bank):

Front                                    Back
  ↓                                        ↓
[Alice] [Bob] [Charlie] [Diana] [Eve]

Operations:

1. ENQUEUE (Join back of line):
   [Alice] [Bob] [Charlie] [Diana] [Eve] [Frank]
                                          ↑ Added at back

2. DEQUEUE (Serve from front):
   [Bob] [Charlie] [Diana] [Eve] [Frank]
   ↑ Removed from front
```

### Why It's O(1)

```
Queue implemented as array with pointers:
[10][20][30][40][50]
 ↑              ↑
front          rear

ENQUEUE 60:
  rear++
  array[rear] = 60
  Time: O(1) - just update pointer

DEQUEUE:
  value = array[front]
  front++
  Time: O(1) - just get value and move pointer

No shifting = O(1) operations!
```

### Circular Queue Problem

```
Linear queue wastes space:

After many operations:
[  ][  ][  ][40][50]
 ↑                ↑
front            rear

Left side has space but can't use it!

Solution: Circular Queue
[50][40][  ][  ][  ]
     ↑              ↑
   rear           front

When rear reaches end, wrap to beginning.
```

---

## Core Theory

### Queue Terminology

```
1. ENQUEUE — add element to rear
2. DEQUEUE — remove element from front
3. FRONT/PEEK — view front element
4. REAR — view rear element
5. EMPTY — check if queue is empty
6. SIZE — number of elements
```

### Queue Properties

1. **FIFO:** First element added is first removed
2. **Two access points:** Add at rear, remove at front
3. **Efficient:** All operations O(1)
4. **Ordered:** Maintains insertion order

### Types of Queues

**Simple Queue:**
```
Front → [1] [2] [3] [4] [5] ← Rear
Enqueue at rear, dequeue from front
```

**Circular Queue:**
```
Wraps around: when rear reaches end, next position is 0
```

**Priority Queue:**
```
Elements removed based on priority, not order
Implemented with heap
```

**Double-ended Queue (Deque):**
```
Can add/remove from both ends
```

---

## Java Implementation

### Using Queue Interface

```java
import java.util.Queue;
import java.util.LinkedList;
import java.util.ArrayDeque;

// Create queue
Queue<Integer> queue = new LinkedList<>();

// ENQUEUE: Add to rear
queue.add(10);      // throws exception if full
queue.offer(20);    // returns false if full

// Queue: [10, 20]

// PEEK: View front without removing
int front = queue.peek();  // 10
int front2 = queue.element();  // 10 (throws exception if empty)

// DEQUEUE: Remove from front
int removed = queue.poll();   // 10, returns null if empty
int removed2 = queue.remove();  // throws exception if empty

// Queue now: [20]

// Check if empty
if (queue.isEmpty()) {
    System.out.println("Queue is empty");
}

// SIZE
int size = queue.size();

// Iterate
for (int num : queue) {
    System.out.println(num);
}
```

### Queue vs Deque

```java
// Queue: Add rear, remove front
Queue<Integer> q = new LinkedList<>();
q.add(1);
q.poll();  // Remove front

// Deque: Add/remove both ends
Deque<Integer> deque = new ArrayDeque<>();
deque.addFirst(1);   // Add front
deque.addLast(2);    // Add rear
deque.removeFirst(); // Remove front: O(1)
deque.removeLast();  // Remove rear: O(1)
deque.pollFirst();   // Remove front, null if empty
deque.pollLast();    // Remove rear, null if empty
```

### Custom Queue Implementation

```java
class MyQueue<T> {
    private Node<T> front;
    private Node<T> rear;
    private int size;
    
    MyQueue() {
        this.front = null;
        this.rear = null;
        this.size = 0;
    }
    
    public void enqueue(T value) {
        Node<T> newNode = new Node<>(value);
        
        if (isEmpty()) {
            front = newNode;
        } else {
            rear.next = newNode;
        }
        rear = newNode;
        size++;
    }
    
    public T dequeue() {
        if (isEmpty()) {
            throw new IllegalStateException("Queue underflow");
        }
        T value = front.data;
        front = front.next;
        size--;
        
        if (isEmpty()) {
            rear = null;  // Both become null
        }
        return value;
    }
    
    public T peek() {
        if (isEmpty()) {
            throw new IllegalStateException("Queue is empty");
        }
        return front.data;
    }
    
    public boolean isEmpty() {
        return size == 0;
    }
    
    public int size() {
        return size;
    }
    
    private class Node<T> {
        T data;
        Node<T> next;
        
        Node(T data) {
            this.data = data;
            this.next = null;
        }
    }
}
```

### Common Pattern: BFS (Breadth-First Search)

```java
public List<Integer> levelOrder(TreeNode root) {
    List<Integer> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        
        // Process all nodes at current level
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            result.add(node.val);
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
    }
    
    return result;
}

Time: O(n), Space: O(w) where w is max width
```

---

## Python Implementation

### Using Collections.deque

```python
from collections import deque

# Create queue
queue = deque()

# ENQUEUE: Add to rear
queue.append(10)      # Add to back
queue.append(20)

# PEEK: View front
front = queue[0]      # 10

# DEQUEUE: Remove from front
removed = queue.popleft()  # 10

# Queue now: [20]

# Check if empty
if not queue:
    print("Queue is empty")

# SIZE
size = len(queue)

# Iterate
for item in queue:
    print(item)
```

### Using Queue Module

```python
from queue import Queue

# Create queue (thread-safe)
q = Queue(maxsize=10)

# ENQUEUE
q.put(10)
q.put(20)

# PEEK (not directly available)
# DEQUEUE
item = q.get()  # 10

# Check if empty
if q.empty():
    print("Queue is empty")

# SIZE
size = q.qsize()
```

### BFS in Python

```python
from collections import deque

def level_order(root):
    if not root:
        return []
    
    result = []
    queue = deque([root])
    
    while queue:
        level_size = len(queue)
        
        for _ in range(level_size):
            node = queue.popleft()
            result.append(node.val)
            
            if node.left:
                queue.append(node.left)
            if node.right:
                queue.append(node.right)
    
    return result
```

---

## Internal Working

### Circular Queue Implementation

```
Linear Queue Problem:
After operations, front moves right:
[  ][  ][10][20][30]
space wasted at left!

Circular Queue Solution:
Wrap around at end:
[20][30][  ][  ][10]
 ↑
When rear reaches end (index 4), next is index 0

Formulas:
enqueue: rear = (rear + 1) % size
dequeue: front = (front + 1) % size
full: (rear + 1) % size == front
empty: rear == front
```

### Queue Memory

```
Linked List Queue (most flexible):
Front Node: [10|ptr] -> [20|ptr] -> [30|ptr]
            ↑ dequeue from here
            
            [30|ptr] -> [40|null]
                        ↑ enqueue here
            
Array Queue (needs size management):
[10][20][30][40][  ]
 ↑              ↑
front          rear
```

---

## Time Complexity Table

| Operation | Linked List | Array (circular) | Why? |
|-----------|-------------|------------------|------|
| Enqueue | O(1) | O(1) | Just add to rear |
| Dequeue | O(1) | O(1) | Just remove from front |
| Peek | O(1) | O(1) | Just view front |
| isEmpty | O(1) | O(1) | Check pointers |

**Overall Space:** O(n) for n elements

---

## Interview Questions & Answers

### Q1: Level Order Traversal (BFS)

**Question:** Visit all nodes level by level

**Answer:**
```java
public List<List<Integer>> levelOrder(TreeNode root) {
    List<List<Integer>> result = new ArrayList<>();
    if (root == null) return result;
    
    Queue<TreeNode> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        int levelSize = queue.size();
        List<Integer> currentLevel = new ArrayList<>();
        
        for (int i = 0; i < levelSize; i++) {
            TreeNode node = queue.poll();
            currentLevel.add(node.val);
            
            if (node.left != null) queue.offer(node.left);
            if (node.right != null) queue.offer(node.right);
        }
        result.add(currentLevel);
    }
    
    return result;
}

Time: O(n), Space: O(w) where w is max width
```

### Q2: Number of Islands (BFS)

**Question:** Count distinct islands in grid

**Answer:**
```java
public int numIslands(char[][] grid) {
    if (grid == null || grid.length == 0) return 0;
    
    int count = 0;
    
    for (int i = 0; i < grid.length; i++) {
        for (int j = 0; j < grid[0].length; j++) {
            if (grid[i][j] == '1') {
                bfs(grid, i, j);
                count++;
            }
        }
    }
    
    return count;
}

private void bfs(char[][] grid, int i, int j) {
    Queue<int[]> queue = new LinkedList<>();
    queue.offer(new int[]{i, j});
    grid[i][j] = '0';  // Mark visited
    
    int[][] directions = {{0,1}, {0,-1}, {1,0}, {-1,0}};
    
    while (!queue.isEmpty()) {
        int[] current = queue.poll();
        
        for (int[] dir : directions) {
            int row = current[0] + dir[0];
            int col = current[1] + dir[1];
            
            if (row >= 0 && row < grid.length &&
                col >= 0 && col < grid[0].length &&
                grid[row][col] == '1') {
                queue.offer(new int[]{row, col});
                grid[row][col] = '0';
            }
        }
    }
}

Time: O(m*n), Space: O(m*n)
```

### Q3: Perfect Squares (BFS)

**Question:** Minimum squares that sum to n

**Answer:**
```java
public int numSquares(int n) {
    Queue<int[]> queue = new LinkedList<>();
    queue.offer(new int[]{n, 0});  // {remaining, steps}
    
    while (!queue.isEmpty()) {
        int[] current = queue.poll();
        int num = current[0];
        int steps = current[1];
        
        if (num == 0) return steps;
        
        for (int i = 1; i * i <= num; i++) {
            queue.offer(new int[]{num - i*i, steps + 1});
        }
    }
    
    return -1;  // Should not reach here
}

Time: O(n), Space: O(n)
```

---

## Common Mistakes

### ❌ Mistake 1: DEQUEUE from Empty Queue

```java
// WRONG
Queue<Integer> queue = new LinkedList<>();
int value = queue.remove();  // NoSuchElementException

// CORRECT
if (!queue.isEmpty()) {
    int value = queue.poll();
}
```

### ❌ Mistake 2: Confusing Front and Rear

```java
// WRONG - Adding to front instead of rear
queue.add(10);
queue.add(20);
queue.remove();  // Should remove 10, not 20

// CORRECT - Use proper methods
queue.offer(10);   // Add to rear
queue.poll();      // Remove from front
```

### ❌ Mistake 3: Infinite Loop in BFS

```java
// WRONG - Not marking as visited
while (!queue.isEmpty()) {
    TreeNode node = queue.poll();
    if (node.left != null) queue.offer(node.left);
    // If node.left points back to node, infinite loop!
}

// CORRECT - Mark visited
Set<TreeNode> visited = new HashSet<>();
while (!queue.isEmpty()) {
    TreeNode node = queue.poll();
    if (!visited.contains(node.left) && node.left != null) {
        queue.offer(node.left);
        visited.add(node.left);
    }
}
```

---

## Real Interview Traps

### Trap 1: Level Size in BFS

```java
// WRONG - Size changes during iteration
while (!queue.isEmpty()) {
    for (int i = 0; i < queue.size(); i++) {
        // queue.size() might change as we add children
    }
}

// CORRECT - Store size before loop
while (!queue.isEmpty()) {
    int levelSize = queue.size();  // Fixed
    for (int i = 0; i < levelSize; i++) {
        // Now size is constant
    }
}
```

### Trap 2: Array Indices

```java
// WRONG - Forgot to check bounds
for (int i = 1; i * i <= n; i++) {
    queue.offer(n - i*i);  // Might be negative
}

// CORRECT - Check before adding
for (int i = 1; i * i <= n; i++) {
    if (n - i*i >= 0) {
        queue.offer(n - i*i);
    }
}
```

---

## Real World Applications

**Printer Queue:**
- Users submit print jobs
- Printer processes from front
- New jobs added to back

**Customer Service:**
- Customers join queue
- First customer served first
- New customers join back

**Task Scheduling:**
- OS maintains ready queue
- Processes scheduled in order
- Fair resource allocation

---

## Common LeetCode Problems

**Easy:**
1. Implement Queue Using Stacks
2. Moving Average from Data Stream
3. Number of Recent Calls

**Medium:**
1. Level Order Traversal
2. Number of Islands
3. Perfect Squares

**Hard:**
1. Trapping Rain Water II
2. Maximal Rectangle

---

## 5-Minute Revision

```
Queue = FIFO (First In, First Out)

Operations (all O(1)):
  Enqueue: Add to rear
  Dequeue: Remove from front
  Peek: View front
  isEmpty: Check if empty

Used for:
  - BFS (Breadth-First Search)
  - Level-order traversal
  - Task scheduling
  - Message queues
  - Resource management

Implementation:
  - Linked List: Flexible, O(1) front/rear
  - Array (Circular): Fixed size, wraparound
  - Java: Queue<>, Deque<>
  - Python: collections.deque

BFS Pattern:
  1. Add root to queue
  2. While queue not empty:
     - Get current level size
     - Process all nodes at level
     - Add children to queue
```

---

Next: Read **05_PRIORITY_QUEUE_HEAP.md**
