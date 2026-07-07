# 06 — Linked List

## 1. Pattern Overview

Linked lists trade random access (arrays: O(1) index, O(n) insert) for pointer surgery (lists: O(n) access, O(1) insert *given the node*). Interview problems are rarely about the structure itself — they test whether you can **manipulate pointers without losing nodes** and whether you know the two core tricks:

- **Dummy (sentinel) node:** a fake head placed before the real list. Eliminates every "is this the head?" special case in insert/delete problems. Cheap insurance; use it by default.
- **Fast & slow pointers (Floyd):** two pointers at different speeds. Finds middles (fast 2×), detects cycles (they must meet inside a cycle), finds nth-from-end (fixed gap). Works because relative speed is constant: in a cycle of length C, fast gains one step per tick and must lap slow within C ticks.
- **In-place reversal:** iterate with `(prev, cur)`, flipping one `next` pointer per step.

**Complexity:** almost everything is O(n) time, O(1) space — that O(1) is usually the point (the array-copy solution is trivial and worth mentioning, then discarding, in the interview).

## 2. Recognition Triggers

- The input *is* a linked list — the pattern set is small and closed: reverse, merge, cycle, middle, nth-from-end, reorder, deep copy, add-as-numbers.
- "Reverse (part of) a list" → in-place reversal with prev/cur.
- "Detect cycle / find cycle start" → fast & slow.
- "Middle of the list" → fast & slow (fast 2×).
- "Nth node from the end" → two pointers with an n-gap, single pass.
- "Merge two/k sorted lists" → dummy node + tail pointer (+ heap for k, see `09`).
- "Remove/insert node" where head might change → dummy node.
- "O(1) space" on a list problem → pointer manipulation, no array copy.
- Design problems: "LRU cache" → doubly linked list + hashmap.
- Cycle detection on *values as pointers* (Find the Duplicate Number LC 287) — arrays can hide linked lists: `i → nums[i]` is a functional graph → Floyd applies.

## 3. Template Code

### In-place reversal (Java)
```java
// Walk the list flipping one arrow per step.
// Invariant: prev = head of the already-reversed part; cur = head of the rest.
public ListNode reverseList(ListNode head) {
    ListNode prev = null, cur = head;
    while (cur != null) {
        ListNode next = cur.next;  // save forward link BEFORE overwriting it
        cur.next = prev;           // flip the arrow
        prev = cur;                // reversed part grows by one
        cur = next;                // advance into the unreversed part
    }
    return prev;                   // cur is null; prev is the new head
}
```

### In-place reversal (Python)
```python
def reverse_list(head):
    prev, cur = None, head
    while cur:
        cur.next, prev, cur = prev, cur, cur.next  # simultaneous: no temp needed
    return prev
```

### Fast & slow (Python — same shape in Java)
```python
def middle(head):
    slow = fast = head
    while fast and fast.next:      # guard BOTH: fast may be at last or past-last node
        slow = slow.next           # 1 step
        fast = fast.next.next      # 2 steps
    return slow                    # even length: second of the two middles

def has_cycle(head) -> bool:
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:           # identity, not value equality
            return True
    return False
```

### Dummy node pattern (Java)
```java
// WHY: deleting/inserting near the head is a special case ONLY if you don't
// have a node before the head. So create one.
public ListNode removeElements(ListNode head, int val) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode cur = dummy;
    while (cur.next != null) {
        if (cur.next.val == val) cur.next = cur.next.next;  // splice out
        else cur = cur.next;
    }
    return dummy.next;             // real head, whatever it now is
}
```

## 4. Language-Specific Gotchas

- **Python:** compare nodes with `is`, not `==` (unless the class defines equality — LC's doesn't, but `==` on the same object works by identity anyway; `is` states intent).
- **Python tuple assignment** in reversal: `cur.next, prev, cur = prev, cur, cur.next` — evaluation order matters and this exact order is correct; reordering breaks it silently.
- **Java:** `NullPointerException` is your enemy — every `.next.next` needs the preceding null guard. Python raises `AttributeError` similarly; the guard style `while fast and fast.next` is identical.
- Neither language has a built-in singly linked list worth using in interviews (Java's `LinkedList` is doubly linked with list-API overhead; Python has none) — you always work with the given `ListNode` class.
- Drawing boxes and arrows on paper is not optional. Pointer bugs are visual bugs.
- Java: object references make aliasing explicit; Python: same, but it's easier to accidentally rebind a *local variable* thinking you changed the list. `head = head.next` moves your label, not the list.

## 5. Worked Examples

### Easy — Reverse Linked List (LC 206)

**Problem:** Reverse a singly linked list.

**Recognition trace:** the canonical reversal — template above is the answer. Also know the recursive version (interviewers ask for both).

**Java (recursive variant):**
```java
public ListNode reverseList(ListNode head) {
    if (head == null || head.next == null) return head;  // empty or last node
    ListNode newHead = reverseList(head.next);  // reverse everything after me
    head.next.next = head;                      // my successor now points back at me
    head.next = null;                           // I'm the (current) tail
    return newHead;                             // bubbles up unchanged
}
```

**Python (iterative — the default answer):** see template above.

**Trace** for `1 → 2 → 3 → ∅`:
```
prev=∅        cur=1→2→3
step: 1.next=∅        prev=1        cur=2      state: ∅←1  2→3
step: 2.next=1        prev=2        cur=3      state: ∅←1←2  3
step: 3.next=2        prev=3        cur=∅      state: ∅←1←2←3
return prev = 3 → 2 → 1 → ∅
```

### Medium — Linked List Cycle II (LC 142)

**Problem:** Return the node where the cycle begins, or null.

**Recognition trace:** cycle → Floyd. Phase 1: fast/slow meet inside the cycle. Phase 2 (the clever part): distance from head to cycle start = distance from meeting point to cycle start (mod cycle length). Proof sketch: if head→start = a, start→meet = b, cycle = C, then 2(a+b) = a+b+kC → a = kC − b — walking a from head equals walking a from meet.

**Java:**
```java
public ListNode detectCycle(ListNode head) {
    ListNode slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) {                    // phase 1: met inside cycle
            ListNode p = head;                 // phase 2: two walkers, same speed
            while (p != slow) {
                p = p.next;
                slow = slow.next;
            }
            return p;                          // they meet exactly at cycle start
        }
    }
    return null;                               // fast fell off the end → no cycle
}
```

**Python:**
```python
def detect_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
        if slow is fast:                # collision inside the cycle
            p = head
            while p is not slow:        # both advance 1 step; meet at entrance
                p = p.next
                slow = slow.next
            return p
    return None
```

**Trace** for `3 → 2 → 0 → -4 ─┐` with `-4 → 2` (cycle starts at node `2`):
```
        3 → 2 → 0 → -4
            ↑________|
phase 1: slow: 3,2,0,-4   fast: 3,0,2,0? …
  t1: slow=2  fast=0
  t2: slow=0  fast=0? no: fast=-4→2 → fast=2… (they meet at node 0 or -4 dep. on layout)
  meeting happens; say meet = -4
phase 2: p=3, slow=-4
  t1: p=2,  slow=2  → p is slow → return node 2 ✓
```

### Hard — Reverse Nodes in k-Group (LC 25)

**Problem:** Reverse the list k nodes at a time; leftover tail (< k nodes) stays as-is.

**Recognition trace:** reversal + segmentation. Combine the two templates: dummy node (head will change), probe k ahead to confirm a full group, reverse that window in place, stitch: `groupPrev → (reversed group) → nextGroup`. The stitching is the hard part — name your pointers.

**Java:**
```java
public ListNode reverseKGroup(ListNode head, int k) {
    ListNode dummy = new ListNode(0);
    dummy.next = head;
    ListNode groupPrev = dummy;                 // node BEFORE the current group

    while (true) {
        ListNode kth = groupPrev;               // probe: is there a full group?
        for (int i = 0; i < k && kth != null; i++) kth = kth.next;
        if (kth == null) break;                 // fewer than k left → done

        ListNode groupNext = kth.next;          // first node AFTER the group
        // reverse [groupPrev.next .. kth]; seed prev with groupNext so the
        // group's old head automatically points at the following group.
        ListNode prev = groupNext, cur = groupPrev.next;
        while (cur != groupNext) {
            ListNode next = cur.next;
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        ListNode oldGroupHead = groupPrev.next; // will become the group's TAIL
        groupPrev.next = kth;                   // stitch front: prev → new head
        groupPrev = oldGroupHead;               // advance to before next group
    }
    return dummy.next;
}
```

**Python:**
```python
def reverse_k_group(head, k):
    dummy = ListNode(0)
    dummy.next = head
    group_prev = dummy

    while True:
        kth = group_prev                    # find k-th node of this group
        for _ in range(k):
            kth = kth.next
            if not kth:
                return dummy.next           # partial group → leave untouched
        group_next = kth.next

        prev, cur = group_next, group_prev.next   # reverse group; tail links onward
        while cur is not group_next:
            cur.next, prev, cur = prev, cur, cur.next

        old_head = group_prev.next          # old head is now group tail
        group_prev.next = kth               # attach reversed group
        group_prev = old_head               # move anchor past this group
```

**Trace** for `1→2→3→4→5`, `k = 2`:
```
dummy→1→2→3→4→5     groupPrev=dummy, kth=2, groupNext=3
reverse [1,2] seeded with 3:  2→1→3→4→5
stitch: dummy→2 …             groupPrev=1
dummy→2→1→3→4→5     groupPrev=1, kth=4, groupNext=5
reverse [3,4] seeded with 5:  4→3→5
stitch: 1→4 …                 groupPrev=3
dummy→2→1→4→3→5     probe from 3: 5, then null → partial → stop
answer: 2→1→4→3→5
```

## 6. Common Mistakes & Edge Cases

- Losing the rest of the list: overwrite `cur.next` before saving it → everything after is garbage-collected. Save `next` first, always.
- No dummy node → separate branch for head deletion → bugs under pressure.
- `while fast and fast.next` — missing either guard NPEs on even/odd lengths respectively.
- Even-length middle: fast/slow yields the *second* middle. "Split in half" problems often need the first → stop earlier (`while fast.next and fast.next.next`) or track prev.
- Cycle detection with `==` value comparison instead of node identity.
- Forgetting to sever (`tail.next = null`) after splitting — creates accidental cycles → infinite loops.
- k-group: reversing a partial final group (must probe first).
- Off-by-one on "nth from end" gap: advance the lead pointer n+1 from dummy to land *before* the deletion target.
- Single node, empty list, k=1, all inputs in a cycle.

## 7. Fallback Map

- Merge k lists → **Heap** (`09`) holding one head per list.
- Need random access / index arithmetic → copy values into an array first (say the tradeoff out loud), or the problem isn't really a list problem.
- "LRU/LFU cache" → doubly linked list + hashmap combo (design pattern, worth memorizing wholesale).
- Cycle in a functional graph disguised as an array (`i → nums[i]`) → Floyd (this file), not hashing, when O(1) space demanded.
- List problems asking about order statistics → convert to array + **Binary Search/sort** (`05`).
- If pointer surgery gets 4+ levels deep, stop and re-draw; usually a missing dummy or a mis-ordered stitch.
