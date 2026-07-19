# 02 — Linked List: Sequential Data with Dynamic Structure

## Introduction

### What Is a Linked List?

A **linked list** is a linear data structure where elements (called **nodes**) are connected via **pointers/references**. Each node contains:
1. **Data** — the value you want to store
2. **Next pointer** — reference to the next node

### Real-World Analogy

Imagine a treasure hunt:
- You start at Clue #1
- Clue #1 says: "Go to Clue #5"
- Clue #5 says: "Go to Clue #3"
- Clue #3 says: "You found the treasure!"

Each clue points to the next one. You can't jump directly to Clue #3 without finding Clue #1 first.

### Why Does It Matter?

**Linked Lists solve array problems:**
- Arrays: Insert/delete at front takes O(n) (shift everything)
- Linked Lists: Insert/delete at front takes O(1) (just change pointers)

Linked lists are ideal when you need **dynamic insertion/deletion** especially at the front or middle.

### Where Is It Used in Industry?

**Browser Back Button:**
- Each webpage is a node
- Linked in the order you visited
- Quick to add/remove from history

**Undo/Redo in Text Editors:**
- Each action is a node
- Quick to navigate backward (Undo)

**Music Playlist:**
- Each song is a node
- Quick to insert/delete songs
- Play next song instantly

**LRU Cache:**
- Track recently used items
- Quick to move items around

---

## Intuition Section

### Visual Understanding

```
Array (contiguous memory):
┌────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │  (stored next to each other)
└────┴────┴────┴────┘

Linked List (scattered memory):
Node1        Node2        Node3        Node4
┌────┐      ┌────┐      ┌────┐      ┌────┐
│ 10 │──>   │ 20 │──>   │ 30 │──>   │ 40 │──> NULL
└────┘      └────┘      └────┘      └────┘
Memory:     Memory:     Memory:     Memory:
0x1000      0x3000      0x2000      0x4000
         (not in order!)
```

### Why Insert at Front is O(1)

```
Original: HEAD -> [10] -> [20] -> [30] -> NULL

Insert 5 at front:

Step 1: Create new node
        NEW NODE
        ┌────┐
        │ 5  │
        └────┘

Step 2: Point new node to current HEAD
        [5] -> [10] -> [20] -> [30] -> NULL

Step 3: Update HEAD
        HEAD now points to [5]

Total: 3 operations = O(1)

No shifting needed! This is the power of linked lists.
```

### Why Access is O(n)

```
Linked List: [10] -> [20] -> [30] -> [40] -> NULL

Access element at index 3 (value 40):

Step 1: Start at HEAD: [10]
Step 2: Follow next pointer: [20]
Step 3: Follow next pointer: [30]
Step 4: Follow next pointer: [40] ← Found!

Why O(n)? Must follow n pointers sequentially.
No direct memory address calculation.
```

---

## Core Theory

### Node Structure

```
Single Node:
┌──────────┬──────────┐
│   Data   │   Next   │
├──────────┼──────────┤
│   Value  │ Pointer  │
└──────────┴──────────┘
```

### Types of Linked Lists

**1. Singly Linked List:**
```
[10] -> [20] -> [30] -> NULL
  ↓       ↓       ↓
 next   next    next

Can only go forward.
```

**2. Doubly Linked List:**
```
NULL <- [10] <-> [20] <-> [30] -> NULL
         ↓        ↓        ↓
       prev/next prev/next prev/next

Can go backward and forward.
```

**3. Circular Linked List:**
```
[10] -> [20] -> [30] -> [10] (back to start)

Last node points to first node.
```

### Key Operations

```
1. TRAVERSE — Visit each node sequentially
2. INSERT — Add node at any position
3. DELETE — Remove node from any position
4. SEARCH — Find value in list
5. REVERSE — Reverse the direction
```

---

## Java Implementation

### Node Class Definition

```java
// Node structure
class Node {
    int data;           // Value
    Node next;          // Pointer to next node
    
    Node(int data) {
        this.data = data;
        this.next = null;  // Initially points to nothing
    }
}
```

### Singly Linked List Operations

```java
class LinkedList {
    private Node head;   // Start of the list
    private int size;
    
    LinkedList() {
        this.head = null;
        this.size = 0;
    }
    
    // 1. INSERT at beginning: O(1)
    public void insertAtBeginning(int data) {
        Node newNode = new Node(data);
        newNode.next = head;      // Point to current head
        head = newNode;           // Become the new head
        size++;
    }
    
    // 2. INSERT at end: O(n)
    public void insertAtEnd(int data) {
        Node newNode = new Node(data);
        
        if (head == null) {
            head = newNode;  // Empty list
            size++;
            return;
        }
        
        Node current = head;
        while (current.next != null) {
            current = current.next;  // Traverse to end
        }
        current.next = newNode;  // Link at end
        size++;
    }
    
    // 3. INSERT at index: O(n)
    public void insertAtIndex(int index, int data) {
        if (index < 0 || index > size) {
            throw new IndexOutOfBoundsException();
        }
        
        if (index == 0) {
            insertAtBeginning(data);
            return;
        }
        
        Node newNode = new Node(data);
        Node current = head;
        
        for (int i = 0; i < index - 1; i++) {
            current = current.next;
        }
        
        newNode.next = current.next;
        current.next = newNode;
        size++;
    }
    
    // 4. DELETE at beginning: O(1)
    public void deleteAtBeginning() {
        if (head == null) {
            throw new IllegalStateException("List is empty");
        }
        head = head.next;  // Skip first node
        size--;
    }
    
    // 5. DELETE at end: O(n)
    public void deleteAtEnd() {
        if (head == null) {
            throw new IllegalStateException("List is empty");
        }
        
        if (head.next == null) {
            head = null;  // Only one node
            size--;
            return;
        }
        
        Node current = head;
        while (current.next.next != null) {
            current = current.next;  // Traverse to second-last
        }
        current.next = null;  // Remove last
        size--;
    }
    
    // 6. SEARCH: O(n)
    public boolean contains(int data) {
        Node current = head;
        while (current != null) {
            if (current.data == data) {
                return true;
            }
            current = current.next;
        }
        return false;
    }
    
    // 7. TRAVERSE and print: O(n)
    public void display() {
        Node current = head;
        while (current != null) {
            System.out.print(current.data + " -> ");
            current = current.next;
        }
        System.out.println("NULL");
    }
    
    // 8. REVERSE list: O(n)
    public void reverse() {
        Node prev = null;
        Node current = head;
        Node next = null;
        
        while (current != null) {
            next = current.next;     // Save next
            current.next = prev;     // Reverse pointer
            prev = current;          // Move prev forward
            current = next;          // Move current forward
        }
        head = prev;  // New head
    }
}
```

### Common Interview Patterns

**Pattern 1: Traverse with Two Pointers**
```java
public Node findMiddle(Node head) {
    Node slow = head;   // Moves 1 step
    Node fast = head;   // Moves 2 steps
    
    while (fast != null && fast.next != null) {
        slow = slow.next;        // 1 step
        fast = fast.next.next;   // 2 steps
    }
    
    return slow;  // Middle element
}

// Why this works:
// When fast reaches end, slow is exactly in middle
// Time: O(n), Space: O(1)
```

**Pattern 2: Detect Cycle**
```java
public boolean hasCycle(Node head) {
    Node slow = head;
    Node fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        
        if (slow == fast) {  // They meet = cycle exists
            return true;
        }
    }
    
    return false;  // No cycle
}
```

---

## Python Implementation

### Node Class

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None

class LinkedList:
    def __init__(self):
        self.head = None
        self.size = 0
    
    # Insert at beginning: O(1)
    def insert_at_beginning(self, data):
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node
        self.size += 1
    
    # Insert at end: O(n)
    def insert_at_end(self, data):
        new_node = Node(data)
        
        if not self.head:
            self.head = new_node
            self.size += 1
            return
        
        current = self.head
        while current.next:
            current = current.next
        current.next = new_node
        self.size += 1
    
    # Delete at beginning: O(1)
    def delete_at_beginning(self):
        if not self.head:
            raise ValueError("List is empty")
        self.head = self.head.next
        self.size -= 1
    
    # Search: O(n)
    def contains(self, data):
        current = self.head
        while current:
            if current.data == data:
                return True
            current = current.next
        return False
    
    # Display: O(n)
    def display(self):
        elements = []
        current = self.head
        while current:
            elements.append(str(current.data))
            current = current.next
        print(" -> ".join(elements) + " -> NULL")
    
    # Reverse: O(n)
    def reverse(self):
        prev = None
        current = self.head
        
        while current:
            next_node = current.next
            current.next = prev
            prev = current
            current = next_node
        
        self.head = prev
    
    # Find middle: O(n)
    def find_middle(self):
        slow = self.head
        fast = self.head
        
        while fast and fast.next:
            slow = slow.next
            fast = fast.next.next
        
        return slow
```

---

## Internal Working

### Memory Management

```
Array vs Linked List Memory:

Array (contiguous):
  0x1000 [10]
  0x1004 [20]
  0x1008 [30]  <- Easy to access any element

Linked List (scattered):
  0x5000 [10|ptr:0x2000]
  0x2000 [20|ptr:0x7000]
  0x7000 [30|ptr:NULL]
  
  To access element at "index 2":
  1. Start at 0x5000, get 0x2000
  2. Go to 0x2000, get 0x7000
  3. Go to 0x7000, found!
  Takes 3 steps = O(n)
```

### Insertion Process

```
Insert 15 between 10 and 20:

Before:
[10|ptr:0x2000] -> [20|ptr:0x7000] -> [30|ptr:NULL]

Step 1: Create new node
[15|ptr:NULL]

Step 2: Set new node's pointer to next
[15|ptr:0x2000]
  (now points to 20)

Step 3: Update previous node's pointer
[10|ptr:0x3000] -> [15|ptr:0x2000] -> [20|ptr:0x7000] -> [30|ptr:NULL]

After: [10] -> [15] -> [20] -> [30] -> NULL
```

---

## Time Complexity Table

| Operation | Singly LL | Doubly LL | Circular LL | Why? |
|-----------|-----------|-----------|-------------|------|
| Access by index | O(n) | O(n) | O(n) | Must traverse from head |
| Insert at front | O(1) | O(1) | O(1) | Just change pointers |
| Insert at end | O(n) | O(1) with tail | O(1) with tail | Must traverse to end (unless tracking) |
| Insert at position | O(n) | O(n) | O(n) | Must find position |
| Delete at front | O(1) | O(1) | O(1) | Just unlink |
| Delete at end | O(n) | O(1) | O(1) with proper tracking | Must traverse/backtrack |
| Delete at position | O(n) | O(n) | O(n) | Must find position |
| Search | O(n) | O(n) | O(n) | Linear scan |
| Reverse | O(n) | O(n) | O(n) | Visit each node |

### Space Complexity

**Storage:** O(n) for n nodes
**Extra pointers:** 
- Singly: 1 pointer per node
- Doubly: 2 pointers per node
- Circular: 1 pointer per node (same as singly)

---

## Interview Questions & Answers

### Q1: Reverse a Linked List

**Answer:**
```java
public Node reverseList(Node head) {
    Node prev = null;
    Node current = head;
    
    while (current != null) {
        Node next = current.next;  // Save next
        current.next = prev;       // Reverse pointer
        prev = current;            // Move forward
        current = next;
    }
    
    return prev;  // New head
}

Example:
Before: 1 -> 2 -> 3 -> NULL
After:  3 -> 2 -> 1 -> NULL

Time: O(n), Space: O(1)
```

### Q2: Detect Cycle in Linked List

**Answer:**
```java
public boolean hasCycle(Node head) {
    if (head == null) return false;
    
    Node slow = head;
    Node fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;        // 1 step
        fast = fast.next.next;   // 2 steps
        
        if (slow == fast) {      // Met = cycle
            return true;
        }
    }
    
    return false;  // Fast reached end = no cycle
}

Why it works:
- If no cycle: fast reaches NULL first
- If cycle: fast and slow eventually meet
Time: O(n), Space: O(1)
```

### Q3: Find Middle of Linked List

**Answer:**
```java
public Node findMiddle(Node head) {
    Node slow = head;
    Node fast = head;
    
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    
    return slow;  // Slow is at middle
}

Example:
1 -> 2 -> 3 -> 4 -> 5
Returns 3 (middle)

Time: O(n), Space: O(1)
```

### Q4: Merge Two Sorted Linked Lists

**Answer:**
```java
public Node mergeSorted(Node l1, Node l2) {
    if (l1 == null) return l2;
    if (l2 == null) return l1;
    
    Node dummy = new Node(0);  // Dummy node
    Node current = dummy;
    
    while (l1 != null && l2 != null) {
        if (l1.data <= l2.data) {
            current.next = l1;
            l1 = l1.next;
        } else {
            current.next = l2;
            l2 = l2.next;
        }
        current = current.next;
    }
    
    // Attach remaining
    if (l1 != null) current.next = l1;
    if (l2 != null) current.next = l2;
    
    return dummy.next;
}

Time: O(n + m), Space: O(1)
```

### Q5: Remove Nth Node from End

**Answer:**
```java
public Node removeNthFromEnd(Node head, int n) {
    Node dummy = new Node(0);
    dummy.next = head;
    Node first = dummy;
    Node second = dummy;
    
    // Move first n+1 steps ahead
    for (int i = 0; i <= n; i++) {
        first = first.next;
    }
    
    // Move both until first reaches end
    while (first != null) {
        first = first.next;
        second = second.next;
    }
    
    // Remove the node
    second.next = second.next.next;
    
    return dummy.next;
}

Time: O(n), Space: O(1)
```

---

## Common Mistakes

### ❌ Mistake 1: Null Pointer Exception

```java
// WRONG
Node current = head;
while (current != null) {
    current = current.next;
    System.out.println(current.data);  // NPE when current becomes null
}

// CORRECT
Node current = head;
while (current != null) {
    System.out.println(current.data);
    current = current.next;
}
```

### ❌ Mistake 2: Lost Reference During Modification

```java
// WRONG - Lost reference to remaining list
Node current = head;
while (current != null) {
    current = current.next;  // This line
    current.next = null;     // Breaks entire list
}

// CORRECT - Save reference first
Node current = head;
while (current != null) {
    Node next = current.next;  // Save first
    current.next = null;
    current = next;
}
```

### ❌ Mistake 3: Forgetting to Update Head

```java
// WRONG
public void deleteAtBeginning() {
    head.next = head.next.next;  // This doesn't change head!
}

// CORRECT
public void deleteAtBeginning() {
    head = head.next;  // Update head
}
```

### ❌ Mistake 4: Inefficient Traversal

```java
// INEFFICIENT - Traverses from head each time
public void addToEnd(int... data) {
    for (int val : data) {
        insertAtEnd(val);  // Each call traverses full list O(n²)
    }
}

// EFFICIENT - Single traversal
public void addMultiple(int... data) {
    for (int val : data) {
        Node newNode = new Node(val);
        if (head == null) {
            head = newNode;
        } else {
            // Find last once, reuse
        }
    }
}
```

---

## Real Interview Traps

### Trap 1: Empty List Edge Case

```java
// This might fail on empty list
public Node reverseList(Node head) {
    head.next = null;  // NullPointerException if head is null
}

// Always check:
if (head == null || head.next == null) return head;
```

### Trap 2: Single Node Edge Case

```java
// Fails for single node
Node slow = head;
Node fast = head.next.next;  // NullPointerException if size < 3

// Always check:
if (head == null || head.next == null) return;
```

### Trap 3: Modifying While Iterating

```java
// WRONG - Modifies structure during iteration
Node current = head;
while (current != null) {
    if (shouldDelete(current)) {
        deleteNode(current);  // Changes next pointers
    }
    current = current.next;  // Might skip nodes
}
```

---

## Real World Applications

**Browser History:**
```
insert_at_beginning() for new page
reverse() for back button
```

**Music Playlist:**
```
insert_at_end() to add song
delete_at_position() to skip
```

**Undo/Redo:**
```
insert_at_beginning() for undo
traverse() for redo history
```

---

## Common LeetCode Problems

**Easy:**
1. Reverse Linked List
2. Merge Two Sorted Lists
3. Linked List Cycle

**Medium:**
1. Add Two Numbers
2. Remove Nth Node From End
3. Reorder List

**Hard:**
1. Merge K Sorted Lists
2. LRU Cache
3. Reverse Nodes in K-Group

---

## Pattern Recognition

**Use Linked List when:**
- Insert/delete frequently at front
- Don't know size in advance
- Memory is fragmented
- Building LRU cache

**Use Array when:**
- Need random access
- Know size in advance
- Memory is contiguous
- Cache performance matters

---

## Comparison: Linked List vs Array

| Operation | Array | Linked List |
|-----------|-------|-------------|
| Access | O(1) | O(n) |
| Insert front | O(n) | O(1) |
| Insert end | O(1) amort | O(n) |
| Delete front | O(n) | O(1) |
| Search | O(n) | O(n) |
| Space | O(n) | O(n) + pointers |

---

## 5-Minute Revision

```
Linked List = nodes connected by pointers

Node:
  - data: value
  - next: pointer to next node

Operations:
  Insert front:    O(1)
  Insert end:      O(n)
  Delete front:    O(1)
  Delete end:      O(n)
  Access:          O(n)
  Search:          O(n)
  Reverse:         O(n)

Power Move: Two Pointers
  slow = head
  fast = head
  while fast && fast.next:
    slow.next
    fast.next.next
  → Find middle, detect cycle, etc.

Common Interview Questions:
  1. Reverse linked list
  2. Detect cycle
  3. Find middle
  4. Merge sorted lists
  5. Remove nth from end
```

---

Next: Read **03_STACK.md**
