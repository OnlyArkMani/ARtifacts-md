# 01 — Arrays & Hashing

## 1. Pattern Overview

The foundation of everything else. The core idea: **trade space for time** by storing what you've already seen in a hash map/set, so lookups that would take O(n) scanning take O(1).

Three sub-ideas cover ~90% of these problems:

- **Complement lookup** — instead of asking "does a pair exist?" (O(n²) to check all pairs), ask for each element "have I already seen the *thing that would complete me*?" (O(n) total).
- **Canonical key (bucketing)** — map each element to a canonical form so "equivalent" items collide: sorted string for anagrams, `(row, col)` tuples, frequency signatures.
- **Frequency counting** — count occurrences first, then reason about the counts instead of the raw data.

**Complexity:** typically O(n) time, O(n) space. Hash operations are O(1) *average* (O(n) worst case with adversarial collisions — never matters in interviews, but say "average" if asked).

## 2. Recognition Triggers

- "Find two/three elements that sum to / relate to each other" → complement lookup.
- "Contains duplicate", "first unique", "has appeared before" → HashSet.
- "Group items by X" (anagrams, equivalence classes) → canonical key → HashMap<key, list>.
- "Count frequency", "most/least common", "top K frequent" → frequency map (often + heap or bucket sort).
- "O(1) lookup required", "in constant time" stated explicitly → hash structure is the answer.
- Unsorted input + you're tempted to sort just to find matches → hashing usually beats the O(n log n) sort.
- "Longest consecutive/streak" in unsorted data → set membership checks.
- n ≤ 10^5–10^6 with a required O(n) or O(n log n) solution and no obvious structure → default to a hash-based pass before anything fancier.
- Values in a small fixed range (e.g., lowercase letters) → array-as-hashmap (`int[26]`) is faster and cleaner.
- **Prefix sums:** "subarray sum equals k", "running total" → prefix sum + hash map of previously-seen prefix values.

## 3. Template Code

### Complement lookup (Java)
```java
// WHY: one pass; for each element, the "partner" it needs is target - num.
// If we've stored every element we've passed, checking for the partner is O(1).
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> seen = new HashMap<>(); // value -> index
    for (int i = 0; i < nums.length; i++) {
        int need = target - nums[i];          // what would complete this element
        if (seen.containsKey(need)) {
            return new int[]{seen.get(need), i};
        }
        seen.put(nums[i], i);                 // store AFTER checking: avoids using self twice
    }
    return new int[]{};
}
```

### Complement lookup (Python)
```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}                       # value -> index
    for i, num in enumerate(nums):
        need = target - num         # the partner this element needs
        if need in seen:            # O(1) membership check
            return [seen[need], i]
        seen[num] = i               # insert after checking → can't pair with itself
    return []
```

### Canonical-key grouping (Java)
```java
// WHY: two strings are anagrams iff they have the same sorted form (or same
// char-count signature). Use that form as the map key so anagrams collide.
public List<List<String>> groupAnagrams(String[] strs) {
    Map<String, List<String>> groups = new HashMap<>();
    for (String s : strs) {
        char[] cs = s.toCharArray();
        Arrays.sort(cs);                        // canonical form
        String key = new String(cs);
        groups.computeIfAbsent(key, k -> new ArrayList<>()).add(s);
    }
    return new ArrayList<>(groups.values());
}
```

### Canonical-key grouping (Python)
```python
from collections import defaultdict

def group_anagrams(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)          # missing key -> new empty list automatically
    for s in strs:
        key = tuple(sorted(s))          # tuple: lists aren't hashable, tuples are
        groups[key].append(s)
    return list(groups.values())
```

### Prefix sum + hashmap (Python — same shape in Java)
```python
def subarray_sum_equals_k(nums: list[int], k: int) -> int:
    # WHY: sum(i..j) == k  <=>  prefix[j] - prefix[i-1] == k
    # So at each j, count how many earlier prefixes equal prefix[j] - k.
    count = 0
    prefix = 0
    seen = {0: 1}                   # empty prefix: subarrays starting at index 0
    for num in nums:
        prefix += num
        count += seen.get(prefix - k, 0)
        seen[prefix] = seen.get(prefix, 0) + 1
    return count
```

## 4. Language-Specific Gotchas

| Concern | Java | Python |
|---|---|---|
| Default map | `HashMap<K,V>` | `dict`; `defaultdict`/`Counter` remove boilerplate |
| Missing key | `getOrDefault(k, 0)` / `computeIfAbsent` | `d.get(k, 0)` / `defaultdict(int)` |
| Hashable keys | Arrays don't override `equals`/`hashCode` — **never use `int[]` as a map key**; convert to `String` or `List<Integer>` | Lists/sets/dicts unhashable — use `tuple`/`frozenset` |
| Int overflow | `int` overflows at ~2.1e9; use `long` for sums/products | Ints are arbitrary precision — never overflow |
| Integer equality | `Integer` boxing: `==` compares references above 127! Use `.equals()` | `==` is fine for ints |
| Char counting | `int[26]`, index via `c - 'a'` | `Counter(s)` or `[0]*26` with `ord(c) - ord('a')` |
| Iteration order | `HashMap` is unordered; `LinkedHashMap` preserves insertion | `dict` preserves insertion order (3.7+) — don't rely on it in Java |

## 5. Worked Examples

### Easy — Two Sum (LC 1)

**Problem:** Given `nums` and `target`, return indices of the two numbers that add to `target`. Exactly one solution; can't reuse an element.

**Recognition trace:** "Two elements that sum to target" → complement lookup. Brute force is O(n²) pairs; storing seen values makes the partner check O(1) → one pass, O(n).

Solution: the templates above *are* the solution.

**Trace** for `nums = [2, 7, 11, 15]`, `target = 9`:
```
i=0  num=2   need=7   seen={}          → miss, store {2:0}
i=1  num=7   need=2   seen={2:0}       → HIT  → return [0, 1]
```

### Medium — Top K Frequent Elements (LC 347)

**Problem:** Return the `k` most frequent elements of `nums`.

**Recognition trace:** "Most frequent" → frequency map. Then "top K" → either heap (O(n log k)) or the neat O(n) trick: **bucket sort by count** — a count can't exceed n, so index buckets by count.

**Java:**
```java
public int[] topKFrequent(int[] nums, int k) {
    Map<Integer, Integer> count = new HashMap<>();
    for (int n : nums) count.merge(n, 1, Integer::sum);

    // buckets[c] = all numbers appearing exactly c times. Max possible count = nums.length.
    List<Integer>[] buckets = new List[nums.length + 1];
    for (var e : count.entrySet()) {
        int c = e.getValue();
        if (buckets[c] == null) buckets[c] = new ArrayList<>();
        buckets[c].add(e.getKey());
    }
    int[] res = new int[k];
    int idx = 0;
    for (int c = nums.length; c >= 0 && idx < k; c--)   // walk from highest count down
        if (buckets[c] != null)
            for (int n : buckets[c]) { res[idx++] = n; if (idx == k) break; }
    return res;
}
```

**Python:**
```python
from collections import Counter

def top_k_frequent(nums: list[int], k: int) -> list[int]:
    count = Counter(nums)
    buckets = [[] for _ in range(len(nums) + 1)]  # index = frequency
    for num, c in count.items():
        buckets[c].append(num)
    res = []
    for c in range(len(buckets) - 1, 0, -1):      # highest frequency first
        for num in buckets[c]:
            res.append(num)
            if len(res) == k:
                return res
    return res
```

**Trace** for `nums = [1,1,1,2,2,3]`, `k = 2`:
```
count = {1:3, 2:2, 3:1}
buckets: idx 0..6 → [ ] [3] [2] [1] [ ] [ ] [ ]
walk c=6→1: c=3 → take 1; c=2 → take 2 → len==k → [1, 2]
```

### Hard — First Missing Positive (LC 41)

**Problem:** Given an unsorted int array, return the smallest missing positive integer. Must be O(n) time, O(1) extra space.

**Recognition trace:** Without the O(1)-space constraint this is trivial with a set. The constraint is the whole problem → **use the array itself as the hash table** (index i should hold value i+1). Key insight: the answer is always in `[1, n+1]`, so values outside that range are noise.

**Java:**
```java
public int firstMissingPositive(int[] nums) {
    int n = nums.length;
    // Cyclic sort: put value v at index v-1, if v is in [1, n].
    for (int i = 0; i < n; i++) {
        // keep swapping until slot i holds something un-placeable or correct
        while (nums[i] > 0 && nums[i] <= n && nums[nums[i] - 1] != nums[i]) {
            int t = nums[nums[i] - 1];
            nums[nums[i] - 1] = nums[i];
            nums[i] = t;
        }
    }
    for (int i = 0; i < n; i++)
        if (nums[i] != i + 1) return i + 1;   // first slot with wrong tenant
    return n + 1;                              // array is a perfect 1..n permutation
}
```

**Python:**
```python
def first_missing_positive(nums: list[int]) -> int:
    n = len(nums)
    for i in range(n):
        # swap nums[i] into its home slot until we can't
        while 0 < nums[i] <= n and nums[nums[i] - 1] != nums[i]:
            home = nums[i] - 1
            nums[i], nums[home] = nums[home], nums[i]
    for i in range(n):
        if nums[i] != i + 1:
            return i + 1
    return n + 1
```

**Trace** for `nums = [3, 4, -1, 1]`:
```
i=0: 3 → home idx 2:  [-1, 4, 3, 1]   (-1 unplaceable, stop)
i=1: 4 → home idx 3:  [-1, 1, 3, 4]   then 1 → home idx 0: [1, -1, 3, 4]  (-1 stops)
i=2: 3 already home.  i=3: 4 already home.
scan: idx0=1 ✓  idx1≠2 → answer 2
```
Note the while-loop total work is O(n) amortized: each swap places one value permanently.

## 6. Common Mistakes & Edge Cases

- Inserting into the map **before** checking the complement → element pairs with itself (`target = 8`, `num = 4`).
- Java: using `==` on boxed `Integer`s — works for small test cases (cache −128..127), fails on big inputs. Classic silent bug.
- Forgetting duplicates: "does the map key already exist and does that matter?" Ask before coding.
- Prefix-sum problems: forgetting to seed `{0: 1}` → misses subarrays that start at index 0.
- Empty input, single-element input, all-identical elements.
- Overflow in Java when summing (`long prefix`, not `int`).
- Sorting when hashing suffices (costs you the optimal complexity) — or hashing when the follow-up says "O(1) space" (then think: sort, in-place cyclic sort, or math).

## 7. Fallback Map

- Need pairs/triples in a **sorted** array (or sorting is allowed and O(1) space wanted) → **Two Pointers** (`02`).
- "Contiguous subarray" with a max/min length or a running constraint → **Sliding Window** (`03`).
- Need range queries repeatedly / sorted-order statistics → **Binary Search** (`05`) or sort first.
- "Top K" with streaming data → **Heap** (`09`).
- Counting structure per-prefix of strings → **Trie** (`08`).
- If O(1) space is demanded and hashing dies → in-place index tricks (cyclic sort, sign-marking) or bit manipulation (`16`).
