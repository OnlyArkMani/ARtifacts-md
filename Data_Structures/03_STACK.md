# 03 — Stack: LIFO (Last In, First Out) Data Structure

## Introduction

### What Is a Stack?

A **stack** is a linear data structure that follows the **LIFO (Last In, First Out)** principle:
- The **last element added** is the **first element removed**
- Think of a stack of plates: you add on top, you remove from top

### Real-World Analogy

**Stack of Plates:**
- You add plates on top
- You remove from the top
- You can't remove a plate in the middle without removing the top plate first

**Browser Back Button:**
- Each webpage is "pushed" onto stack
- "Back" button "pops" the current page
- You see the previously visited page

### Why Does It Matter?

Stacks power:
1. **Function calls** — recursion works via stack
2. **Undo/Redo** — each action pushed onto stack
3. **Expression evaluation** — parentheses matching, postfix evaluation
4. **DFS (Depth-First Search)** — graph traversal
5. **Backtracking** — puzzle solving, maze solving

### Where Is It Used?

**Google Chrome:**
- Back button uses stack of pages

**Code Editors:**
- Undo/Redo uses two stacks

**Function Call Stack:**
- Every function call uses stack internally
- This is why deep recursion causes "stack overflow"

**Compiler/Parser:**
- Matching braces: `{ [ ( ) ] }`
- Converting infix to postfix notation

---

## Intuition Section

### Visual Understanding

```
Stack of Plates:

     ┌─────┐
     │ 30  │  ← Top (can only remove from here)
     ├─────┤
     │ 20  │
     ├─────┤
     │ 10  │  ← Bottom
     └─────┘

Operations:
- PUSH (add on top):
     ┌─────┐
     │ 40  │  ← New top
     ├─────┤
     │ 30  │
     ├─────┤
     │ 20  │
     ├─────┤
     │ 10  │
     └─────┘

- POP (remove from top):
     ┌─────┐
     │ 30  │  ← New top
     ├─────┤
     │ 20  │
     ├─────┤
     │ 10  │
     └─────┘
     Removed: 40
```

### Why It's O(1)

```
Stack implemented as array:
[10][20][30][40][50]
              ↑
           top pointer

PUSH 60:
  top = top + 1
  array[top] = 60
  Time: O(1) - just update pointer and add value

POP:
  value = array[top]
  top = top - 1
  Time: O(1) - just get value and update pointer

No shifting, no searching. Always constant time!
```

---

## Core Theory

### Stack Terminology

```
1. PUSH — add element to top
2. POP — remove element from top
3. PEEK/TOP — view top element without removing
4. EMPTY — check if stack is empty
5. SIZE — number of elements
```

### Stack Properties

1. **LIFO:** Last element added is first removed
2. **Sequential:** Elements added and removed in order
3. **Single access point:** Can only access the top
4. **Efficient:** All operations are O(1)

### Implementation Choices

**Using Array:**
```
Pros: Direct memory access, cache friendly
Cons: Fixed size (unless dynamic)
```

**Using Linked List:**
```
Pros: Dynamic size, can grow/shrink
Cons: Extra pointer overhead per node
```

---

## Java Implementation

### Using Stack Class (Built-in)

```java
import java.util.Stack;

// Create stack
Stack<Integer> stack = new Stack<>();

// PUSH: Add element
stack.push(10);
stack.push(20);
stack.push(30);
// Stack: [10, 20, 30]

// PEEK: View top without removing
int top = stack.peek();  // 30
System.out.println(stack);  // [10, 20, 30] unchanged

// POP: Remove and return top
int removed = stack.pop();   // 30
System.out.println(stack);  // [10, 20]

// Check if empty
if (stack.isEmpty()) {
    System.out.println("Stack is empty");
}

// SIZE
int size = stack.size();  // 2

// SEARCH: Find position of element
int position = stack.search(20);  // 2 (from top)
if (position == -1) {
    System.out.println("Not found");
}
```

### Custom Stack Implementation

```java
class MyStack<T> {
    private List<T> data;
    
    MyStack() {
        this.data = new ArrayList<>();
    }
    
    public void push(T value) {
        data.add(value);  // Add to end (top)
    }
    
    public T pop() {
        if (isEmpty()) {
            throw new IllegalStateException("Stack underflow");
        }
        return data.remove(data.size() - 1);  // Remove from end
    }
    
    public T peek() {
        if (isEmpty()) {
            throw new IllegalStateException("Stack is empty");
        }
        return data.get(data.size() - 1);
    }
    
    public boolean isEmpty() {
        return data.size() == 0;
    }
    
    public int size() {
        return data.size();
    }
}
```

### Common Patterns

**Pattern 1: Matching Parentheses**
```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);  // Opening bracket
        } else {
            if (stack.isEmpty()) return false;
            char open = stack.pop();
            
            // Check if matching pair
            if ((c == ')' && open != '(') ||
                (c == ']' && open != '[') ||
                (c == '}' && open != '{')) {
                return false;
            }
        }
    }
    
    return stack.isEmpty();  // All matched?
}

Example: "()[]{}" → true
         "([)]" → false
```

**Pattern 2: Next Greater Element**
```java
public int[] nextGreater(int[] arr) {
    int n = arr.length;
    int[] result = new int[n];
    Stack<Integer> stack = new Stack<>();  // Store indices
    
    for (int i = n - 1; i >= 0; i--) {
        // Pop smaller elements
        while (!stack.isEmpty() && arr[stack.peek()] <= arr[i]) {
            stack.pop();
        }
        
        // Top of stack is next greater (or -1 if none)
        result[i] = stack.isEmpty() ? -1 : arr[stack.peek()];
        
        stack.push(i);
    }
    
    return result;
}

Example: [1, 3, 2, 4] → [3, 4, 4, -1]
```

**Pattern 3: Evaluate Postfix Expression**
```java
public int evaluatePostfix(String expression) {
    Stack<Integer> stack = new Stack<>();
    String[] tokens = expression.split(" ");
    
    for (String token : tokens) {
        if (isOperator(token)) {
            int b = stack.pop();  // Second operand
            int a = stack.pop();  // First operand
            int result = calculate(a, b, token);
            stack.push(result);
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    
    return stack.pop();
}

private int calculate(int a, int b, String op) {
    switch (op) {
        case "+": return a + b;
        case "-": return a - b;
        case "*": return a * b;
        case "/": return a / b;
        default: return 0;
    }
}

Example: "5 3 +" → 8
         "10 2 /" → 5
```

---

## Python Implementation

### Using List as Stack

```python
# Python lists naturally work as stacks
stack = []

# PUSH: Add to end
stack.append(10)
stack.append(20)
stack.append(30)
# Stack: [10, 20, 30]

# PEEK: Access top without removing
top = stack[-1]  # 30
print(stack)  # [10, 20, 30]

# POP: Remove from top
removed = stack.pop()  # 30
print(stack)  # [10, 20]

# Check if empty
if not stack:
    print("Stack is empty")

# SIZE
size = len(stack)
```

### Custom Stack Class

```python
class Stack:
    def __init__(self):
        self.data = []
    
    def push(self, value):
        self.data.append(value)
    
    def pop(self):
        if self.is_empty():
            raise IndexError("Stack underflow")
        return self.data.pop()
    
    def peek(self):
        if self.is_empty():
            raise IndexError("Stack is empty")
        return self.data[-1]
    
    def is_empty(self):
        return len(self.data) == 0
    
    def size(self):
        return len(self.data)
```

### Common Patterns in Python

**Pattern 1: Valid Parentheses**
```python
def is_valid(s: str) -> bool:
    stack = []
    matching = {'(': ')', '[': ']', '{': '}'}
    
    for char in s:
        if char in matching:
            stack.append(char)
        else:
            if not stack or matching[stack.pop()] != char:
                return False
    
    return not stack
```

**Pattern 2: Reverse String**
```python
def reverse_string(s: str) -> str:
    stack = []
    for char in s:
        stack.append(char)
    
    result = ""
    while stack:
        result += stack.pop()
    
    return result
```

---

## Internal Working

### Stack Memory Layout

```
Array-based Stack:
[10][20][30][  ][  ]
             ↑
           top = 2

PUSH 40:
[10][20][30][40][  ]
             ↑
           top = 3

POP:
[10][20][30][  ][  ]
          ↑
        top = 2
Returned: 30
```

### Why Operations Are O(1)

```
PUSH:
  1. Check if capacity exceeded
  2. array[top] = value
  3. top++
  Total: Constant operations = O(1)

POP:
  1. top--
  2. return array[top]
  Total: Constant operations = O(1)

No searching, no shifting = Always O(1)
```

### Recursion Uses Stack

```
Recursive function: factorial(n)

factorial(5)
  factorial(4)
    factorial(3)
      factorial(2)
        factorial(1)
          factorial(0) → 1 (base case)
        return 1
      return 2
    return 6
  return 24
return 120

Each call is pushed onto call stack.
When stack exceeds limit → StackOverflowError!
```

---

## Time Complexity Table

| Operation | Time | Space | Why? |
|-----------|------|-------|------|
| PUSH | O(1) | O(1) | Just add to top, update pointer |
| POP | O(1) | O(1) | Just remove from top, update pointer |
| PEEK | O(1) | O(1) | Just view top element |
| Search | O(n) | O(1) | Worst case: search entire stack |
| isEmpty | O(1) | O(1) | Check if top pointer |

**Overall Space:** O(n) for n elements

---

## Interview Questions & Answers

### Q1: Valid Parentheses

**Question:** Check if brackets are balanced

**Answer:**
```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '[' || c == '{') {
            stack.push(c);
        } else {
            if (stack.isEmpty() || !isMatching(stack.pop(), c)) {
                return false;
            }
        }
    }
    
    return stack.isEmpty();
}

Time: O(n), Space: O(n)
```

### Q2: Reverse Polish Notation (Postfix Evaluation)

**Question:** Evaluate postfix expression

**Answer:**
```java
public int evalRPN(String[] tokens) {
    Stack<Integer> stack = new Stack<>();
    
    for (String token : tokens) {
        if (isOperator(token)) {
            int b = stack.pop();
            int a = stack.pop();
            stack.push(calculate(a, b, token));
        } else {
            stack.push(Integer.parseInt(token));
        }
    }
    
    return stack.pop();
}

Time: O(n), Space: O(n)
```

### Q3: Daily Temperatures

**Question:** Find next warmer day for each day

**Answer:**
```java
public int[] dailyTemperatures(int[] temperatures) {
    int n = temperatures.length;
    int[] result = new int[n];
    Stack<Integer> stack = new Stack<>();  // Indices
    
    for (int i = n - 1; i >= 0; i--) {
        while (!stack.isEmpty() && temperatures[stack.peek()] <= temperatures[i]) {
            stack.pop();
        }
        result[i] = stack.isEmpty() ? 0 : stack.peek() - i;
        stack.push(i);
    }
    
    return result;
}

Time: O(n), Space: O(n)
```

---

## Common Mistakes

### ❌ Mistake 1: POP from Empty Stack

```java
// WRONG
Stack<Integer> stack = new Stack<>();
int value = stack.pop();  // EmptyStackException

// CORRECT
if (!stack.isEmpty()) {
    int value = stack.pop();
}
```

### ❌ Mistake 2: Using Wrong End

```java
// WRONG - Using beginning (FIFO, not LIFO)
public void push(int x) {
    list.add(0, x);  // O(n) operation!
}

// CORRECT - Using end (LIFO)
public void push(int x) {
    list.add(x);     // O(1) operation
}
```

### ❌ Mistake 3: Overflow Management

```java
// WRONG - Not checking size
Stack<Integer> stack = new Stack<>(5);
for (int i = 0; i < 1000; i++) {
    stack.push(i);  // May cause issues
}

// CORRECT - Check size
if (stack.size() < MAX_SIZE) {
    stack.push(i);
}
```

---

## Real Interview Traps

### Trap 1: Matching Pairs

```java
// WRONG - Only checking count
if (open_count == close_count) {
    // Assumes balanced, but ")(" has same count!
}

// CORRECT - Use stack to match
Stack<Character> stack = new Stack<>();
for (char c : s.toCharArray()) {
    if (c == '(') stack.push(c);
    else if (stack.isEmpty() || stack.pop() != c) return false;
}
```

### Trap 2: Recursion Depth

```java
// This causes stack overflow for large n
public int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);  // Stack grows with n
}

// Better: Convert to iterative
public int factorial(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;  // O(1) stack space
}
```

---

## Real World Applications

**Browser Back Button:**
- Each webpage pushed when visited
- Back button pops previous page

**Undo in Text Editors:**
- Each keystroke pushed
- Undo pops and reverses

**Function Call Stack:**
- Each function call pushed
- Return pops and continues

**Expression Evaluation:**
- Converting infix to postfix
- Evaluating postfix expressions

---

## Common LeetCode Problems

**Easy:**
1. Valid Parentheses
2. Min Stack
3. Implement Stack Using Queues

**Medium:**
1. Evaluate Reverse Polish Notation
2. Daily Temperatures
3. Largest Rectangle in Histogram

**Hard:**
1. Trapping Rain Water
2. Maximal Rectangle

---

## 5-Minute Revision

```
Stack = LIFO (Last In, First Out)

Operations (all O(1)):
  PUSH: Add to top
  POP: Remove from top
  PEEK: View top
  isEmpty: Check if empty
  size: Get element count

Used for:
  - Recursion (function call stack)
  - Undo/Redo
  - Bracket matching
  - Expression evaluation
  - DFS
  - Backtracking

Implementation:
  - Array: Fixed size, cache friendly
  - Linked List: Dynamic size, flexible
  - Java: Use Stack<> class
  - Python: Use list with append/pop

Key Insight: Monotonic Stack
  - Track elements in decreasing order
  - Useful for "next greater/smaller" problems
  - Time: O(n) for full array
```

---

Next: Read **04_QUEUE.md**
