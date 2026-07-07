# 04 — Stack (incl. Monotonic Stack)

## 1. Pattern Overview

A stack (LIFO) is the right tool whenever the **most recent unresolved thing** is the next thing you'll need. You push items whose fate isn't decided yet; the moment something arrives that resolves them, you pop.

Three big use-cases:

- **Matching/nesting:** parentheses, tags, nested expressions — the innermost open item must close first. LIFO *is* nesting.
- **Monotonic stack:** keep the stack sorted (increasing or decreasing). When a new element violates the order, popping tells you: "this new element is the first smaller/greater element to the right of everything I pop." Turns "next greater element" from O(n²) to O(n).
- **Simulated recursion / state saving:** iterative DFS, expression evaluation, undo semantics.

**Complexity:** O(n) — each element is pushed once and popped at most once, so even nested-looking loops are linear. Space O(n).

## 2. Recognition Triggers

- Explicit nesting: "valid parentheses", "decode nested string", "calculator", "simplify path".
- **"Next greater/smaller element"**, "days until warmer", "stock span" → monotonic stack, near-guaranteed.
- "Largest rectangle", "max area in histogram" → monotonic increasing stack.
- "Remove elements to make smallest/lexicographically smallest result" → greedy pop from a monotonic stack.
- "Previous smaller element" / for each element, find nearest X on the left or right → monotonic stack.
- Processing where **you can't decide an element's answer until a later element arrives** → park it on the stack.
- "Evaluate expression", "RPN", operator precedence → operand/operator stacks.
- Undo/backtrack of most recent action → stack.
- Constraint smell: n ≤ 10^5–10^6, each element's answer depends on the nearest dominating neighbor → the O(n) monotonic stack is intended.

## 3. Template Code

### Monotonic (decreasing) stack — next greater element (Java)
```java
// Stack holds INDICES of elements still waiting for their "next greater".
// Invariant: values at stacked indices are strictly decreasing top-to-bottom-reversed,
// i.e., each new element pops everything smaller than it — and thereby ANSWERS them.
public int[] nextGreater(int[] nums) {
    int n = nums.length;
    int[] res = new int[n];
    Arrays.fill(res, -1);                       // default: no greater element exists
    Deque<Integer> stack = new ArrayDeque<>();  // indices, values decreasing
    for (int i = 0; i < n; i++) {
        while (!stack.isEmpty() && nums[i] > nums[stack.peek()]) {
            res[stack.pop()] = i;               // nums[i] is the first greater to the right
        }
        stack.push(i);                          // i now waits for ITS next greater
    }
    return res;                                  // res holds indices; use nums[i] for values
}
```

### Monotonic stack (Python)
```python
def next_greater(nums: list[int]) -> list[int]:
    res = [-1] * len(nums)
    stack = []                        # indices; nums[stack] is decreasing
    for i, x in enumerate(nums):
        while stack and x > nums[stack[-1]]:
            res[stack.pop()] = i      # x resolves everything smaller on the stack
        stack.append(i)
    return res
```

### Matching template (Python — same logic in Java with ArrayDeque)
```python
def valid_nesting(s: str) -> bool:
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []
    for c in s:
        if c in pairs:                        # closer: must match most recent opener
            if not stack or stack.pop() != pairs[c]:
                return False
        else:
            stack.append(c)                   # opener: fate undecided, park it
    return not stack                          # leftovers = unclosed openers
```

**Choosing the direction:** finding next **greater** → maintain **decreasing** stack (pop while new > top). Next **smaller** → increasing stack. Mnemonic: the stack holds elements *still waiting*, so it must be ordered such that a new element can only resolve the top.

## 4. Language-Specific Gotchas

- **Java: never use `java.util.Stack`** — it's a legacy synchronized `Vector`. Use `ArrayDeque<Integer>` (`push`/`pop`/`peek`).
- **Python:** a plain `list` is the idiomatic stack (`append`/`pop`/`[-1]`). `collections.deque` only needed for queue behavior.
- Python `stack[-1]` on empty list throws `IndexError`; Java `peek()` on empty `ArrayDeque` returns `null` (NPE when unboxed!) — always guard with `isEmpty()` / `if stack`.
- Java autoboxing: `stack.pop() == someInt` may compare references. Use `int` extraction: `int top = stack.pop();`.
- Store **indices, not values**, when you need distances or positions — values alone can't recover position, and duplicates break value-keyed maps.
- Strict vs non-strict pops (`>` vs `>=`): decides how duplicates are handled; histogram problems often need one specific choice — reason it out, don't guess.

## 5. Worked Examples

### Easy — Valid Parentheses (LC 20)

**Problem:** Given `s` containing `()[]{}`, decide if brackets close in valid order.

**Recognition trace:** nesting/matching → stack. Each closer must match the most recent unmatched opener.

Solution: the matching template above is the full solution (Java version):

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pairs = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : s.toCharArray()) {
        if (pairs.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pairs.get(c)) return false;
        } else stack.push(c);
    }
    return stack.isEmpty();
}
```

**Trace** for `s = "([{}])"`:
```
( → push       stack: (
[ → push       stack: ( [
{ → push       stack: ( [ {
} → pop { ✓    stack: ( [
] → pop [ ✓    stack: (
) → pop ( ✓    stack: ∅   → true
```

### Medium — Daily Temperatures (LC 739)

**Problem:** For each day, how many days until a warmer temperature? 0 if never.

**Recognition trace:** "For each element, first greater to the right" → textbook monotonic decreasing stack. Answer needs a *distance* → store indices.

**Java:**
```java
public int[] dailyTemperatures(int[] temps) {
    int[] res = new int[temps.length];            // default 0 = never warmer
    Deque<Integer> stack = new ArrayDeque<>();    // indices of unresolved days
    for (int i = 0; i < temps.length; i++) {
        while (!stack.isEmpty() && temps[i] > temps[stack.peek()]) {
            int day = stack.pop();
            res[day] = i - day;                   // waited (i - day) days
        }
        stack.push(i);
    }
    return res;
}
```

**Python:**
```python
def daily_temperatures(temps: list[int]) -> list[int]:
    res = [0] * len(temps)
    stack = []                                  # indices, temps decreasing
    for i, t in enumerate(temps):
        while stack and t > temps[stack[-1]]:
            day = stack.pop()
            res[day] = i - day
        stack.append(i)
    return res
```

**Trace** for `temps = [73, 74, 75, 71, 69, 72, 76]`:
```
i=0 (73): push               stack: [0]
i=1 (74): 74>73 pop0 res[0]=1; push  stack: [1]
i=2 (75): 75>74 pop1 res[1]=1; push  stack: [2]
i=3 (71): push               stack: [2,3]
i=4 (69): push               stack: [2,3,4]
i=5 (72): 72>69 pop4 res[4]=1; 72>71 pop3 res[3]=2; push  stack: [2,5]
i=6 (76): pop5 res[5]=1; pop2 res[2]=4; push        stack: [6]
res = [1,1,4,2,1,1,0]
```

### Hard — Largest Rectangle in Histogram (LC 84)

**Problem:** Given bar heights, find the largest rectangle fitting under the histogram.

**Recognition trace:** For each bar, the widest rectangle at *its* height extends left/right until a **shorter** bar. "Nearest smaller on each side" → monotonic **increasing** stack. When a bar pops, the popper is its right boundary and the new stack top is its left boundary — compute area at pop time.

**Java:**
```java
public int largestRectangleArea(int[] heights) {
    Deque<Integer> stack = new ArrayDeque<>();  // indices, heights increasing
    int best = 0, n = heights.length;
    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];      // sentinel 0 flushes the stack at the end
        while (!stack.isEmpty() && h < heights[stack.peek()]) {
            int height = heights[stack.pop()];  // this bar's rectangle is now closed
            // width: from element after new top, up to i-1
            int width = stack.isEmpty() ? i : i - stack.peek() - 1;
            best = Math.max(best, height * width);
        }
        stack.push(i);
    }
    return best;
}
```

**Python:**
```python
def largest_rectangle_area(heights: list[int]) -> int:
    stack = []          # indices into padded array, heights increasing
    hs = heights + [0]  # sentinel
    best = 0
    for i, h in enumerate(hs):
        while stack and h < hs[stack[-1]]:
            height = hs[stack.pop()]
            width = i if not stack else i - stack[-1] - 1
            best = max(best, height * width)
        stack.append(i)
    return best
```

**Trace** for `heights = [2, 1, 5, 6, 2, 3]` (+ sentinel 0):
```
i=0 h=2: push                       stack: [0]
i=1 h=1: 1<2 → pop0: h=2 w=1 area=2 | push   stack: [1]
i=2 h=5: push                       stack: [1,2]
i=3 h=6: push                       stack: [1,2,3]
i=4 h=2: 2<6 → pop3: h=6 w=1 area=6
         2<5 → pop2: h=5 w=2 area=10 ★
         push                       stack: [1,4]
i=5 h=3: push                       stack: [1,4,5]
i=6 h=0: pop5: h=3 w=1 area=3
         pop4: h=2 w=4 area=8
         pop1: h=1 w=6 area=6       → best = 10
```

## 6. Common Mistakes & Edge Cases

- Peeking/popping an empty stack (guard every access).
- Wrong monotonic direction — if your pops never resolve anything, you built the stack backwards.
- Histogram width formula: `i - stack.peek() - 1` after popping, and `i` when the stack empties (the bar extends to the far left). Both branches are needed.
- Forgetting the sentinel/final flush — bars still on the stack at the end have their answer too.
- Storing values when you need indices.
- `>` vs `>=` with duplicates: for histogram either works (equal bars recompute), but for "next strictly greater" it must be strict.
- Java `Deque` push/pop operate on the *head* — mixing `push` with `addLast` corrupts order.
- Single bar, all-increasing, all-decreasing, all-equal inputs.

## 7. Fallback Map

- Need FIFO order instead of LIFO → **queue / BFS** (`07`, `11`).
- Max/min over a sliding window (both ends active) → **monotonic deque** (stack + window hybrid).
- Nested structure but you need global aggregation over subtrees → **recursion / Trees** (`07`).
- "Next greater in a circular array" → same stack, iterate 2n with modulo.
- Expression parsing beyond one level of precedence → recursive descent (recursion is a stack anyway).
- If the answer for each element depends on *all* elements to its right (not just nearest dominating) → suffix scans or DP (`12`).
