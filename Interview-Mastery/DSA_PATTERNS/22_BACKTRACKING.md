# 22_BACKTRACKING

1. Introduction

What this concept is

Backtracking is a general algorithmic technique for solving problems incrementally, building candidates to the solutions step by step and abandoning a candidate (“backtracking”) as soon as it determines the candidate cannot possibly lead to a valid solution. It's a depth-first search through a solution space with pruning based on feasibility checks.

Why it exists

Many combinatorial search problems have an exponential number of possible candidates. Backtracking provides a structured way to explore the search tree while pruning invalid partial candidates early, often reducing the practical search space dramatically.

What problem it solves

Backtracking solves constraint-satisfaction and combinatorial enumeration problems like puzzles, permutations, combinations, subset-sum, N-Queens, Sudoku, word search, and many search/optimization tasks where partial solutions can be checked for viability.

Real-world analogy

Think of trying to arrange books on a shelf with constraints: you place a book, check if constraints still possible; if not, you remove it and try the next book (backtrack).

Where it is used in industry

- Constraint solvers and configuration systems
- Automated test-case generation and program synthesis
- Combinatorial optimization in scheduling and resource allocation


2. Intuition Section

Visualize the search space as a tree where each level corresponds to a decision (choose a value for the next variable). Backtracking is a depth-first traversal with pruning.

ASCII: generating permutations of [1,2,3]

Start: []
Level1: [1], [2], [3]
Level2 after choosing 1: [1,2], [1,3]
Level3: [1,2,3] completed → backtrack to [1,3]

Pruning example (N-Queens)

Place queens row by row. If placing queen at column c on row r conflicts with earlier queens (same column or diagonal), prune this branch early.


3. Core Theory

Framework

- Choose: pick a variable or position to assign next
- Explore: for each possible value, assign and recurse
- Unchoose: undo assignment (backtrack)
- Prune: check feasibility and skip impossible branches

Complexity

- Worst-case exponential in problem size because search space grows combinatorially; pruning reduces constants and sometimes reduces exponent.

Advanced pruning techniques

- Constraint propagation (forward checking, arc consistency): reduce domains of variables as assignments happen
- Heuristics: choose variable with smallest domain (MRV - minimum remaining values), prefer promising branches (heuristic order)
- Memoization and DP table for overlapping subproblems


4. Java Implementation

Template pseudocode

```java
void backtrack(State state) {
    if (state.isSolution()) {
        output(state);
        return;
    }
    for (Choice c : state.choices()) {
        if (!state.isValidChoice(c)) continue; // pruning
        state.makeChoice(c);
        backtrack(state);
        state.unmakeChoice(c);
    }
}
```

N-Queens example (outline)

- Represent partial solution: array colForRow[row] = chosen column
- isValid checks columns and diagonals
- Use boolean arrays for columns and diagonals to make validity O(1)

Explain: unmakeChoice undoes arrays and col assignment. Complexity: worst-case O(n!).


5. Python Implementation

Permutations example

```python
def permutations(nums):
    res = []
    used = [False]*len(nums)
    cur = []

    def dfs():
        if len(cur) == len(nums):
            res.append(cur.copy())
            return
        for i, x in enumerate(nums):
            if used[i]:
                continue
            used[i] = True
            cur.append(x)
            dfs()
            cur.pop()
            used[i] = False

    dfs()
    return res
```

Explain: used array prevents reuse, copying list to store result, pop/unset on backtrack.

N-Queens pythonic approach uses sets for cols/diag1/diag2 to check conflicts in O(1).


6. Internal Working

Call stack and recursion

- Backtracking uses recursion stack depth equal to number of decisions (e.g., N rows for N-Queens)
- Each recursion frame keeps local variables; use iterative variants if depth too large

Memory

- Auxiliary memory includes recursion stack O(depth) and any domain/used arrays O(n)

Pruning effectiveness

- Heuristics and constraint propagation can cut search space exponentially in practice


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Generate all permutations | O(n!) |
| N-Queens (count solutions) | O(n!) worst-case, less with pruning |
| Subset sum backtracking | O(2^n) worst-case |


8. Space Complexity

- O(depth) for recursion stack
- O(n) for state arrays like used/columns/diagonals


9. Common Interview Questions

Beginner

- Generate permutations/combinations/subsets
- Solve N-Queens (place N queens)

Intermediate

- Word search in grid (use backtracking + trie for multiple words)
- Sudoku solver with backtracking + heuristics

Advanced

- Backtracking combined with DP/memoization (e.g., partition into k subsets with equal sum)
- Constraint satisfaction with forward checking and MRV heuristic


10. Common Mistakes

Mistake: not undoing state correctly (forgetting to pop or unset used flag)
Why: side effects persist across branches and corrupt results
Fix: always pair makeChoice and unmakeChoice in same function frame

Mistake: copying large objects each recursion instead of reusing and undoing causing heavy memory and time costs
Fix: mutate in place and revert to previous state


11. Real Interview Traps

Trap: expecting polynomial solution for inherently exponential problems — clarify limits (n small) and that backtracking is acceptable for n ≤ 15–20

Trap: not applying problem constraints (e.g., uniqueness or limited alphabet) that drastically reduce complexity


12. Real World Applications

- Sudoku solvers, configuration search, automated scheduling, combinatorial design


13. Common LeetCode Problems

Easy
- Subsets
- Permutations

Medium
- Word Search
- Combination Sum

Hard
- Sudoku Solver
- N-Queens II counting solutions for large N


14. Pattern Recognition

When to use backtracking

- Problems asking to enumerate combinations/permutations/subsets or search under constraints
- Problems where you can check partial solutions early (prune)

Decision checklist

- Can you easily check partial feasibility? → backtracking likely useful
- Can you apply heuristics (MRV, forward-checking)? → implement to improve performance


15. Comparison Section

Backtracking vs Dynamic Programming

- Backtracking explores combinations and prunes infeasible ones; DP exploits overlapping subproblems and caches results
- Use memoization on top of backtracking when overlapping subproblems exist (less than exponential distinct states)

Backtracking vs Greedy

- Greedy is faster but may fail; backtracking can explore alternatives


16. 5 Minute Revision

Backtracking

- DFS through solution-space with make-choice/unmake-choice pattern
- Prune early using validity checks; use heuristics like MRV
- Worst-case exponential but often practical with pruning


---
