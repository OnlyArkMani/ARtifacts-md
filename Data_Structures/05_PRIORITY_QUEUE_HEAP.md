# 05 — Priority Queue & Heap: Efficient Priority-Based Access

## Introduction

### What Is a Heap?

A **heap** is a specialized tree-based data structure that satisfies the **heap property**:
- **Min-Heap:** Parent ≤ Children (smallest at root)
- **Max-Heap:** Parent ≥ Children (largest at root)

It enables efficient retrieval of the highest (or lowest) priority element.

### Real-World Analogy

**Hospital Emergency Room:**
- Patients arrive with different urgency levels
- Critical patients served first (highest priority)
- Not necessarily first-come-first-serve
- Doctor picks the most urgent patient

**Streaming Platform (Netflix):**
- Shows sorted by "trending now"
- Most popular shown first
- Dynamically updated

### Why Does It Matter?

Heaps solve the **"find maximum/minimum efficiently" problem**:
- Array: O(n) to find max
- Sorted array: O(log n) binary search but O(n) to maintain
- **Heap: O(log n) to insert/remove, O(1) to peek max/min**

This is ideal for:
1. **Top-K problems** (K most frequent, K largest)
2. **Dijkstra's algorithm** (shortest paths)
3. **Priority queues** (task scheduling)
4. **Huffman coding** (data compression)
5. **Merge K sorted lists**

### Where Is It Used?

**Google Search:**
- Rank results by relevance score
- Need top 10 instantly
- Heap maintains this efficiently

**Airport Boarding:**
- First class, business, economy
- Board by priority
- Efficient management

**Stock Trading:**
- Monitor top gainers/losers
- Update in real-time

---

## Intuition Section

### Visual Understanding

**Min-Heap Example:**
```
         1
       /   \
      3     5
     / \   /
    7   10 40

Property: Parent ≤ Children
1 < 3, 1 < 5, 3 < 7, 3 < 10, 5 < 40 ✓
```

**Max-Heap Example:**
```
         100
        /    \
       30    50
      / \    /
     10  20  5

Property: Parent ≥ Children
100 > 30, 100 > 50, 30 > 10, 30 > 20, 50 > 5 ✓
```

### Why Heap is O(log n)

```
Heap with n elements is a BALANCED binary tree
Height = log₂(n)

Insert/Delete operations go from root to leaf
Traverse at most log₂(n) levels = O(log n)

Example: 1,000 elements
Height = log₂(1000) ≈ 10
So insert/delete takes ~10 steps max

Compare to array:
Find max in 1,000 elements = 1,000 comparisons
```

### Array Representation

```
Heap stored as array (no explicit pointers needed!):

Tree:
       1
      / \
     3   5
    / \ /
   7 10 40

Array: [1, 3, 5, 7, 10, 40]
       0  1  2  3   4   5

Formulas for index i:
- Left child: 2i + 1
- Right child: 2i + 2  
- Parent: (i - 1) / 2

Example: index 1 (value 3)
- Left child: 2(1) + 1 = 3 (value 7)
- Right child: 2(1) + 2 = 4 (value 10)
- Parent: (1 - 1) / 2 = 0 (value 1)
```

---

## Core Theory

### Heap Properties

1. **Complete Binary Tree:** All levels filled except possibly last (filled left-to-right)
2. **Heap Property:** Parent is min/max compared to children
3. **Efficient:** Operations in O(log n)
4. **Array-Based:** Can be stored as simple array
5. **Not Sorted:** Full sorting is O(n log n), but heap maintains only heap property

### Heap Operations

```
1. INSERT: Add element, bubble up if needed
2. DELETE (extract-min/max): Remove root, move last to root, bubble down
3. PEEK: View min/max without removing
4. HEAPIFY: Convert array into valid heap
```

### Min-Heap vs Max-Heap

```
Min-Heap: Smallest at root
- Use for: Find minimum efficiently
- LeetCode: Default for priority queue problems

Max-Heap: Largest at root
- Use for: Find maximum efficiently
- Common: K largest elements
```

---

## Java Implementation

### Using PriorityQueue (Min-Heap by default)

```java
import java.util.PriorityQueue;

// Create min-heap
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// INSERT: O(log n)
minHeap.add(50);
minHeap.add(30);
minHeap.offer(70);
minHeap.offer(10);
minHeap.offer(40);

// Heap: [10, 30, 40, 50, 70]

// PEEK: O(1)
int min = minHeap.peek();  // 10 (don't remove)

// EXTRACT: O(log n)
int extracted = minHeap.poll();  // 10 (remove + return)

// SIZE
int size = minHeap.size();

// CHECK EMPTY
if (minHeap.isEmpty()) {
    System.out.println("Empty");
}

// ITERATE (not necessarily in order)
for (int num : minHeap) {
    System.out.println(num);
}
```

### Max-Heap in Java

```java
// Max-heap using comparator
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);

maxHeap.add(50);
maxHeap.add(30);
maxHeap.add(70);
maxHeap.add(10);

int max = maxHeap.peek();  // 70
int extracted = maxHeap.poll();  // 70
```

### Custom Objects in Heap

```java
class Task implements Comparable<Task> {
    String name;
    int priority;  // Higher number = higher priority
    
    Task(String name, int priority) {
        this.name = name;
        this.priority = priority;
    }
    
    @Override
    public int compareTo(Task other) {
        return Integer.compare(this.priority, other.priority);
    }
}

PriorityQueue<Task> taskQueue = new PriorityQueue<>();
taskQueue.add(new Task("Email", 1));
taskQueue.add(new Task("Meeting", 5));
taskQueue.add(new Task("Code", 3));

Task mostUrgent = taskQueue.poll();  // Meeting (priority 5)
```

### Common Pattern: Top-K Elements

```java
public int[] topKFrequent(int[] nums, int k) {
    // Count frequencies
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.put(num, freq.getOrDefault(num, 0) + 1);
    }
    
    // Min-heap of size k
    PriorityQueue<Integer> heap = new PriorityQueue<>(
        (a, b) -> freq.get(a) - freq.get(b)
    );
    
    for (int num : freq.keySet()) {
        heap.offer(num);
        if (heap.size() > k) {
            heap.poll();  // Remove least frequent
        }
    }
    
    int[] result = new int[k];
    int i = k - 1;
    while (!heap.isEmpty()) {
        result[i--] = heap.poll();
    }
    return result;
}

Time: O(n log k), Space: O(n)
```

---

## Python Implementation

### Using heapq Module

```python
import heapq

# Create min-heap (default)
heap = []

# INSERT: O(log n)
heapq.heappush(heap, 50)
heapq.heappush(heap, 30)
heapq.heappush(heap, 70)
heapq.heappush(heap, 10)
heapq.heappush(heap, 40)

# Heap: [10, 30, 40, 50, 70]

# PEEK: O(1)
min_val = heap[0]  # 10 (don't remove)

# EXTRACT: O(log n)
min_extracted = heapq.heappop(heap)  # 10

# Convert list to heap: O(n)
arr = [50, 30, 70, 10, 40]
heapq.heapify(arr)  # Convert to valid heap in-place

# SIZE
size = len(heap)

# Nlargest / Nsmallest: O(n log k)
largest_3 = heapq.nlargest(3, heap)   # [70, 50, 40]
smallest_2 = heapq.nsmallest(2, heap) # [10, 30]
```

### Max-Heap in Python

```python
import heapq

# Python heapq is min-heap by default
# For max-heap, negate values
heap = []

# INSERT (negate for max-heap)
heapq.heappush(heap, -50)
heapq.heappush(heap, -30)
heapq.heappush(heap, -70)

max_val = -heap[0]  # 70
max_extracted = -heapq.heappop(heap)  # 70
```

### Top-K in Python

```python
def topKFrequent(nums, k):
    from collections import Counter
    import heapq
    
    # Count frequencies
    freq = Counter(nums)
    
    # Return k most frequent
    # Use negative frequencies for max-heap behavior
    return heapq.nlargest(k, freq.keys(), key=freq.get)
```

---

## Internal Working

### Insert Operation (Bubble Up)

```
Insert 5 into min-heap [10, 30, 40, 50, 70]:

Step 1: Add to end
[10, 30, 40, 50, 70, 5]

Step 2: Bubble up (compare with parent)
Parent of index 5 is index (5-1)/2 = 2 (value 40)
5 < 40? Yes, swap
[10, 30, 5, 50, 70, 40]

Step 3: Continue
Parent of index 2 is index (2-1)/2 = 0 (value 10)
5 < 10? No, stop

Final: [10, 30, 5, 50, 70, 40]
```

### Delete Operation (Bubble Down)

```
Delete min from [10, 30, 5, 50, 70, 40]:

Step 1: Remove root, move last to root
[40, 30, 5, 50, 70]

Step 2: Bubble down
Children of index 0:
- Left: index 1 (value 30)
- Right: index 2 (value 5)
Smaller child is 5
40 > 5? Yes, swap
[5, 30, 40, 50, 70]

Step 3: Continue from index 2
Children of index 2:
- Left: index 5 (doesn't exist)
- Right: doesn't exist
Stop

Final: [5, 30, 40, 50, 70]
```

### Heapify Process

```
Convert array to valid heap:
[50, 30, 70, 10, 40]

Heapify: Start from last non-leaf node
Last non-leaf: index (5-1)/2 = 2

Bubble down each node:
Time: O(n) - faster than inserting n times!
```

---

## Time Complexity Table

| Operation | Time | Space | Why? |
|-----------|------|-------|------|
| Insert | O(log n) | O(1) | Bubble up, tree height = log n |
| Delete | O(log n) | O(1) | Bubble down, tree height = log n |
| Peek Min/Max | O(1) | O(1) | Direct access to root |
| Heapify | O(n) | O(1) | Build from bottom up |
| Extract All (sorted) | O(n log n) | O(n) | n extracts, each O(log n) |

**Overall Space:** O(n) for n elements

---

## Interview Questions & Answers

### Q1: Kth Largest Element

**Question:** Find Kth largest element in array

**Answer:**
```java
public int findKthLargest(int[] nums, int k) {
    // Min-heap of size k
    PriorityQueue<Integer> heap = new PriorityQueue<>();
    
    for (int num : nums) {
        heap.offer(num);
        if (heap.size() > k) {
            heap.poll();  // Remove smallest
        }
    }
    
    return heap.peek();  // Kth largest is min of k largest
}

Time: O(n log k), Space: O(k)
```

### Q2: Merge K Sorted Lists

**Question:** Merge k sorted linked lists

**Answer:**
```java
public ListNode mergeKLists(ListNode[] lists) {
    if (lists == null || lists.length == 0) return null;
    
    // Min-heap of nodes by value
    PriorityQueue<ListNode> heap = new PriorityQueue<>(
        (a, b) -> a.val - b.val
    );
    
    // Add first node of each list
    for (ListNode list : lists) {
        if (list != null) {
            heap.offer(list);
        }
    }
    
    ListNode dummy = new ListNode(0);
    ListNode current = dummy;
    
    while (!heap.isEmpty()) {
        ListNode smallest = heap.poll();
        current.next = smallest;
        current = current.next;
        
        if (smallest.next != null) {
            heap.offer(smallest.next);
        }
    }
    
    return dummy.next;
}

Time: O(n log k), Space: O(k) where n is total nodes
```

### Q3: Median of Data Stream

**Question:** Find median as data streams in

**Answer:**
```java
class MedianFinder {
    private PriorityQueue<Integer> maxHeap;  // Left half (larger values)
    private PriorityQueue<Integer> minHeap;  // Right half (smaller values)
    
    public MedianFinder() {
        maxHeap = new PriorityQueue<>((a, b) -> b - a);  // Max-heap
        minHeap = new PriorityQueue<>();  // Min-heap
    }
    
    public void addNum(int num) {
        // Add to max-heap first
        maxHeap.offer(num);
        
        // Ensure every element in min-heap >= every element in max-heap
        if (!maxHeap.isEmpty() && !minHeap.isEmpty() &&
            maxHeap.peek() > minHeap.peek()) {
            int val = maxHeap.poll();
            minHeap.offer(val);
        }
        
        // Maintain size: |maxHeap| = |minHeap| or |maxHeap| = |minHeap| + 1
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.offer(maxHeap.poll());
        }
        if (minHeap.size() > maxHeap.size()) {
            maxHeap.offer(minHeap.poll());
        }
    }
    
    public double findMedian() {
        if (maxHeap.size() == minHeap.size()) {
            return (maxHeap.peek() + minHeap.peek()) / 2.0;
        }
        return maxHeap.peek();
    }
}

Time: O(log n) per operation, Space: O(n)
```

---

## Common Mistakes

### ❌ Mistake 1: Wrong Comparator

```java
// WRONG - This creates max-heap but we want min
PriorityQueue<Integer> heap = new PriorityQueue<>((a, b) -> b - a);
int min = heap.peek();  // Not actually min!

// CORRECT
PriorityQueue<Integer> heap = new PriorityQueue<>();  // Min-heap by default
int min = heap.peek();
```

### ❌ Mistake 2: Modifying After Insert

```java
// WRONG - Heap property violated
List<Integer> arr = new ArrayList<>();
arr.add(50);
arr.set(0, 5);  // Doesn't maintain heap property

// CORRECT - Always use offer/poll
PriorityQueue<Integer> heap = new PriorityQueue<>();
heap.offer(50);
heap.poll();
heap.offer(5);
```

### ❌ Mistake 3: Empty Heap Access

```java
// WRONG
PriorityQueue<Integer> heap = new PriorityQueue<>();
int val = heap.peek();  // NoSuchElementException

// CORRECT
if (!heap.isEmpty()) {
    int val = heap.peek();
}
```

---

## Real World Applications

**Dijkstra's Algorithm:**
```
Find shortest path in weighted graph
Use min-heap to always process nearest unvisited vertex
```

**Huffman Coding:**
```
Data compression algorithm
Build optimal prefix tree using min-heap
```

**Task Scheduling:**
```
Prioritize tasks by urgency
Execute highest priority first
```

**Stock Price Tracking:**
```
Track K highest prices
Update efficiently with K-sized heap
```

---

## Common LeetCode Problems

**Easy:**
1. Kth Largest Element in Stream
2. Relative Sort Array

**Medium:**
1. Kth Largest Element in Array
2. Top K Frequent Elements
3. Merge K Sorted Lists

**Hard:**
1. Median of Data Stream
2. Trapping Rain Water II
3. Smallest Range Covering Elements

---

## Pattern Recognition

**Use Heap when you see:**
- "Kth largest/smallest"
- "Top K"
- "Merge K sorted"
- "Frequent/most popular"
- "Priority-based"
- "Find max/min efficiently"

---

## 5-Minute Revision

```
Heap = Complete binary tree + heap property

Min-Heap: Parent ≤ Children
Max-Heap: Parent ≥ Children

Operations (all involve tree height log n):
  Insert: O(log n)
  Delete: O(log n)
  Peek: O(1)
  Heapify: O(n)

Array representation:
  Left child of i: 2i + 1
  Right child of i: 2i + 2
  Parent of i: (i - 1) / 2

Common problems:
  1. Kth largest → Min-heap of size K
  2. Top K frequent → Count + min-heap
  3. Merge K sorted → Min-heap of heads
  4. Median stream → Two heaps

Java:
  PriorityQueue<T> heap = new PriorityQueue<>();
  heap.offer(x);
  heap.poll();
  heap.peek();

Python:
  import heapq
  heapq.heappush(heap, x)
  heapq.heappop(heap)
  heap[0]
```

---

Next: Read **06_HASHMAP_HASHSET.md**
