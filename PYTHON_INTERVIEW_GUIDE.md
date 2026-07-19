# PYTHON — Interview Essentials & Gotchas

**Target:** Write elegant, bug-free Python in interviews. This file covers syntax quirks, performance traps, built-in utilities, and patterns that let you solve faster than in Java or C++.

---

## Table of Contents

1. [Lists vs Tuples vs Sets](#1-lists-vs-tuples-vs-sets) — when to use each
2. [Dictionaries (Hashmaps)](#2-dictionaries-hashmaps) — the workhorse
3. [String Handling](#3-string-handling) — slicing, immutability, operations
4. [List Comprehensions & Generators](#4-list-comprehensions--generators) — elegant && fast
5. [Sorting & Comparators](#5-sorting--comparators) — lambda, custom key functions
6. [Common Library Methods](#6-common-library-methods) — `collections`, `heapq`, `itertools`
7. [Iteration & Slicing](#7-iteration--slicing) — Python idioms
8. [Recursion & Memoization](#8-recursion--memoization) — the `@lru_cache` decorator
9. [Type Hints & None](#9-type-hints--none) — defensive coding
10. [Performance Tips](#10-performance-tips) — avoid slow patterns
11. [Edge Cases Checklist](#11-edge-cases-checklist) — pre-submission scan
12. [Example: Two Pointers in Python](#12-example-two-pointers-in-python) — putting it together

---

## 1. Lists vs Tuples vs Sets

### Quick Reference

| Type | Syntax | Mutable | Hashable | Use Case | Time (avg) |
|------|--------|---------|----------|----------|------------|
| **List** | `[1, 2, 3]` | ✅ Yes | ❌ No | Dynamic collection, maintain order | O(1) append, O(n) insert |
| **Tuple** | `(1, 2, 3)` | ❌ No | ✅ Yes | Immutable, use as dict key | O(1) access |
| **Set** | `{1, 2, 3}` | ✅ Yes | ✅ No | No duplicates, fast membership | O(1) lookup, add, remove |
| **Dict** | `{1: 'a'}` | ✅ Yes | N/A | Key → value mapping | O(1) lookup |

### Lists

```python
list1 = [1, 2, 3]
list1.append(4)              # add to end: O(1) amortized
list1.extend([5, 6])         # add multiple: O(k)
list1.insert(0, 0)           # insert at index: O(n)
list1.remove(2)              # remove by value (first): O(n)
list1.pop()                  # remove + return last: O(1)
list1.pop(0)                 # remove + return at index: O(n) for index 0!
list1.index(3)               # find index: O(n)
list1.count(1)               # count occurrences: O(n)
len(list1)                   # O(1)

# In-place modification
list1[0] = 10                # O(1)
list1[1:3] = [20, 21]        # slice assignment

# Iteration
for x in list1:              # O(n)
    print(x)
for i, x in enumerate(list1): # index + value
    print(i, x)
```

### Tuples

```python
tuple1 = (1, 2, 3)           # immutable
tuple1[0]                    # 1, O(1) access
# tuple1[0] = 10            # TypeError — can't modify

# Use as dict key or in set (lists cannot)
frequency = {}
frequency[(1, 2)] = "seen"    # tuple as key OK
# frequency[[1, 2]] = "seen"  # TypeError — list unhashable

# Tuple unpacking
a, b, c = (1, 2, 3)
x, *rest = [1, 2, 3, 4]       # x=1, rest=[2,3,4]

# Single element tuple needs comma
single = (1,)                 # tuple
not_tuple = (1)               # just int 1
```

### Sets

```python
set1 = {1, 2, 3}             # unordered, no duplicates
set1.add(4)                  # O(1) avg
set1.remove(1)               # O(1) avg; KeyError if absent
set1.discard(1)              # O(1) avg; silent if absent
set1.pop()                    # remove arbitrary element
len(set1)                     # O(1)
1 in set1                     # O(1) avg — MUCH faster than list!
set1.clear()                 # empty the set

# Set operations
set1 & set2                   # intersection
set1 | set2                   # union
set1 - set2                   # difference
set1 ^ set2                   # symmetric difference
set1.issubset(set2)
set1.issuperset(set2)

# Remove duplicates from list
list1 = [1, 2, 2, 3, 3, 3]
list1 = list(set(list1))      # [1, 2, 3] (order not guaranteed)
list1 = list(dict.fromkeys(list1))  # [1, 2, 3] (preserves order in Python 3.7+)
```

---

## 2. Dictionaries (Hashmaps)

### Core Operations

```python
dict1 = {'a': 1, 'b': 2}

dict1['a']                   # 1, O(1) avg; KeyError if missing
dict1.get('a')               # 1, O(1) avg
dict1.get('c', 0)            # 0 (default if missing)
dict1.get('c')               # None (default)
dict1.setdefault('c', 3)     # 3; also sets dict1['c'] = 3

dict1['c'] = 3               # insert/update: O(1) avg
dict1.update({'d': 4})       # merge dicts

del dict1['a']               # delete: O(1) avg
dict1.pop('b')               # 2 (pop + return); KeyError if missing
dict1.pop('b', None)         # None (default if missing)

dict1.keys()                 # dict_keys object (iterable)
dict1.values()               # dict_values object
dict1.items()                # dict_items of (k, v) tuples
len(dict1)
dict1.clear()                # empty

# Membership
'a' in dict1                 # O(1) avg — checks keys
'a' in dict1.values()        # O(n) — checks values (slow!)
```

### Iteration Patterns

```python
# Iterate keys
for key in dict1:
    print(key, dict1[key])

# Iterate keys + values
for key, value in dict1.items():
    print(key, value)

# Default dict for counters
from collections import defaultdict
freq = defaultdict(int)      # missing keys default to 0
for char in "hello":
    freq[char] += 1          # no KeyError on first access
print(freq)                  # {'h': 1, 'e': 1, 'l': 2, 'o': 1}

# Counter class — even more convenient
from collections import Counter
freq = Counter("hello")
print(freq.most_common(2))   # [('l', 2), ('h', 1)] — top 2
print(freq['x'])             # 0 (missing key → 0, never KeyError)
```

### Dictionary Comprehension

```python
# Create dict from list
dict1 = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Conditional
dict2 = {x: x**2 for x in range(5) if x % 2 == 0}  # only evens

# From two lists
keys = ['a', 'b', 'c']
values = [1, 2, 3]
dict3 = {k: v for k, v in zip(keys, values)}  # {'a': 1, 'b': 2, 'c': 3}
```

---

## 3. String Handling

### Immutability & Operations

```python
s = "hello"
# Strings are immutable; every operation returns new string
s.upper()                    # "HELLO" — doesn't modify s
s = s.upper()                # now s = "HELLO"

# Concatenation (immutable, but efficient in Python 3)
s1 = "hello"
s2 = "world"
s = s1 + " " + s2            # "hello world"

# String methods
len(s)                       # 5
s[0]                         # 'h' — indexing is O(1)
s[1:4]                       # 'ell' — slicing
s[::-1]                       # "olleh" — reverse
s.find('l')                  # 2 (first occurrence; -1 if not found)
s.rfind('l')                 # 3 (last)
s.count('l')                 # 2 (occurrences)
s.startswith('he')           # True
s.endswith('lo')             # True
s.replace('l', 'x')          # "hexxo"
s.split(' ')                 # ['hello', 'world'] — split on delimiter
' '.join(['hello', 'world']) # "hello world" — join list into string
s.strip()                    # remove leading/trailing whitespace
s.lower()                    # lowercase
s.isdigit()                  # True if all digits
s.isalpha()                  # True if all letters
s.isalnum()                  # True if alphanumeric
```

### Slicing (Powerful in Python)

```python
s = "hello"

s[0]                         # 'h'
s[-1]                        # 'o' (last char)
s[1:4]                       # 'ell' (s[1], s[2], s[3]; excludes 4)
s[:3]                        # 'hel' (from start)
s[2:]                        # 'llo' (to end)
s[:]                         # "hello" (full copy)
s[::2]                       # 'hlo' (every 2nd char)
s[::-1]                       # "olleh" (reverse)

# Same logic for lists
list1 = [1, 2, 3, 4, 5]
list1[1:4]                   # [2, 3, 4]
list1[::-1]                  # [5, 4, 3, 2, 1] (reversed)
list1[::2]                   # [1, 3, 5] (every 2nd)
```

### String Formatting

```python
name, age = "Alice", 30

# f-string (modern, preferred)
print(f"Name: {name}, Age: {age}")  # "Name: Alice, Age: 30"
print(f"Square of 5: {5**2}")        # "Square of 5: 25"

# Old-style (still works)
print("Name: %s, Age: %d" % (name, age))
print("Name: {}, Age: {}".format(name, age))
```

### Convert String to Numbers

```python
int("123")                   # 123
int("123", 16)               # 291 (hexadecimal)
float("3.14")                # 3.14
str(123)                     # "123"

# Safe parsing
try:
    x = int("abc")
except ValueError:
    x = 0                    # fallback
```

---

## 4. List Comprehensions & Generators

### List Comprehensions (Fast, Readable)

```python
# Basic
list1 = [x**2 for x in range(5)]           # [0, 1, 4, 9, 16]

# With condition
list2 = [x for x in range(10) if x % 2 == 0]  # [0, 2, 4, 6, 8]

# Multiple conditions
list3 = [x for x in range(10) if x % 2 == 0 if x > 3]  # [4, 6, 8]

# Nested
list4 = [x*y for x in range(3) for y in range(3)]  # [0, 0, 0, 0, 1, 2, 0, 2, 4]

# String manipulation
words = ["hello", "world"]
caps = [w.upper() for w in words]          # ["HELLO", "WORLD"]

# Flatten nested list
nested = [[1, 2], [3, 4], [5, 6]]
flat = [x for row in nested for x in row]  # [1, 2, 3, 4, 5, 6]

# Compare to loop (verbose & slow)
list1 = []
for x in range(5):
    list1.append(x**2)      # less readable, slower
```

### Generators (Lazy Evaluation, Memory-Efficient)

```python
# Generator expression (like list comp, but lazy)
gen = (x**2 for x in range(1000000))  # NOT evaluated yet
print(next(gen))                       # 0 (compute one at a time)
print(next(gen))                       # 1

# Consume all
list1 = list(gen)                      # now evaluated

# Generator function
def countdown(n):
    while n > 0:
        yield n              # yields value without returning
        n -= 1

for val in countdown(3):     # 3, 2, 1
    print(val)

# Use in loops to save memory
for value in (x**2 for x in range(1000000)):
    if value > 100:
        break              # stop early — doesn't compute rest
```

---

## 5. Sorting & Comparators

### Basic Sorting

```python
list1 = [3, 1, 2]
list1.sort()                 # sorts in-place: [1, 2, 3]

sorted(list1)                # returns new sorted list (original unchanged)

list2 = [3, 1, 2]
sorted(list2, reverse=True)  # [3, 2, 1]
```

### Sort by Key Function

```python
words = ["zebra", "apple", "banana"]

# Sort by length
sorted(words, key=len)                     # ['apple', 'zebra', 'banana']

# Sort by length descending
sorted(words, key=len, reverse=True)       # ['banana', 'zebra', 'apple']

# Sort by custom function
sorted(words, key=lambda w: w[-1])         # sort by last char

# Sort by multiple criteria (tuple unpacking)
pairs = [("a", 2), ("b", 1), ("a", 1)]
sorted(pairs, key=lambda p: (p[0], p[1]))  # sorts by first elem, then second
# Result: [('a', 1), ('a', 2), ('b', 1)]
```

### Sort Objects

```python
class Person:
    def __init__(self, name, age):
        self.name = name
        self.age = age
    
    def __repr__(self):
        return f"Person({self.name}, {self.age})"

people = [Person("Alice", 30), Person("Bob", 25), Person("Charlie", 30)]

# Sort by age ascending
sorted(people, key=lambda p: p.age)

# Sort by age descending, name ascending (tiebreaker)
sorted(people, key=lambda p: (-p.age, p.name))
```

---

## 6. Common Library Methods

### `collections` Module

```python
from collections import Counter, defaultdict, deque

# Counter — frequency table on steroids
freq = Counter([1, 1, 2, 2, 2, 3])
print(freq)                  # Counter({2: 3, 1: 2, 3: 1})
freq.most_common(2)          # [(2, 3), (1, 2)] — top 2
freq['x']                    # 0 (missing key → 0, never KeyError)

# defaultdict — dict with automatic default values
from collections import defaultdict
fallback = defaultdict(list)  # missing keys default to []
fallback['a'].append(1)       # no KeyError
print(fallback)              # defaultdict(list, {'a': [1]})

# deque — efficient queue/stack from both ends
from collections import deque
q = deque([1, 2, 3])
q.append(4)                  # O(1) add right
q.appendleft(0)              # O(1) add left
q.pop()                       # O(1) remove right
q.popleft()                  # O(1) remove left
```

### `heapq` Module (Min-Heap)

```python
import heapq

heap = [5, 3, 7, 1]
heapq.heapify(heap)           # convert to heap in-place: O(n)
# heap is now [1, 3, 7, 5]

heapq.heappush(heap, 2)       # insert: O(log n)
smallest = heapq.heappop(heap)  # pop min: O(log n), returns 1

# Max-heap (negate for numbers)
max_heap = [-x for x in [5, 3, 7]]
heapq.heapify(max_heap)
largest = -heapq.heappop(max_heap)  # 7

# nlargest / nsmallest
heapq.nlargest(2, [1, 5, 3, 9, 2])  # [9, 5]
heapq.nsmallest(2, [1, 5, 3, 9, 2]) # [1, 2]

# Top K pattern
import heapq
nums = [1, 2, 3, 4, 5]
top_k = heapq.nlargest(3, nums)  # [5, 4, 3]
```

### `itertools` Module

```python
from itertools import combinations, permutations, product

# All pairs
list(combinations([1, 2, 3], 2))      # [(1,2), (1,3), (2,3)]

# All orderings
list(permutations([1, 2, 3]))         # [(1,2,3), (1,3,2), ...] 6 total

# Cartesian product
list(product([1, 2], ['a', 'b']))     # [(1,'a'), (1,'b'), (2,'a'), (2,'b')]

# Chain multiple iterables
from itertools import chain
list(chain([1, 2], [3, 4], [5]))      # [1, 2, 3, 4, 5]

# Infinite counter
from itertools import count
for i in count(1):                    # 1, 2, 3, ... (infinite)
    if i > 5: break
    print(i)
```

### `bisect` Module (Binary Search)

```python
import bisect

sorted_list = [1, 3, 3, 3, 7, 9]

# Find insertion point
bisect.bisect_left(sorted_list, 3)    # 1 (leftmost 3)
bisect.bisect_right(sorted_list, 3)   # 4 (rightmost 3)

# Insert and maintain order
bisect.insort(sorted_list, 5)         # insert 5 in-place
print(sorted_list)                    # [1, 3, 3, 3, 5, 7, 9]
```

---

## 7. Iteration & Slicing

### Enumerate (Index + Value)

```python
list1 = ['a', 'b', 'c']

for i, val in enumerate(list1):
    print(i, val)            # 0 a, 1 b, 2 c

# Start from custom index
for i, val in enumerate(list1, start=1):
    print(i, val)            # 1 a, 2 b, 3 c
```

### Zip (Pair Up Iterables)

```python
list1 = ['a', 'b', 'c']
list2 = [1, 2, 3, 4]

for x, y in zip(list1, list2):
    print(x, y)              # a 1, b 2, c 3 (stops at shortest)

# Create dict from pairs
dict1 = dict(zip(['a', 'b', 'c'], [1, 2, 3]))
print(dict1)                 # {'a': 1, 'b': 2, 'c': 3}

# Unzip
list1, list2 = zip(*[(1, 'a'), (2, 'b'), (3, 'c')])
print(list1)                 # (1, 2, 3)
print(list2)                 # ('a', 'b', 'c')
```

### Reversed & Range

```python
list1 = [1, 2, 3, 4, 5]

for x in reversed(list1):    # 5, 4, 3, 2, 1 (doesn't create copy)
    print(x)

# Range
for i in range(5):           # 0, 1, 2, 3, 4
    print(i)

for i in range(2, 8, 2):     # 2, 4, 6 (start, stop, step)
    print(i)
```

---

## 8. Recursion & Memoization

### Basic Recursion

```python
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))          # 120
```

### Memoization with `@lru_cache` (Interview Hero)

```python
from functools import lru_cache

@lru_cache(maxsize=None)     # cache all results
def fib(n):
    if n <= 1:
        return n
    return fib(n - 1) + fib(n - 2)

print(fib(10))               # 55 (instant, cached)
print(fib(100))              # instant (would be slow without cache)
```

### Manual Memoization (Interview Standard)

```python
def fib(n, memo=None):
    if memo is None:
        memo = {}
    if n in memo:
        return memo[n]
    if n <= 1:
        return n
    memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]

print(fib(10))               # 55
```

### Recursion Depth Limit

```python
import sys
print(sys.getrecursionlimit())      # Default: 1000

sys.setrecursionlimit(10000)        # Increase if needed (be careful!)
# Note: deep recursion can still cause stack overflow
```

---

## 9. Type Hints & None

### Type Hints (Documentation & Clarity)

```python
def add(a: int, b: int) -> int:
    """Add two numbers."""
    return a + b

def process_list(items: list[int]) -> None:
    """Process integers (no return)."""
    for item in items:
        print(item)

def fetch_data(key: str) -> dict[str, int] | None:
    """Fetch data or return None if not found."""
    if key == "valid":
        return {"count": 5}
    return None

# Optional (modern Python 3.10+)
from typing import Optional
def older_style(key: str) -> Optional[dict]:
    return None
```

### Defensive Null Checks

```python
# RISKY — accessing None causes AttributeError
data = fetch_data("missing")
print(data["count"])         # TypeError if data is None

# SAFE — check for None
data = fetch_data("missing")
if data is not None:
    print(data["count"])

# OR use .get() with default
value = data.get("count", 0) if data else 0

# Short-circuit with `or`
value = (data or {}).get("count", 0)
```

---

## 10. Performance Tips

### Avoid These

```python
# SLOW — string concatenation in loop
result = ""
for word in words:
    result += word           # O(n²) — creates new string each time

# FAST — use join
result = "".join(words)      # O(n)

# SLOW — list.pop(0) is O(n)
list1 = [1, 2, 3, 4, 5]
while list1:
    x = list1.pop(0)         # O(n) each time

# FAST — use deque
from collections import deque
q = deque([1, 2, 3, 4, 5])
while q:
    x = q.popleft()          # O(1)

# SLOW — 'in' on list is O(n)
if x in large_list:          # scans entire list
    pass

# FAST — use set
if x in large_set:           # O(1) avg
    pass

# SLOW — list.remove() is O(n)
list1.remove(x)              # finds + removes

# FAST — set difference or list comprehension
list1 = [i for i in list1 if i != x]
```

### Copy Gotcha (Shallow vs Deep)

```python
# Shallow copy (points to same nested objects)
original = [[1, 2], [3, 4]]
copy_shallow = original[:]   # or original.copy()
copy_shallow[0][0] = 999
print(original)              # [[999, 2], [3, 4]] — nested list changed!

# Deep copy (independent)
import copy
copy_deep = copy.deepcopy(original)
copy_deep[0][0] = 999
print(original)              # [[1, 2], [3, 4]] — original untouched
```

### Append Paths in Backtracking

```python
# WRONG — modifies shared list
result = []
path = [1, 2, 3]
result.append(path)          # adds reference
path.append(4)
print(result)                # [[1, 2, 3, 4]] — path changed!

# CORRECT — copy before appending
result = []
path = [1, 2, 3]
result.append(path[:])       # or path.copy() or list(path)
path.append(4)
print(result)                # [[1, 2, 3]] — copy is safe
```

---

## 11. Edge Cases Checklist

Before submitting, verify:

- [ ] **Empty input:** `[]`, `""`, `None`?
- [ ] **Single element:** `[1]`, `[1]`?
- [ ] **All same:** `[5, 5, 5]`?
- [ ] **Negative numbers:** Do they break assumptions?
- [ ] **None values:** Does code handle `None`? Use `.get()` or `if x is None`?
- [ ] **Duplicates:** Are duplicates handled correctly?
- [ ] **Off-by-one:** Array indices, ranges, slices?
- [ ] **String case sensitivity:** `.lower()` vs `.upper()`?
- [ ] **Floating point:** Are `==` comparisons safe (they usually aren't)?
- [ ] **Recursion depth:** Could deep recursion cause stack overflow?
- [ ] **Modifying while iterating:** Don't modify lists while looping over them.
- [ ] **Copied object references:** When appending paths/lists, copy first.
- [ ] **Type mismatches:** Mixed int/float, string/number?

---

## 12. Example: Two Pointers in Python

Let's solve the same problem as Java with Python idioms:

### Problem: Container with Most Water

*Given a list of heights, find the two indices that form a container holding the most water.*

### Solution with Explanations

```python
class Solution:
    def maxArea(self, height: list[int]) -> int:
        """
        Two pointers approach.
        - Start at both ends (widest container).
        - Move the pointer at the shorter bar (only chance to improve height).
        - Track max area.
        """
        # Edge case
        if not height or len(height) < 2:
            return 0
        
        left, right = 0, len(height) - 1
        max_area = 0
        
        # Invariant: max_area = best area found so far
        while left < right:
            # Current area: width × minimum height
            current_area = (right - left) * min(height[left], height[right])
            max_area = max(max_area, current_area)
            
            # Move the pointer at the shorter bar
            # Rationale: taller bar can't improve area (width decreases, min height doesn't increase)
            # Only hope is shorter bar pointing to something taller
            if height[left] < height[right]:
                left += 1
            else:
                right -= 1
        
        return max_area


# Test
if __name__ == "__main__":
    solver = Solution()
    
    test1 = [1, 8, 6, 2, 5, 4, 8, 3, 7]
    print(solver.maxArea(test1))  # 49 (indices 1 and 8, height 8)
    
    test2 = [1, 1]
    print(solver.maxArea(test2))  # 1
    
    test3 = []
    print(solver.maxArea(test3))  # 0 (edge case)
```

### Key Python Patterns Applied

1. **Type hints** — clarity on input/output
2. **Docstring** — explains approach
3. **Edge case check** — `not height or len(height) < 2`
4. **min/max functions** — built-in, readable
5. **List unpacking** — `left, right = ...`
6. **Clear invariant** — comment documents the loop invariant
7. **Pythonic comparison** — `if height[left] < height[right]`

---

## Cheat Sheet (One-Pager)

```
[COLLECTIONS] list→mutable, tuple→hashable, set→no-dups, dict→key-value
[DICT] .get(key, default), .items(), defaultdict, Counter
[STRINGS] immutable; slicing is [start:stop:step]; .join() for concat
[LIST COMP] [x for x in arr if condition] — fast, readable
[SORTING] sorted(list, key=lambda x: x[0], reverse=True)
[DEQUE] from collections; O(1) both ends
[HEAP] heapq; heappush, heappop, nlargest/nsmallest
[MEMO] @lru_cache(maxsize=None) or manual dict
[COPY] shallow: list[:]; deep: copy.deepcopy()
[BINARY SEARCH] bisect.bisect_left/right
[EDGE CASES] Empty, None, duplicates, off-by-one, deep recursion
```

---

## Next Steps

1. **Pick a DSA pattern file** (e.g., 01_ARRAYS_AND_HASHING.md) and solve 3 problems in Python using this guide.
2. **Time yourself** against Java solutions — Python is often 30% faster to code.
3. **Practice list comprehensions** for every loop that builds a new list.
4. **Memoize a recursive solution** using `@lru_cache` to feel the speed boost.
5. **Convert a solution from another language** to Python — focus on idioms, not 1:1 port.
