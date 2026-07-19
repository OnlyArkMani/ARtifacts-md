# 06 — HashMap & HashSet: O(1) Lookup Magic

## Introduction

### What Is a HashMap?

A **HashMap** is a data structure that implements a **hash table** for **key-value storage**. It uses a **hash function** to map keys to indices, enabling:
- **O(1) average lookup** by key
- **Dynamic resizing** as size grows
- **Flexible data storage**

### Real-World Analogy

**Dictionary:**
- Word (key) → Definition (value)
- You don't flip through every page; you can jump to "H" for "hello"
- Hash function = alphabet organization

**Locker Room:**
- Locker number (key) → Your belongings (value)
- You know locker #42 directly
- No need to search locker #1, #2, #3...

### Why Does It Matter?

**The Problem HashMap Solves:**
```
Find if value 5 exists in array [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

Array search: O(n) - check each element
HashMap lookup: O(1) - direct access

For 1 million elements:
- Array: 1 million comparisons
- HashMap: 1 lookup
```

HashMaps power:
1. **Frequency counting** (most common interview pattern)
2. **Two-sum problems** (find pairs)
3. **Duplicate detection** (check if seen before)
4. **Caching** (store computed results)
5. **Grouping** (group elements by property)

### Where Is It Used?

**Google Search:**
- Index all webpages
- Search keyword → find matching pages instantly

**Facebook User Database:**
- Username/email → User profile instantly

**Cache Systems:**
- Frequently accessed data stored in hash table
- O(1) retrieval

**DNA Sequencing:**
- Find patterns in genetic code
- Hash table stores known patterns

---

## Intuition Section

### How Hash Function Works

```
Hash Function: Maps any key to an array index

Example: Store user ages by username

Key: "alice"
Hash Function: hash("alice") = 5
Store at index 5: ages[5] = 25

Later, lookup "alice":
hash("alice") = 5 (same function)
Return ages[5] = 25

Magic: Same input always gives same output!
Time: O(1) - just calculate hash and look up
```

### Simple Hash Function

```
For strings, sum of character codes mod array size:

"alice" = a(97) + l(108) + i(105) + c(99) + e(101) = 510
hash("alice") = 510 % 10 = 0

"bob" = b(98) + o(111) + b(98) = 307
hash("bob") = 307 % 10 = 7

Array (size 10):
Index 0: alice -> 25
Index 7: bob -> 30

Lookup: hash("alice") = 0, return ages[0] = 25 ✓
Time: O(1)
```

### The Collision Problem

```
What if two different keys hash to same index?

"alice" -> index 0
"brad" -> index 0 (collision!)

Both can't occupy index 0

Solution 1: Chaining
Index 0: ["alice" -> 25] -> ["brad" -> 28]

Solution 2: Open Addressing
Find next available index
Index 0: taken
Index 1: empty, put here
```

### Load Factor & Resizing

```
Load Factor = number of elements / array size

Example:
- Array size: 10
- Elements stored: 7
- Load Factor: 0.7

When load factor exceeds 0.75:
- Create new array (double size = 20)
- Rehash all elements
- This is why insert is O(1) AMORTIZED
```

---

## Core Theory

### Hash Table Components

1. **Hash Function:** Key → Array index
2. **Array:** Store key-value pairs
3. **Collision Resolution:** Handle hash conflicts
4. **Resizing:** Grow when full

### Collision Resolution Methods

**Chaining (Used by Java HashMap):**
```
Each array index contains a linked list
Collisions stored in same list

Array (size 10):
Index 0: ["alice" -> 25] -> ["brad" -> 28]
Index 5: ["charlie" -> 35]
Index 7: ["diana" -> 32]

Lookup: Hash to index, then search chain
```

**Open Addressing:**
```
Find next empty slot

Array (size 10):
Index 0: "alice" -> 25
Index 1: "brad" -> 28  (collision, moved here)
Index 2: empty
Index 3: "charlie" -> 35
```

### Time Complexity Analysis

**Average Case:** Load factor α = n/m (elements/size)
- Search: O(1 + α)
- If we keep α < 0.75 via resizing: O(1)

**Worst Case:** All keys hash to same index
- Search: O(n)
- Rarely happens with good hash function

---

## Java Implementation

### HashMap Basics

```java
import java.util.HashMap;
import java.util.Map;

// Create HashMap
Map<String, Integer> map = new HashMap<>();

// INSERT: O(1) average
map.put("alice", 25);
map.put("bob", 30);
map.put("charlie", 35);

// LOOKUP: O(1) average
int age = map.get("alice");  // 25
int notFound = map.get("david");  // null

// GET with default
int ageOrZero = map.getOrDefault("david", 0);  // 0

// CHECK if key exists
if (map.containsKey("alice")) {
    System.out.println("Found");
}

// DELETE: O(1) average
map.remove("bob");

// SIZE
int size = map.size();

// ITERATE
for (String key : map.keySet()) {
    System.out.println(key + ": " + map.get(key));
}

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + ": " + entry.getValue());
}
```

### HashSet Basics

```java
import java.util.HashSet;
import java.util.Set;

// Create HashSet (stores only keys, no values)
Set<String> names = new HashSet<>();

// INSERT: O(1) average
names.add("alice");
names.add("bob");
names.add("charlie");
names.add("alice");  // Duplicate, not added

// SIZE
int size = names.size();  // 3 (alice counted once)

// CHECK if contains: O(1) average
if (names.contains("bob")) {
    System.out.println("Found");
}

// REMOVE: O(1) average
names.remove("bob");

// ITERATE
for (String name : names) {
    System.out.println(name);
}

// REMOVE duplicates from list
List<Integer> list = Arrays.asList(1, 2, 2, 3, 3, 3);
Set<Integer> unique = new HashSet<>(list);
List<Integer> noDupes = new ArrayList<>(unique);
```

### Common Pattern 1: Frequency Counter

```java
public Map<Character, Integer> countFrequency(String s) {
    Map<Character, Integer> freq = new HashMap<>();
    
    for (char c : s.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    
    return freq;
}

// Usage
Map<Character, Integer> freq = countFrequency("hello");
// {h: 1, e: 1, l: 2, o: 1}
```

### Common Pattern 2: Two Sum

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        
        seen.put(nums[i], i);
    }
    
    return new int[]{-1, -1};
}

// Time: O(n), Space: O(n)
```

### Common Pattern 3: Top K Frequent

```java
public int[] topKFrequent(int[] nums, int k) {
    // Count frequencies
    Map<Integer, Integer> freq = new HashMap<>();
    for (int num : nums) {
        freq.put(num, freq.getOrDefault(num, 0) + 1);
    }
    
    // Use heap for top k
    PriorityQueue<Integer> heap = new PriorityQueue<>(
        (a, b) -> freq.get(a) - freq.get(b)
    );
    
    for (int num : freq.keySet()) {
        heap.offer(num);
        if (heap.size() > k) {
            heap.poll();
        }
    }
    
    int[] result = new int[k];
    int i = k - 1;
    while (!heap.isEmpty()) {
        result[i--] = heap.poll();
    }
    return result;
}

// Time: O(n log k), Space: O(n)
```

---

## Python Implementation

### Dict & Set

```python
# Create dictionary (HashMap)
user_ages = {}

# INSERT: O(1) average
user_ages["alice"] = 25
user_ages["bob"] = 30
user_ages["charlie"] = 35

# LOOKUP: O(1) average
age = user_ages["alice"]  # 25
age_or_default = user_ages.get("david", 0)  # 0

# CHECK if key exists
if "alice" in user_ages:
    print("Found")

# DELETE: O(1) average
del user_ages["bob"]
user_ages.pop("charlie")

# SIZE
size = len(user_ages)

# ITERATE
for key in user_ages:
    print(f"{key}: {user_ages[key]}")

for key, value in user_ages.items():
    print(f"{key}: {value}")

# Get all keys/values
keys = list(user_ages.keys())
values = list(user_ages.values())
```

### Set

```python
# Create set (no duplicates)
names = {"alice", "bob", "charlie"}

# ADD: O(1) average
names.add("diana")
names.add("alice")  # Already exists, not added

# SIZE
size = len(names)  # 4

# CHECK if contains: O(1) average
if "bob" in names:
    print("Found")

# REMOVE: O(1) average
names.remove("bob")
names.discard("bob")  # No error if not found

# ITERATE
for name in names:
    print(name)

# Remove duplicates from list
lst = [1, 2, 2, 3, 3, 3]
unique = list(set(lst))  # [1, 2, 3] (order not guaranteed)
```

### Counter (Frequency Counter)

```python
from collections import Counter

# Count frequencies
freq = Counter("hello")
# Counter({'l': 2, 'h': 1, 'e': 1, 'o': 1})

freq["l"]  # 2
freq["z"]  # 0 (missing key returns 0, not error)

# Most common
freq.most_common(2)  # [('l', 2), ('h', 1)]

# For list
nums = [1, 1, 2, 2, 2, 3]
freq = Counter(nums)
freq.most_common(1)  # [(2, 3)]
```

### Common Pattern: Two Sum in Python

```python
def twoSum(nums, target):
    seen = {}
    
    for i, num in enumerate(nums):
        complement = target - num
        
        if complement in seen:
            return [seen[complement], i]
        
        seen[num] = i
    
    return [-1, -1]
```

---

## Internal Working

### Hash Function in Java

```java
// Simplified version of how Java hashes strings
public int hashCode(String s) {
    int hash = 0;
    for (char c : s.toCharArray()) {
        hash = hash * 31 + c;  // 31 is prime
    }
    return hash;
}

// Then mapped to array index:
int index = hash % array.length;
```

### Resizing (Rehashing)

```
Original HashMap (size 16, load factor 0.75):
Element count threshold: 0.75 * 16 = 12

When 13th element added:
1. Create new array (size 32)
2. Iterate all elements in old array
3. Recalculate hash for each: hash % 32
4. Place in new array
5. Point to new array

Why O(1) amortized?
The growth is geometric (2x each time)
So rarely resizing happens
Total operations for n insertions ≈ 2n = O(n) amortized
```

### Collision Resolution with Chaining

```
HashMap internal:
Array of linked lists (buckets)

Array (size 16):
Index 0: null
Index 1: ["bob" -> 30]
Index 5: ["alice" -> 25] -> ["carol" -> 32]  (collision!)
Index 7: null
...

Lookup "carol":
1. hash("carol") % 16 = 5
2. Go to bucket 5
3. Traverse linked list
4. Find ["carol" -> 32]
Time: O(1 + chain_length)
If collisions rare: O(1)
```

---

## Time Complexity Table

| Operation | HashMap | HashSet | Why? |
|-----------|---------|---------|------|
| Insert | O(1) avg | O(1) avg | Hash to index, store |
| Lookup | O(1) avg | O(1) avg | Hash to index, retrieve |
| Delete | O(1) avg | O(1) avg | Hash to index, remove |
| Contains | N/A | O(1) avg | Hash to index, check |
| Iterate | O(n) | O(n) | Visit all elements |
| Worst case | O(n) | O(n) | All hash to same bucket |

**Space:** O(n) for n elements + overhead

---

## Interview Questions & Answers

### Q1: Two Sum

**Question:** Find indices of two numbers that sum to target

**Answer:**
```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (seen.containsKey(complement)) {
            return new int[]{seen.get(complement), i};
        }
        seen.put(nums[i], i);
    }
    return new int[]{-1, -1};
}

Time: O(n), Space: O(n)
```

### Q2: Valid Anagram

**Question:** Check if two strings are anagrams

**Answer:**
```java
public boolean isAnagram(String s, String t) {
    if (s.length() != t.length()) return false;
    
    Map<Character, Integer> freq = new HashMap<>();
    
    for (char c : s.toCharArray()) {
        freq.put(c, freq.getOrDefault(c, 0) + 1);
    }
    
    for (char c : t.toCharArray()) {
        if (!freq.containsKey(c)) return false;
        freq.put(c, freq.get(c) - 1);
        if (freq.get(c) < 0) return false;
    }
    
    return true;
}

Time: O(n), Space: O(1) for English letters only
```

### Q3: Contains Duplicate

**Question:** Check if array has duplicates

**Answer:**
```java
public boolean containsDuplicate(int[] nums) {
    Set<Integer> seen = new HashSet<>();
    
    for (int num : nums) {
        if (seen.contains(num)) {
            return true;
        }
        seen.add(num);
    }
    
    return false;
}

Time: O(n), Space: O(n)
```

---

## Common Mistakes

### ❌ Mistake 1: Null Key/Value Confusion

```java
// WRONG - HashMap allows null key and null values
Map<String, Integer> map = new HashMap<>();
map.put(null, 1);
int val = map.get(null);  // 1

// But HashTable does NOT allow null
Hashtable<String, Integer> ht = new Hashtable<>();
ht.put(null, 1);  // NullPointerException
```

### ❌ Mistake 2: ConcurrentModificationException

```java
// WRONG - Modifying while iterating
Map<String, Integer> map = new HashMap<>();
map.put("a", 1);
map.put("b", 2);

for (String key : map.keySet()) {
    if (key.equals("a")) {
        map.remove(key);  // ConcurrentModificationException
    }
}

// CORRECT - Collect first, then remove
Set<String> toRemove = new HashSet<>();
for (String key : map.keySet()) {
    if (key.equals("a")) {
        toRemove.add(key);
    }
}
for (String key : toRemove) {
    map.remove(key);
}
```

### ❌ Mistake 3: Comparing with ==

```java
// WRONG
Integer a = 128;
Integer b = 128;
if (a == b) {  // false! Different objects
    // ...
}

// CORRECT
if (a.equals(b)) {  // true
    // ...
}
```

---

## Real World Applications

**Frequency Counting:**
- "Find mode (most frequent element)"
- Count word frequencies in text

**Deduplication:**
- Remove duplicates from list
- Detect if username already exists

**Caching:**
- Store computed results
- Avoid recomputation

**Grouping:**
- Group people by age
- Group items by category

---

## Common LeetCode Problems

**Easy:**
1. Two Sum
2. Contains Duplicate
3. Valid Anagram

**Medium:**
1. Top K Frequent Elements
2. Group Anagrams
3. Longest Substring Without Repeating

**Hard:**
1. LRU Cache
2. All Possible Full Binary Trees

---

## Pattern Recognition

**Use HashMap when:**
- "Count frequencies"
- "Find if exists"
- "Find pairs/complement"
- "Group by property"
- "Cache results"

**Use HashSet when:**
- "Remove duplicates"
- "Check membership"
- "Track seen elements"

---

## 5-Minute Revision

```
HashMap = Hash function + Array + Collision handling

O(1) average because:
- Hash function maps key to index in O(1)
- Direct array access in O(1)
- Resizing keeps load factor < 0.75

Common patterns:
1. Frequency count
   freq = new HashMap()
   freq.put(key, freq.getOrDefault(key, 0) + 1)

2. Two pointer lookup
   seen = new HashMap()
   if (seen.containsKey(complement)) {
     return seen.get(complement)
   }

3. Group elements
   group = new HashMap()
   group.put(key, group.getOrDefault(key, new List()))
   group.get(key).add(element)

4. Deduplication
   unique = new HashSet()
   for (item) {
     if (!unique.contains(item)) {
       unique.add(item)
     }
   }

Collision handling:
- Chaining (Java HashMap)
- Open addressing (some implementations)
```

---

Next: Read **19_TWO_POINTERS.md**
