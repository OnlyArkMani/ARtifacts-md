# JAVA — Interview Essentials & Gotchas

**Target:** Confidently write Java in interviews without surprises. This file is your cheat sheet: common pitfalls, performance traps, syntax quirks, and the patterns that separate "passes" from "aced."

---

## Table of Contents

1. [Primitive vs. Boxed Types](#1-primitive-vs-boxed-types) — where bugs hide
2. [String Handling](#2-string-handling) — immutability, concatenation, performance
3. [Collections & Data Structures](#3-collections--data-structures) — when to pick what
4. [Integer Overflow & Arithmetic](#4-integer-overflow--arithmetic) — the `Integer.MAX_VALUE` trap
5. [Sorting & Comparators](#5-sorting--comparators) — custom comparisons, lambda, thread-safety
6. [Common Library Methods](#6-common-library-methods) — shortcuts that save time
7. [Memory & Performance](#7-memory--performance) — space complexity pitfalls
8. [Multithreading (briefly)](#8-multithreading-basics) — if you see concurrent questions
9. [Edge Cases Checklist](#9-edge-cases-checklist) — pre-interview scan
10. [Example: Two Pointers in Java](#10-example-two-pointers-in-java) — pulling it together

---

## 1. Primitive vs. Boxed Types

### The Difference

| Aspect | Primitive | Boxed |
|--------|-----------|-------|
| Type | `int`, `long`, `double`, `boolean` | `Integer`, `Long`, `Double`, `Boolean` |
| Memory | 4–8 bytes (on stack) | ~16 bytes (on heap) + reference |
| Default value | `0`, `false` | `null` |
| Equality | `==` compares value | `==` compares reference (TRAP!) |
| Collections | ❌ Can't use directly | ✅ Must use in `List`, `Map`, `Set` |
| Performance | Fast | Slower (boxing/unboxing overhead) |

### The Interview Trap: `==` vs `.equals()`

```java
// WRONG — compares memory address, not value
Integer a = 100;
Integer b = 100;
if (a == b) { /* UNRELIABLE — may be true or false */ }

// Cache trick: Java caches integers -128 to 127
Integer x = 127;  // cached
Integer y = 127;  // returns cached instance
if (x == y) { /* TRUE — pure luck */ }

Integer p = 128;  // NOT cached
Integer q = 128;  // new instance
if (p == q) { /* FALSE — oops */ }

// CORRECT — always use .equals() for boxed types
if (a.equals(b)) { /* TRUE */ }

// In a loop: use primitives for performance
int sum = 0;  // not Integer sum = 0;
for (int i = 0; i < n; i++) {
    sum += i;  // no boxing overhead
}
```

### Auto-boxing / Unboxing

Java automatically converts between primitive and boxed:

```java
Integer x = 5;           // auto-boxing: Integer.valueOf(5)
int y = x;               // auto-unboxing: x.intValue()
List<Integer> list = new ArrayList<>();
list.add(10);            // auto-boxing
int first = list.get(0); // auto-unboxing

// GOTCHA: Null unboxing throws NullPointerException
Integer nullable = null;
int z = nullable;        // NullPointerException at runtime!
```

### When to Use Each

- **Primitive (`int`, `long`, etc.):** Default choice. Faster, simpler. Use for counters, indices, math.
- **Boxed (`Integer`, `Long`, etc.):** Only when you need `null` or collections require it. Accept the overhead.

---

## 2. String Handling

### Immutability is Strict

```java
String s = "hello";
s.toUpperCase();  // Does NOT modify s!
System.out.println(s);  // Still "hello"

String t = s.toUpperCase();  // Assign result or it's lost
System.out.println(t);  // "HELLO"
```

Every operation returns a new `String`.

### Concatenation Trap: O(n²) Hidden

```java
// SLOW — creates new String each iteration
String result = "";
for (int i = 0; i < n; i++) {
    result += i;  // O(n) copy each time → O(n²) total
}

// CORRECT — StringBuilder is O(n)
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(i);
}
String result = sb.toString();
```

**Rule:** Any loop with `+=` on strings → use `StringBuilder`.

### String Methods Cheat Sheet

```java
String s = "hello";

// Querying
s.length()                    // 5
s.charAt(0)                   // 'h'
s.indexOf('l')                // 2 (first occurrence)
s.lastIndexOf('l')            // 3
s.contains("ell")             // true
s.startsWith("he")            // true
s.endsWith("lo")              // true

// Transformation
s.toUpperCase()               // "HELLO"
s.toLowerCase()               // "hello"
s.substring(1, 4)             // "ell" (s[1..3], excludes 4)
s.substring(1)                // "ello" (from 1 to end)
s.replace('l', 'x')           // "hexxo"
s.replaceAll("l+", "x")       // regex: "hexo"
s.trim()                      // removes leading/trailing whitespace
s.split(",")                  // String[] — split on delimiter

// Comparison
s.equals("hello")             // true (case-sensitive)
s.equalsIgnoreCase("HELLO")   // true
s.compareTo("hello")          // 0 (lexicographic)

// Conversion
Integer.parseInt("123")       // 123 (String → int)
Long.parseLong("123456789")  // 123456789L
String.valueOf(123)           // "123" (int → String)
Character.getNumericValue('5') // 5 (char digit → int)
```

### Char Array vs String

```java
// For in-place manipulation, convert to char[]
String s = "hello";
char[] arr = s.toCharArray();  // O(n)
arr[0] = 'H';
String result = new String(arr);  // O(n)

// Faster than building a new String via substring + concatenation
```

---

## 3. Collections & Data Structures

### Quick Reference

| Structure | Interface | Implementation | Best For | Time Complexity |
|-----------|-----------|-----------------|----------|------------------|
| **Dynamic array** | `List` | `ArrayList` | random access, append | O(1) amortized append, O(n) insert |
| **Linked list** | `List` | `LinkedList` | insert at front, remove from middle | O(1) front ops, O(n) random access |
| **Hash table** | `Set` / `Map` | `HashMap` / `HashSet` | lookup, no duplicates | O(1) avg, O(n) worst |
| **Sorted map** | `Map` | `TreeMap` | range queries, sorted iteration | O(log n) |
| **Sorted set** | `Set` | `TreeSet` | sorted + no duplicates | O(log n) |
| **Queue (FIFO)** | `Queue` | `LinkedList` / `ArrayDeque` | BFS, job scheduling | O(1) enqueue/dequeue |
| **Stack (LIFO)** | `Deque` | `ArrayDeque` / `Stack` | backtracking, monotonic | O(1) push/pop |
| **Priority queue** | `Queue` | `PriorityQueue` | heap, top-K | O(log n) insert/remove |
| **Counter** | `Map` | `HashMap<K, Integer>` | frequency tables | O(1) |

### List Operations

```java
List<Integer> list = new ArrayList<>();

list.add(10);                 // append: O(1) amortized
list.add(0, 5);               // insert at index 0: O(n)
list.remove(0);               // remove at index: O(n)
list.remove(Integer.valueOf(10)); // remove by value: O(n) + first match only
list.get(0);                  // access: O(1)
list.set(0, 20);              // update: O(1)
list.size();                  // O(1)
list.contains(10);            // O(n)
list.clear();                 // O(n)

// Iteration
for (int x : list) { }
for (int i = 0; i < list.size(); i++) { int x = list.get(i); }
list.forEach(x -> System.out.println(x));  // lambda
```

### HashMap / HashSet

```java
Map<String, Integer> map = new HashMap<>();

map.put("a", 1);              // insert/update: O(1) avg
map.get("a");                 // 1
map.getOrDefault("b", 0);     // 0 (default if key missing)
map.containsKey("a");         // true
map.containsValue(1);         // O(n) — avoid
map.remove("a");              // O(1) avg
map.size();
map.clear();

// Iteration
for (String key : map.keySet()) { }
for (Integer val : map.values()) { }
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    String key = entry.getKey();
    Integer val = entry.getValue();
}

// Frequency counter pattern
Map<Character, Integer> freq = new HashMap<>();
for (char c : s.toCharArray()) {
    freq.put(c, freq.getOrDefault(c, 0) + 1);
}
```

### PriorityQueue (Min-Heap by default)

```java
PriorityQueue<Integer> minHeap = new PriorityQueue<>();  // min at top
PriorityQueue<Integer> maxHeap = new PriorityQueue<>((a, b) -> b - a);  // max at top

minHeap.offer(5);              // insert: O(log n)
minHeap.offer(3);
minHeap.offer(7);

minHeap.peek();                // 3 (top, don't remove)
minHeap.poll();                // 3 (top + remove)
minHeap.size();
minHeap.isEmpty();
```

### Deque (Stack & Queue in one)

```java
Deque<Integer> deque = new ArrayDeque<>();  // much better than Stack class

// Stack operations
deque.push(5);                 // push: O(1)
deque.peek();                  // top: O(1)
deque.pop();                   // pop: O(1)

// Queue operations
deque.addLast(5);              // enqueue: O(1)
deque.peekFirst();             // front: O(1)
deque.pollFirst();             // dequeue: O(1)

// Also has addFirst, removeLast, etc. — very flexible
```

---

## 4. Integer Overflow & Arithmetic

### The Trap

```java
int a = Integer.MAX_VALUE;  // 2^31 - 1 = 2,147,483,647
int b = Integer.MAX_VALUE;
int c = a + b;              // OVERFLOW → wraps to negative!
System.out.println(c);      // -2

long x = Integer.MAX_VALUE;  // safe: promoted to long
long y = Integer.MAX_VALUE;
long z = x + y;              // OK
```

### When to Use `long`

```java
// Any calculation that might overflow:
int n = 100000;
int result = n * n;  // OVERFLOW
long result = (long) n * n;  // Safe

// Sums over arrays
long sum = 0;  // not int sum = 0;
for (int x : array) {
    sum += x;
}

// Large intermediate calculations
int a = 1000000, b = 1000000;
long product = (long) a * b;  // Cast one operand to long

// Indices for large ranges
long index = (long) left + right;  // binary search mid-point
int mid = (int) (left + (right - left) / 2);  // safe way
```

### Avoid Subtraction for Comparison

```java
// WRONG — a - b overflows
if (a - b > 0) { /* ... */ }

// CORRECT
if (a > b) { /* ... */ }

// If you must compare large numbers:
long diff = (long) a - b;
if (diff > 0) { /* ... */ }
```

### Modulo for Large Numbers

```java
long MOD = 1_000_000_007;
long result = (a % MOD + b % MOD) % MOD;
long product = (a % MOD * b % MOD) % MOD;
// Note: (a - b) % MOD can be negative; add MOD if needed
long diffMod = ((a % MOD) - (b % MOD) + MOD) % MOD;
```

---

## 5. Sorting & Comparators

### Primitive Array Sorting

```java
int[] arr = {3, 1, 2};
Arrays.sort(arr);                       // O(n log n), in-place, ascending
// Result: [1, 2, 3]

// For reverse (descending), must use Integer[] boxed:
Integer[] arr2 = {3, 1, 2};
Arrays.sort(arr2, Collections.reverseOrder());  // O(n log n), descending
// Result: [3, 2, 1]
```

### Custom Comparators (for Objects & Boxed Types)

```java
// Syntax 1: Comparator.comparing() — most readable
List<String> words = Arrays.asList("zebra", "apple", "banana");
Collections.sort(words, Comparator.comparing(String::length));  // by length
// Result: ["apple", "zebra", "banana"] (or any length-3)

Collections.sort(words, Comparator.comparing(String::length).reversed());  // descending

// Syntax 2: Lambda (most flexible)
Collections.sort(words, (a, b) -> Integer.compare(a.length(), b.length()));
// Result: by ascending length

Collections.sort(words, (a, b) -> b.length() - a.length());  // descending by length
// CAUTION: b.length() - a.length() can overflow; use Integer.compare() instead

// Syntax 3: Anonymous Comparator class (older, verbose)
Collections.sort(words, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return Integer.compare(a.length(), b.length());
    }
});
```

### Comparator Rules

```java
// Return value:
// < 0 means first comes before second (a < b)
//   0 means equal
// > 0 means first comes after second (a > b)

// Safe comparison (NO overflow):
Integer.compare(a, b);   // handles Integer.MIN/MAX safely
Long.compare(x, y);
Double.compare(d1, d2);
String s1.compareTo(s2); // lexicographic

// WRONG: a - b can overflow
return a - b;
```

### Sort Custom Objects

```java
class Person {
    String name;
    int age;
    
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

List<Person> people = new ArrayList<>();
people.add(new Person("Alice", 30));
people.add(new Person("Bob", 25));

// Sort by age ascending
Collections.sort(people, (a, b) -> Integer.compare(a.age, b.age));

// Sort by age descending, then name ascending (tiebreaker)
Collections.sort(people, (a, b) -> {
    if (a.age != b.age) {
        return Integer.compare(b.age, a.age);  // age descending
    }
    return a.name.compareTo(b.name);  // name ascending
});
```

### TreeMap / TreeSet (Auto-Sorted)

```java
TreeMap<Integer, String> map = new TreeMap<>();  // keys sorted
map.put(3, "three");
map.put(1, "one");
map.put(2, "two");
// Iteration order: 1, 2, 3

TreeSet<Integer> set = new TreeSet<>();  // always sorted
set.add(3);
set.add(1);
set.add(2);
// Iteration order: 1, 2, 3

// Range queries
set.subSet(1, 3);             // [1, 2] (excludes 3)
set.headSet(2);               // [1]
set.tailSet(2);               // [2, 3]
```

---

## 6. Common Library Methods

### Arrays Utility

```java
int[] arr = {1, 2, 3, 4, 5};

Arrays.sort(arr);                        // sorts in-place
Arrays.binarySearch(arr, 3);             // returns index if found, else -(insertion_point + 1)
Arrays.fill(arr, 0);                     // fill entire array with value
Arrays.fill(arr, 0, 3, 9);               // fill arr[0..2] with 9
Arrays.equals(arr1, arr2);               // compare two arrays
Arrays.asList(arr);                      // wrap in List (backed by array)
Arrays.copyOf(arr, arr.length);          // shallow copy
Arrays.copyOfRange(arr, 1, 3);           // copy arr[1..2]
Arrays.toString(arr);                    // "[1, 2, 3, 4, 5]"
```

### Collections Utility

```java
List<Integer> list = Arrays.asList(1, 2, 3);

Collections.sort(list);                  // O(n log n)
Collections.reverse(list);               // O(n)
Collections.shuffle(list);               // randomize order
Collections.max(list);                   // 3
Collections.min(list);                   // 1
Collections.frequency(list, 2);          // count occurrences of 2
Collections.binarySearch(list, 2);       // list must be sorted
Collections.unmodifiableList(list);      // read-only wrapper
```

### Math Utility

```java
Math.abs(-5);                            // 5
Math.min(a, b);                          // minimum
Math.max(a, b);                          // maximum
Math.sqrt(16);                           // 4.0
Math.pow(2, 3);                          // 8.0
Math.floor(3.7);                         // 3.0
Math.ceil(3.2);                          // 4.0
Math.round(3.5);                         // 4

// Bitwise
Integer.bitCount(5);                     // count 1s in binary: 5 = 101 → 2
Integer.highestOneBit(5);                // 4 (highest power of 2 ≤ n)
Integer.lowestOneBit(5);                 // 1
Integer.numberOfLeadingZeros(5);         // leading zeros in 32-bit
Integer.numberOfTrailingZeros(8);        // trailing zeros in binary
Integer.rotateLeft(5, 1);                // rotate bits
```

### Stream API (Optional—often overkill in interviews)

```java
List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

list.stream()
    .filter(x -> x > 2)
    .map(x -> x * 2)
    .collect(Collectors.toList());      // [6, 8, 10]

// Sum
int sum = list.stream().mapToInt(Integer::intValue).sum();

// Distinct
list.stream().distinct().collect(Collectors.toList());

// Sorted
list.stream().sorted().collect(Collectors.toList());
```

**Advice:** Streams are elegant but often slower than explicit loops for small data. Use them if they clarify logic, not for micro-optimization.

---

## 7. Memory & Performance

### Space Complexity Pitfalls

```java
// WRONG — recursion stack grows
private int dfs(int node, List<Integer>[] graph, boolean[] visited) {
    // O(depth) call stack; in worst case (chain graph) → O(n)
    visited[node] = true;
    for (int next : graph[node]) {
        if (!visited[next]) {
            dfs(next, graph, visited);
        }
    }
    return 0;
}
// Deep recursion can cause StackOverflowError

// BETTER — explicit stack (BFS)
Queue<Integer> q = new LinkedList<>();
q.add(start);
visited[start] = true;
while (!q.isEmpty()) {
    int node = q.poll();
    for (int next : graph[node]) {
        if (!visited[next]) {
            visited[next] = true;
            q.add(next);
        }
    }
}
```

### Creating New Objects in Loops

```java
// SLOW — creates n new objects
List<String> result = new ArrayList<>();
for (int i = 0; i < n; i++) {
    result.add(new String(data));  // unnecessary new String
}

// BETTER — reuse StringBuilder
StringBuilder sb = new StringBuilder();
for (int i = 0; i < n; i++) {
    sb.append(data);
}
String result = sb.toString();
```

### ArrayList vs LinkedList

```java
// Prefer ArrayList in 99% of cases
List<Integer> list = new ArrayList<>();  // O(1) random access

// LinkedList only if:
// 1. Frequent removals from middle/front (rare in interviews)
// 2. No random access needed (unlikely)
List<Integer> linked = new LinkedList<>();  // O(n) random access, O(1) front insert
```

---

## 8. Multithreading Basics

*Most interview questions don't involve threading, but here's what to know if they do:*

### Thread-Safe Collections

```java
// NOT thread-safe
List<Integer> list = new ArrayList<>();  // concurrent modification → ConcurrentModificationException

// Thread-safe alternatives
List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
CopyOnWriteArrayList<Integer> list2 = new CopyOnWriteArrayList<>();
```

### Volatile (for simple shared state)

```java
private volatile int flag = 0;  // changes visible across threads

private synchronized int getFlag() {
    return flag;  // lock-based if more complex logic
}
```

---

## 9. Edge Cases Checklist

Before submitting your Java solution, scan this:

- [ ] **Empty input:** `n = 0`, `arr.length = 0`, empty string, null collection?
- [ ] **Single element:** `n = 1`?
- [ ] **All same:** `[5, 5, 5, ...]`?
- [ ] **Negative numbers:** Do they break assumptions (e.g., array indices)?
- [ ] **Integer overflow:** `Integer.MAX_VALUE`, multiplications, sums?
- [ ] **Null references:** Could `map.get(key)` return null? Unbox null Integer?
- [ ] **String case sensitivity:** `.equals()` vs `.equalsIgnoreCase()`?
- [ ] **Off-by-one:** Array indices, substring ranges, loop bounds?
- [ ] **Collections:** Removing during iteration?
- [ ] **Comparator:** Does it handle ties? Overflow on subtraction?
- [ ] **Floating point:** Precision issues with `==` comparison?
- [ ] **Character arithmetic:** `'a' - 'A'`, `Character.isDigit(c)`?

---

## 10. Example: Two Pointers in Java

Let's tie it together with a classic problem:

### Problem: Container with Most Water

*Given an array `height` where `height[i]` is the height of a line, find the two indices that form a container holding the most water.*

### Solution with Inline Explanations

```java
public class ContainerWithMostWater {
    public int maxArea(int[] height) {
        // Edge case: need at least 2 bars
        if (height == null || height.length < 2) {
            return 0;
        }
        
        int left = 0, right = height.length - 1;
        int maxArea = 0;
        
        // Two pointers: start at both ends
        // Invariant: maxArea = best area seen so far
        while (left < right) {
            // Current area: width × min(heights)
            // Use min because water is limited by shorter bar
            int currentArea = (right - left) * Math.min(height[left], height[right]);
            maxArea = Math.max(maxArea, currentArea);
            
            // Move the pointer at the shorter bar
            // Why? Taller bar can't improve area (width ↓, min height ≤ current)
            // Only shorter bar moving toward taller might yield larger min
            if (height[left] < height[right]) {
                left++;  // try to find taller bar on left
            } else {
                right--;  // try to find taller bar on right
            }
        }
        
        return maxArea;
    }
    
    // Test
    public static void main(String[] args) {
        ContainerWithMostWater solver = new ContainerWithMostWater();
        
        int[] test1 = {1, 8, 6, 2, 5, 4, 8, 3, 7};
        System.out.println(solver.maxArea(test1));  // 49 (at indices 1 and 8, height 8)
        
        int[] test2 = {1, 1};
        System.out.println(solver.maxArea(test2));  // 1
        
        int[] test3 = {};
        System.out.println(solver.maxArea(test3));  // 0 (edge case)
    }
}
```

### Key Patterns Applied

1. **Null check + size validation** — defend against empty input
2. **Clear invariant** — maxArea is the best we've found
3. **Pointer movement logic** — greedy: move the shorter bar
4. **Math utilities** — `Math.min()`, `Math.max()`
5. **Test with edge cases** — empty, single size, trivial values

---

## Cheat Sheet (One-Pager)

```
[PRIMITIVES] int/long for math, avoid overflow with long casts
[STRINGS] Immutable; StringBuilder for loops; .equals() not ==
[COLLECTIONS] ArrayList→random access, HashMap→lookup, TreeMap→sorted
[HEAP] PriorityQueue<T>, maxHeap = (a,b) → b-a
[COMPARATOR] Integer.compare(a,b) SAFE; a-b NOT SAFE
[ARRAYS] Arrays.sort(), binarySearch(), fill(), copyOf()
[SORTING] Collections.sort(list, comparator); reversed()
[EDGE CASES] Empty, null, overflow, off-by-one, duplicates
```

---

## Next Steps

1. **Practice with 01_ARRAYS_AND_HASHING.md** using this guide — spot when `.equals()` vs `==` matters.
2. **Build a Comparator** for the custom object exercise in 02_TWO_POINTERS.md.
3. **Rewrite a recursion example as iterative** using `Deque` (09_HEAPS_PRIORITY_QUEUE.md or 11_GRAPHS.md).
4. **Time yourself** converting Python code to Java for familiar problems — get comfortable with the syntax.
