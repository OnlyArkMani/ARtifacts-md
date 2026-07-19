# 01 — Arrays: The Foundation of All Data Structures

## Introduction

### What Is an Array?

An **array** is a collection of elements stored in **contiguous memory locations**. Each element can be accessed using an **index** (position), starting from 0.

### Real-World Analogy

Think of an apartment building:
- The building is the array
- Each apartment is an element
- Apartment numbers are indices
- You can instantly jump to any apartment using its number

### Why Does It Matter?

Arrays are the foundation because:
1. **Direct access:** Get any element in O(1) time
2. **Memory efficient:** Stores data in contiguous memory (CPU cache friendly)
3. **Foundation for everything:** Stacks, queues, and many algorithms use arrays internally
4. **50% of problems:** Array and string problems dominate interviews

### Where Is It Used in Industry?

**Google Search:**
- Pages are ranked by relevance (array of results)
- Position in results matters

**Netflix Recommendations:**
- Movies stored as arrays
- Sort by rating, popularity, date

**Stock Trading Systems:**
- Historical prices stored as array
- Quick access to past prices
- Calculate trends

**Gaming (Unity/Unreal):**
- Sprites and textures stored in arrays
- Pixel data in image arrays

---

## Intuition Section

### Visual Understanding

Let's visualize an array:

```
Array: [10, 20, 30, 40, 50]

Index:  0   1   2   3   4
Value:  10  20  30  40  50

Memory Layout (contiguous):
┌────┬────┬────┬────┬────┐
│ 10 │ 20 │ 30 │ 40 │ 50 │  (Stored next to each other in RAM)
└────┴────┴────┴────┴────┘
     ↑
   Memory Address: 0x1000
```

### Why Contiguous Memory?

When you declare an array, the computer **reserves continuous blocks of memory**:

```
If each integer is 4 bytes:

Array starts at address 0x1000:
  Element 0: Address 0x1000 (value: 10)
  Element 1: Address 0x1004 (value: 20)    [0x1000 + 4]
  Element 2: Address 0x1008 (value: 30)    [0x1000 + 8]
  Element 3: Address 0x100C (value: 40)    [0x1000 + 12]
  Element 4: Address 0x1010 (value: 50)    [0x1000 + 16]

To access arr[2]:
  Calculate address = base_address + (index × element_size)
  = 0x1000 + (2 × 4) = 0x1008
  Then fetch value at that address: 30

This is O(1) because it's just arithmetic!
```

### Why Do Insertions Take O(n)?

```
Original Array: [10, 20, 30, 40, 50]

Insert 25 at index 1:

Step 1: Need space at index 1
        10 | [empty] 20 | 30 | 40 | 50

Step 2: Shift all elements right
        10 | 20 | 30 | 40 | 50 | ?
        ↑ SHIFT ALL TO THE RIGHT ↑
        10 | 20 | 30 | 40 | 50

Step 3: Now insert 25
        10 | 25 | 20 | 30 | 40 | 50

Why O(n)? You must shift every element after insertion point.
```

### Why Do Deletions Take O(n)?

```
Original: [10, 20, 30, 40, 50]

Delete element at index 1 (value 20):

Step 1: Remove element
        10 | [empty] | 30 | 40 | 50

Step 2: Shift left to fill gap
        10 | 30 | 40 | 50 | [removed]
        ↑ SHIFT ALL LEFT ↑

Result: [10, 30, 40, 50]

Why O(n)? You must shift every element after deletion point.
```

---

## Core Theory

### Arrays in Memory

**Static Arrays (Fixed Size):**
```
int arr[5];  // Fixed 5 elements
// Memory allocated at compile time
// Size cannot change
// Rarely used in interviews (we use dynamic arrays)
```

**Dynamic Arrays (Flexible Size):**
```
int[] arr = new int[5];  // Java
arr = [1, 2, 3]          # Python
// Memory allocated at runtime
// Size can be managed
```

### Key Properties

1. **Zero-Indexed:** First element is at index 0
2. **Contiguous:** Elements are stored next to each other
3. **Random Access:** Any element in O(1) time
4. **Fixed Size (mostly):** Can't dynamically resize (ArrayList does this for us)
5. **Homogeneous:** All elements have the same type

### Common Operations

```
1. ACCESS arr[i]           → Get element at index i
2. INSERT arr[i] = value   → Set element at index i
3. SEARCH value in arr     → Find if value exists
4. DELETE arr[i]           → Remove element
```

### Array Indexing Rules

```
Array with 5 elements: indices are 0, 1, 2, 3, 4
                      Valid indices: 0 to length-1

Accessing arr[5]       → Out of bounds! Error!
Accessing arr[-1]      → Out of bounds! Error!
Accessing arr[length]  → Out of bounds! Error!
```

---

## Java Implementation

### Basics: Creating and Accessing

```java
// 1. DECLARE an array
int[] arr;                    // Just a reference, not created yet

// 2. CREATE an array of size 5
arr = new int[5];            // Allocate memory for 5 integers
                              // All initialized to 0 by default

// 3. INITIALIZE with values
arr = new int[]{10, 20, 30, 40, 50};

// 4. COMBINED declaration and initialization
int[] arr2 = {1, 2, 3, 4, 5};

// 5. ACCESS elements
int first = arr[0];           // 10
int last = arr[4];            // 50
int length = arr.length;      // 5 (note: .length is a property, not method)

// 6. MODIFY elements
arr[0] = 100;                 // Change first element to 100

// 7. ITERATE
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// 8. ITERATE (enhanced for loop)
for (int num : arr) {
    System.out.println(num);
}
```

### Dynamic Arrays: ArrayList

Since arrays have fixed size, Java provides **ArrayList** for dynamic arrays:

```java
import java.util.ArrayList;

// CREATE an ArrayList
ArrayList<Integer> list = new ArrayList<>();

// ADD elements (grows automatically)
list.add(10);                 // O(1) amortized
list.add(20);
list.add(30);
// list: [10, 20, 30]

// ACCESS element
int first = list.get(0);      // 10

// MODIFY element
list.set(0, 100);             // O(1)
// list: [100, 20, 30]

// SIZE
int size = list.size();       // 3

// REMOVE element
list.remove(1);               // O(n) - removes 20
// list: [100, 30]

// CHECK if contains
if (list.contains(100)) {
    System.out.println("Found!");
}

// ITERATE
for (int num : list) {
    System.out.println(num);
}

// CONVERT to array
Integer[] array = list.toArray(new Integer[0]);
```

### Common Patterns

**Pattern 1: Find Maximum**
```java
int[] arr = {5, 2, 8, 1, 9};
int max = Integer.MIN_VALUE;  // Start with smallest possible

for (int num : arr) {
    if (num > max) {
        max = num;
    }
}
System.out.println(max);     // 9
```

**Pattern 2: Find Sum**
```java
int[] arr = {1, 2, 3, 4, 5};
int sum = 0;

for (int num : arr) {
    sum += num;
}
System.out.println(sum);     // 15
```

**Pattern 3: Find if Value Exists**
```java
int[] arr = {5, 2, 8, 1, 9};
int target = 8;
boolean found = false;

for (int num : arr) {
    if (num == target) {
        found = true;
        break;               // Exit early when found
    }
}
System.out.println(found);   // true
```

**Pattern 4: Reverse Array**
```java
int[] arr = {1, 2, 3, 4, 5};
int left = 0, right = arr.length - 1;

while (left < right) {
    // Swap
    int temp = arr[left];
    arr[left] = arr[right];
    arr[right] = temp;
    
    left++;
    right--;
}
// Result: [5, 4, 3, 2, 1]
```

---

## Python Implementation

### Basics

```python
# 1. CREATE a list (Python's dynamic array)
arr = [10, 20, 30, 40, 50]

# 2. ACCESS
first = arr[0]               # 10
last = arr[-1]               # 50 (negative indexing)
length = len(arr)            # 5

# 3. MODIFY
arr[0] = 100                 # [100, 20, 30, 40, 50]

# 4. ITERATE
for num in arr:
    print(num)

# 5. SLICE (get part of array)
subarray = arr[1:4]          # [20, 30, 40] (indices 1, 2, 3)
first_three = arr[:3]        # [100, 20, 30]
last_two = arr[-2:]          # [40, 50]
reversed_arr = arr[::-1]     # [50, 40, 30, 20, 100]

# 6. MANIPULATE
arr.append(60)               # Add to end: [100, 20, 30, 40, 50, 60]
arr.insert(0, 5)             # Insert at index 0: [5, 100, 20, ...]
arr.remove(20)               # Remove first occurrence of 20
popped = arr.pop()           # Remove and return last element

# 7. CHECK
if 30 in arr:
    print("Found!")          # O(n) search

# 8. SORT
arr.sort()                   # Sort in-place
arr_sorted = sorted(arr)     # Create sorted copy

# 9. REVERSE
arr.reverse()                # Reverse in-place
```

### Common Patterns in Python

**Pattern 1: Find Maximum**
```python
arr = [5, 2, 8, 1, 9]
max_val = max(arr)           # 9
# Or manually:
max_val = arr[0]
for num in arr:
    if num > max_val:
        max_val = num
```

**Pattern 2: Sum**
```python
arr = [1, 2, 3, 4, 5]
total = sum(arr)             # 15
```

**Pattern 3: List Comprehension**
```python
arr = [1, 2, 3, 4, 5]

# Double each element
doubled = [x * 2 for x in arr]  # [2, 4, 6, 8, 10]

# Filter even numbers
evens = [x for x in arr if x % 2 == 0]  # [2, 4]

# Create range
range_arr = list(range(5))   # [0, 1, 2, 3, 4]
```

**Pattern 4: Zip (Combine Arrays)**
```python
arr1 = [1, 2, 3]
arr2 = ['a', 'b', 'c']

# Combine into pairs
pairs = list(zip(arr1, arr2))  # [(1, 'a'), (2, 'b'), (3, 'c')]

for num, letter in pairs:
    print(num, letter)
```

---

## Internal Working

### How ArrayList Works (Java)

ArrayList internally uses a fixed-size array and grows it when needed:

```
Step 1: Create ArrayList
arr = new ArrayList<>()
Internal array: [] (capacity 10 by default)
Size: 0

Step 2: Add 15 elements
arr.add(1), arr.add(2), ..., arr.add(15)

Internally:
Step 2a: Add elements 1-10
  capacity: 10
  size: 10
  [1][2][3][4][5][6][7][8][9][10][][][][][]

Step 2b: Try to add 11th element
  capacity is full!
  Create new array with capacity 15 (usually 1.5x old capacity)
  Copy all elements: O(n)
  [1][2][3][4][5][6][7][8][9][10][11][][][][]
  
Step 2c: Continue adding
  [1][2][3][4][5][6][7][8][9][10][11][12][13][14][15]

Capacity growth: 10 → 15 → 22 → 33 → ...
```

**Amortized Time Complexity of Add:**

Even though growing takes O(n), we don't grow often:
- 10 adds: no growth
- 11-15 adds: 1 growth
- 16-22 adds: 1 growth (assuming 1.5x growth)

Total: ~1.5n operations for n adds → **O(1) amortized**

### Memory Layout

```
Array in Memory:
[10] [20] [30] [40] [50]
0x1000 0x1004 0x1008 0x100C 0x1010

Accessing arr[2]:
  Address = 0x1000 + (2 × 4 bytes) = 0x1008
  Value at 0x1008 = 30
  Time: O(1) - just arithmetic

Cache Efficiency:
  CPUs cache nearby memory
  Since array is contiguous, arr[0] and arr[1] are in same cache line
  Accessing sequentially is very fast (cache hit)
  
This is why arrays are often faster than linked lists for sequential access!
```

### Why Insert is O(n)

```
Insert 25 at index 2 in array [10, 20, 30, 40, 50]:

1. Find position: O(1)
2. Shift elements right: O(n)
   [10][20][X][30][40][50]
          ↓ shift right
   [10][20][30][40][50][X]
       
3. Insert value: O(1)
   [10][20][25][30][40][50]

Total: O(n) because step 2 takes O(n) time
```

---

## Time Complexity Analysis

### Why Each Operation Has Its Complexity

| Operation | Java | Python | Why? |
|-----------|------|--------|------|
| Access arr[i] | O(1) | O(1) | Direct memory lookup: address = base + (i × size) |
| Append | O(1) amortized | O(1) amortized | Doubling capacity means we rarely grow |
| Insert at index | O(n) | O(n) | Must shift all elements after index |
| Delete at index | O(n) | O(n) | Must shift all elements after index |
| Search | O(n) | O(n) | Linear scan (unless sorted + binary search) |
| Sort | O(n log n) | O(n log n) | Comparison-based sorting |
| Reverse | O(n) | O(n) | Visit each element |

### Space Complexity

**Array of n elements:**
- Space: O(n) - stores n elements
- Cannot be reduced (except by creating new array)

**ArrayList internal:**
- Capacity ≈ 1.5n (on average)
- Space: O(n) - still linear

---

## Interview Questions & Answers

### Q1: What's the difference between Array and ArrayList?

**Answer:**
```
Array:
- Fixed size, declared at creation
- Size never changes
- Slightly faster (no growth overhead)
- Used when size is known and won't change

ArrayList:
- Dynamic size, grows automatically
- Can add/remove elements
- Slight overhead for growth management
- Used when size varies

In interviews, use ArrayList (more flexible).
```

### Q2: Why is accessing an array element O(1)?

**Answer:**
```
Because arrays store elements in contiguous memory:
  arr = [10, 20, 30, 40, 50]
  
To access arr[3]:
  address = base_address + (3 × element_size)
  = 0x1000 + (3 × 4)
  = 0x100C
  
This is just arithmetic—no loop, no search. Always takes the same time.
```

### Q3: Why is insert O(n)?

**Answer:**
```
When you insert at index i:
  1. Elements before index i stay in place
  2. Elements at index i and after must move right to make space
  3. Worst case: insert at index 0 (must shift all n elements)
  
Example: [10, 20, 30] → insert 15 at index 0
  [10, 20, 30] must become [_, 10, 20, 30]
  Shift: 10→pos1, 20→pos2, 30→pos3
  Then insert: [15, 10, 20, 30]
  
This shifting takes O(n) time in worst case.
```

### Q4: Is searching an array O(n) always?

**Answer:**
```
For unsorted array: Yes, O(n)
  Must check each element sequentially
  
For sorted array: No, can be O(log n)
  Use binary search:
  - Check middle element
  - If target > middle, search right half
  - If target < middle, search left half
  - Halve the search space each time
  - Takes log₂(n) steps
```

### Q5: What happens when ArrayList capacity is exceeded?

**Answer:**
```
When you add to a full ArrayList:
  1. Current capacity is exceeded
  2. New internal array created (usually 1.5× or 2× size)
  3. All elements copied: O(n) operation
  4. Old array discarded
  5. New element added
  
But this is amortized O(1) because:
  - We have 10 cheap operations per 1 expensive operation
  - Total: ~1.5n operations for n adds
  - Average: 1.5n/n = O(1) per operation
```

---

## Common Mistakes

### ❌ Mistake 1: Off-by-One Errors

```java
// WRONG
int[] arr = {1, 2, 3, 4, 5};  // length 5, valid indices 0-4
for (int i = 0; i <= arr.length; i++) {
    System.out.println(arr[i]);  // ArrayIndexOutOfBoundsException when i=5
}

// CORRECT
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}
```

### ❌ Mistake 2: Not Handling Empty Arrays

```java
// WRONG
int[] arr = {};  // Empty array
int max = arr[0];  // ArrayIndexOutOfBoundsException

// CORRECT
if (arr.length == 0) {
    return -1;  // or handle empty case
}
int max = arr[0];
```

### ❌ Mistake 3: Modifying While Iterating

```java
// WRONG
ArrayList<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
for (int num : list) {
    if (num % 2 == 0) {
        list.remove((Integer) num);  // ConcurrentModificationException
    }
}

// CORRECT - Create new list
ArrayList<Integer> result = new ArrayList<>();
for (int num : list) {
    if (num % 2 != 0) {
        result.add(num);
    }
}
list = result;
```

### ❌ Mistake 4: Not Considering Space

```java
// INEFFICIENT
int[] arr = {1, 2, 3, 4, 5};
ArrayList<Integer> copy = new ArrayList<>();
for (int num : arr) {
    copy.add(num);  // Creates many ArrayList growths
}

// BETTER
ArrayList<Integer> copy = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
// Or specify capacity to avoid growth
ArrayList<Integer> copy = new ArrayList<>(arr.length);
```

### ❌ Mistake 5: Confused About Pass-by-Reference

```java
// This modifies the original array
int[] arr = {1, 2, 3};
modifyArray(arr);
// arr is changed because arrays are passed by reference

void modifyArray(int[] a) {
    a[0] = 999;  // Modifies original array
}
```

---

## Real Interview Traps

### Trap 1: Java Pass-by-Value Confusion

```java
ArrayList<Integer> list = new ArrayList<>();
list.add(10);

printList(list);

void printList(ArrayList<Integer> l) {
    l.add(20);  // This modifies the original list!
}

// After calling printList, the original list has [10, 20]
// Because ArrayList is an object, passed by reference
```

### Trap 2: Integer Wrapper Cache

```java
ArrayList<Integer> list1 = new ArrayList<>();
list1.add(128);
list1.add(128);

if (list1.get(0) == list1.get(1)) {  // True? False?
    // False! They are different objects
    // Java caches -128 to 127 for Integer, but 128 is not cached
}

if (list1.get(0).equals(list1.get(1))) {  // True
    // Use .equals() for value comparison
}
```

### Trap 3: Array Initialization

```java
int[] arr = new int[5];
// All elements initialized to 0 by default
System.out.println(arr[0]);  // 0 (not garbage value!)

boolean[] boolArr = new boolean[5];
// All elements initialized to false by default
System.out.println(boolArr[0]);  // false

Object[] objArr = new Object[5];
// All elements initialized to null by default
System.out.println(objArr[0]);  // null
```

### Trap 4: ArrayList Remove Confusion

```java
ArrayList<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));

list.remove(2);  // Which overload is called?
// This calls remove(int index) - removes element at index 2 (value 3)

list.remove(Integer.valueOf(2));  // Removes element with value 2
```

---

## Real World Applications

### 1. Google Search Results
```
Query: "machine learning"
Result: [webpage1, webpage2, webpage3, ...]

Array of URLs, ranked by relevance.
Users can:
- Scroll through (sequential access)
- Jump to page 5 (random access, array indexing)
```

### 2. Leaderboard in Games
```
Leaderboard: [
  {rank: 1, player: "Alice", score: 10000},
  {rank: 2, player: "Bob", score: 9500},
  {rank: 3, player: "Charlie", score: 9000}
]

Access top 10: arr[0:10] - O(1) with slicing
Find my rank: linear search O(n)
```

### 3. Image Processing
```
Image: 1920×1080 pixels
Stored as: array of arrays (2D array)
pixels[y][x] = RGB value

Fast pixel access for filters, blur, etc.
```

### 4. Stock Prices
```
prices = [100, 102, 101, 105, 103, 108]

Find max price: max(arr) - O(n)
Calculate trend: compare arr[i] with arr[i+1] - O(n)
Binary search for specific price: O(log n)
```

---

## Common LeetCode Problems

### Easy
1. **Two Sum** - Find indices of two numbers that sum to target
   - Hints: Use HashMap for O(n)
   - Why: Classic array + hash pattern

2. **Best Time to Buy and Sell Stock** - Find max profit
   - Hints: Track minimum price so far
   - Why: Single pass, track state pattern

3. **Contains Duplicate** - Check if array has duplicates
   - Hints: Use HashSet
   - Why: O(1) duplicate checking

### Medium
1. **Product of Array Except Self** - Get products except current element
   - Hints: Use prefix and suffix products
   - Why: Classic two-pointer/prefix pattern

2. **3Sum** - Find three numbers that sum to zero
   - Hints: Sort array, use two pointers
   - Why: Optimization from brute force O(n³) to O(n²)

3. **Longest Consecutive** - Find longest consecutive elements
   - Hints: Use HashSet for O(1) lookup
   - Why: Smart use of hash table

### Hard
1. **First Missing Positive** - Find smallest missing positive integer
   - Hints: Cycle sort or use array as hash
   - Why: Space optimization challenge

2. **Trapping Rain Water** - Calculate trapped water between bars
   - Hints: Two pointers or dynamic programming
   - Why: Complex spatial reasoning

---

## Pattern Recognition

### When to Use Arrays

**Use Array when you see:**
- "Find maximum/minimum"
- "Sum all elements"
- "Check if exists"
- "Sort and process"
- "Two/three sum problems"
- "Subarray problems" (contiguous elements)
- "Product of elements"

**Decision Framework:**
```
Do you need to store multiple elements?
  YES → Array or ArrayList
    ├─ Fixed size known? → Array
    └─ Size varies? → ArrayList
  
Do you need fast random access? → YES → Array
Do you need fast insertion/deletion?
  └─ Mostly at beginning? → LinkedList (not array)
  └─ Mostly at end? → Array (append is O(1))
```

---

## Comparison with Similar Structures

### Array vs Linked List

| Operation | Array | Linked List |
|-----------|-------|-------------|
| Random Access | O(1) | O(n) |
| Insert at End | O(1) amortized | O(1) with tail pointer |
| Insert at Start | O(n) | O(1) |
| Delete at Start | O(n) | O(1) |
| Delete at End | O(1) | O(n) unless doubly-linked |
| Memory | Contiguous | Scattered + pointers |
| Cache Friendly | YES | NO |

**Use Array when:** You need fast random access
**Use Linked List when:** You need fast insertions/deletions at the front

### Array vs HashMap

| Operation | Array | HashMap |
|-----------|-------|----------|
| Insert | O(n) at arbitrary index | O(1) avg |
| Delete | O(n) at arbitrary index | O(1) avg |
| Search by Value | O(n) | O(1) avg if using value as key |
| Search by Index | O(1) | N/A |
| Space | O(n) | O(n) + overhead |
| Order | Maintains order | No order guarantee |

**Use Array when:** You need ordered storage and random access by index
**Use HashMap when:** You need fast lookups by key

---

## 5-Minute Revision

### Array Basics
```
Array = ordered collection with fixed indices

Access arr[i]:        O(1)
Insert at index:      O(n)
Delete at index:      O(n)
Append to end:        O(1) amortized
Search:               O(n) unsorted, O(log n) if sorted

Why O(1) access?
  address = base + (index × element_size)
  Just arithmetic, no loop

Why O(n) insert/delete?
  Must shift all elements after index

ArrayList in Java:
  - Dynamic array
  - Grows when capacity exceeded
  - ArrayList<Type> list = new ArrayList<>()
  
List in Python:
  - Python's dynamic array
  - arr = [1, 2, 3]
  - Built-in methods: append(), remove(), pop()
```

### Common Patterns
```
1. Max/Min → single pass, track variable
2. Sum → accumulate variable
3. Search → loop until found
4. Two Sum → HashMap for O(n)
5. Subarray → sliding window or two pointers
6. Prefix/Suffix → precompute for optimization
```

### Interview Checklist
```
☐ Consider empty array edge case
☐ Check array bounds (0 to length-1)
☐ Clarify: is array sorted?
☐ Clarify: can I modify array?
☐ Think about time/space complexity
☐ Consider if sorting helps
☐ For lookups, consider HashMap
```

---

## Practice Problems

Before moving to next data structure, solve:

1. Two Sum (Easy)
2. Best Time to Buy and Sell Stock (Easy)
3. Contains Duplicate (Easy)
4. Valid Anagram (Easy)
5. Product of Array Except Self (Medium)
6. 3Sum (Medium)
7. Longest Consecutive (Medium)

If you can solve 5/7 in < 30 minutes each, you're ready for Linked Lists.

---

Next: Read **02_LINKED_LIST.md**
