# 16 — Math & Bit Manipulation

## 1. Pattern Overview

A grab-bag category, but the payoff is real: these problems are **short to solve if you know the trick, unsolvable if you don't** — so this file is mostly a curated trick inventory.

**Bit manipulation core:** integers are arrays of 32/64 booleans you can process in parallel, in O(1), with no extra space. The must-know identities:

| Trick | Expression | Why it works |
|---|---|---|
| Get bit i | `(x >> i) & 1` | shift target bit to position 0 |
| Set / clear bit i | `x | (1<<i)` / `x & ~(1<<i)` | OR forces 1; AND-NOT forces 0 |
| Toggle | `x ^ (1<<i)` | XOR flips |
| Drop lowest set bit | `x & (x - 1)` | subtraction borrows through trailing zeros |
| Isolate lowest set bit | `x & -x` | two's complement mirrors below the lowest 1 |
| Power of two? | `x > 0 && (x & (x-1)) == 0` | powers have exactly one set bit |
| XOR self-cancel | `a ^ a = 0`, `a ^ 0 = a` | pairs annihilate → find the unpaired |

**Math core:** modular arithmetic (`(a+b) % m`, apply mod early to avoid overflow), fast exponentiation (square-and-multiply, O(log n)), GCD (Euclid), digit extraction (`% 10`, `/ 10`), overflow awareness.

**Complexity:** typically O(1) space; time O(number of bits) or O(log n) — these problems reward *insight density*, not code volume.

## 2. Recognition Triggers

- **"Without using +/-/*/÷"** or "implement arithmetic" → bit-level simulation (XOR = carryless add, AND<<1 = carry).
- **"Every element appears twice except one"** → XOR everything. Appears-3-times variant → per-bit counting mod 3.
- "O(1) space" + "find duplicate/missing number" in range [0, n] → XOR with indices, or sum difference, or cyclic sort (`01`).
- "Count set bits", "hamming distance/weight" → `x & (x-1)` loop or DP on `i >> 1`.
- "Power of 2/4", "single bit" checks → `x & (x-1)`.
- Subset enumeration over n ≤ 20 items → bitmask `for (mask = 0; mask < 1<<n; mask++)` (pairs with `10`, `12`).
- "Reverse bits/integer", "palindrome number" → digit/bit reversal with overflow guard.
- Geometry-flavored: "max points on a line", "valid square" → hashing slopes/distances (`01` + math normalization).
- `pow(x, n)` with huge n, "mod 10^9+7" anywhere → fast exponentiation + modular discipline.
- Constraint smell: values up to 2^31−1 with arithmetic on them → the *test* is overflow handling (Java) / precision (Python floats).

## 3. Template Code

### Bit toolbox (Java)
```java
int countBits(int x) {              // Brian Kernighan: one loop iteration per SET bit
    int count = 0;
    while (x != 0) {
        x &= (x - 1);               // erase lowest set bit
        count++;
    }
    return count;
}

long fastPow(long base, long exp, long mod) {   // O(log exp) square-and-multiply
    long result = 1;
    base %= mod;
    while (exp > 0) {
        if ((exp & 1) == 1)          // current lowest bit of exponent is on
            result = result * base % mod;
        base = base * base % mod;    // square for the next bit position
        exp >>= 1;
    }
    return result;
}

int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
```

### Bit toolbox (Python)
```python
def count_bits(x: int) -> int:
    count = 0
    while x:
        x &= x - 1          # kill lowest set bit
        count += 1
    return count
    # or: bin(x).count("1") / x.bit_count() (3.10+)

# pow(base, exp, mod) is built-in fast modular exponentiation — use it.

def subsets_by_mask(items: list):
    n = len(items)
    for mask in range(1 << n):                    # each mask = one subset
        yield [items[i] for i in range(n) if mask >> i & 1]
```

### XOR find-the-single (both)
```python
def single_number(nums: list[int]) -> int:
    acc = 0
    for x in nums:
        acc ^= x            # pairs cancel; order irrelevant (XOR is commutative)
    return acc
```

## 4. Language-Specific Gotchas

- **Python ints are infinite-precision** — a blessing (no overflow) and a trap (simulating 32-bit ops needs masking: `x & 0xFFFFFFFF`; interpret the sign bit manually: `x if x < 0x80000000 else x - 0x100000000`).
- **Java `>>` vs `>>>`:** arithmetic shift drags the sign bit; use `>>>` for logical shift on negatives. Python has no `>>>` (and no fixed width to need it — mask instead).
- **Java overflow is silent** — `int * int` wraps. Promote early: `(long) a * b`. Python never overflows but floats lose precision past 2^53 — for exactness use int ops only (`//`, not `/`).
- Java `%` can return negative (`-7 % 3 == -1`); Python's is always non-negative (`-7 % 3 == 2`). Modular arithmetic ports between languages break exactly here — in Java use `((a % m) + m) % m`.
- Python `/` is float division always; `//` floors (toward −∞); Java `/` truncates toward 0 — different on negatives.
- Operator precedence: `&`/`|`/`^` bind *looser* than `==` in both languages → `(x & 1) == 0` needs the parens.

## 5. Worked Examples

### Easy — Single Number (LC 136)

**Problem:** Every element appears twice except one; find it. O(n) time, O(1) space.

**Recognition trace:** "appears twice except one" + O(1) space kills hashing → XOR self-cancellation.

**Java:**
```java
public int singleNumber(int[] nums) {
    int acc = 0;
    for (int x : nums) acc ^= x;   // duplicates annihilate pairwise
    return acc;
}
```

**Python:**
```python
from functools import reduce
from operator import xor

def single_number(nums: list[int]) -> int:
    return reduce(xor, nums)
```

**Trace** for `[4, 1, 2, 1, 2]`:
```
acc: 000 ^ 100(4) = 100
     100 ^ 001(1) = 101
     101 ^ 010(2) = 111
     111 ^ 001(1) = 110    ← the 1s cancelled
     110 ^ 010(2) = 100    ← the 2s cancelled → 4
```

### Medium — Sum of Two Integers (LC 371)

**Problem:** Add two integers without `+` or `-`.

**Recognition trace:** "without arithmetic operators" → build addition from logic gates: XOR adds without carry; AND finds carry positions; shift moves carry left; repeat until no carry. (This is literally how ALUs work.)

**Java:**
```java
public int getSum(int a, int b) {
    while (b != 0) {
        int carry = (a & b) << 1;   // positions where both bits are 1 → carry out
        a = a ^ b;                  // carryless sum
        b = carry;                  // add the carry in the next round
    }
    return a;                       // terminates: carry gains a trailing zero each pass
}
```

**Python (needs 32-bit masking — Python ints never overflow into termination):**
```python
def get_sum(a: int, b: int) -> int:
    MASK = 0xFFFFFFFF               # confine to 32 bits
    while b & MASK:
        carry = ((a & b) << 1) & MASK
        a = (a ^ b) & MASK
        b = carry
    a &= MASK
    return a if a < 0x80000000 else a - 0x100000000   # reinterpret sign bit
```

**Trace** for `a = 5 (101)`, `b = 3 (011)`:
```
round 1: xor = 110 (6)   carry = (101 & 011)<<1 = 001<<1 = 010 (2)
round 2: xor = 100 (4)   carry = (110 & 010)<<1 = 010<<1 = 100 (4)
round 3: xor = 000 (0)   carry = (100 & 100)<<1 = 1000 (8)
round 4: xor = 1000 (8)  carry = 0 → done → 8 ✓
```

### Hard — Max Points on a Line (LC 149)

**Problem:** Given points on a plane, find the max number lying on one straight line.

**Recognition trace:** for a fixed anchor point, every other point's relationship to it is fully captured by **slope** — so count slopes in a hashmap (`01` applied to geometry). Trap: float slopes collide/diverge imprecisely → use a **reduced fraction `(dy/g, dx/g)` as the key**, normalizing signs. O(n²) total, optimal here.

**Java:**
```java
public int maxPoints(int[][] points) {
    int n = points.length;
    if (n <= 2) return n;
    int best = 1;
    for (int i = 0; i < n; i++) {                     // anchor point
        Map<String, Integer> slopes = new HashMap<>();
        for (int j = i + 1; j < n; j++) {
            int dx = points[j][0] - points[i][0];
            int dy = points[j][1] - points[i][1];
            int g = gcd(Math.abs(dx), Math.abs(dy));   // reduce the fraction
            dx /= g; dy /= g;
            if (dx < 0 || (dx == 0 && dy < 0)) {       // canonical sign: dx>0, or dx=0,dy>0
                dx = -dx; dy = -dy;
            }
            String key = dy + "/" + dx;                 // exact — no float error
            best = Math.max(best, slopes.merge(key, 1, Integer::sum) + 1); // +1 = anchor
        }
    }
    return best;
}
private int gcd(int a, int b) { return b == 0 ? a : gcd(b, a % b); }
```

**Python:**
```python
from math import gcd
from collections import defaultdict

def max_points(points: list[list[int]]) -> int:
    n = len(points)
    if n <= 2:
        return n
    best = 1
    for i in range(n):
        slopes = defaultdict(int)
        for j in range(i + 1, n):
            dx = points[j][0] - points[i][0]
            dy = points[j][1] - points[i][1]
            g = gcd(dx, dy)                  # math.gcd handles signs/zeros (returns ≥ 0)
            dx, dy = dx // g, dy // g
            if dx < 0 or (dx == 0 and dy < 0):   # one canonical direction per line
                dx, dy = -dx, -dy
            slopes[(dy, dx)] += 1
            best = max(best, slopes[(dy, dx)] + 1)   # +1 counts the anchor itself
    return best
```

**Trace** for `points = [[1,1],[2,2],[3,3],[1,4]]`:
```
anchor (1,1):
  →(2,2): d=(1,1)  g=1 → key (1,1)  count 1
  →(3,3): d=(2,2)  g=2 → key (1,1)  count 2   ← same reduced slope!
  →(1,4): d=(0,3)  g=3 → key (1,0)  count 1
  best = 2 + 1 (anchor) = 3
anchors (2,2),(3,3): find the same line with fewer remaining points; (1,4): pairs only.
answer 3 — the line y = x through (1,1),(2,2),(3,3)
```

## 6. Common Mistakes & Edge Cases

- **Float slopes** — `1/3 != 0.3333...` reliably; always reduced-integer-pair keys. Same for comparing distances: compare *squared* distances.
- Java silent overflow: `reverse integer`, `x * x`, sums — check bounds *before* the multiply/add, or use `long` and compare against int limits.
- Negative `%` in Java breaking hash keys and cyclic indices.
- Python 32-bit simulation without masking → infinite loop (Sum of Two Integers is the canonical example).
- `x & (x-1)` on `x = 0` (no set bits — loops fine, but "power of two" must exclude 0 and negatives).
- Shifting by ≥ 32 in Java is mod-32 (`1 << 32 == 1`!) — undefined-feeling behavior; guard shift amounts.
- Vertical lines (dx = 0) crashing slope division — the fraction-pair representation sidesteps it; raw `dy/dx` doesn't.
- Duplicate points (LC 149 constraints exclude them now, but ask).
- Edge values: `Integer.MIN_VALUE` (its negation overflows!), 0, single-element arrays.

## 7. Fallback Map

- Frequency/pairing logic where XOR feels forced → plain **hashing** (`01`) — only reach for bits when space is constrained or the pairing structure is exact.
- Subset enumeration where n > 20 → bitmasks die; rethink via **DP** (`12`) or greedy.
- "Kth digit / arithmetic sequences" → pure math derivation; write the formula on paper before coding.
- Geometry with many points and precision demands → integer-only arithmetic (cross products for orientation, squared distances) — never floats.
- Bit tricks inside DP states → **bitmask DP** (`13` fallback note): `dp[mask]` = best over the subset `mask`.
- If a "clever" O(1) trick isn't coming after 2–3 minutes in an interview, ship the hashmap/simulation solution and mention the trick's existence — correctness first.
