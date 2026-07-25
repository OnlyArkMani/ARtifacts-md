# 23_DYNAMIC_PROGRAMMING

1. Introduction

What this concept is

Dynamic Programming (DP) is a method for solving complex problems by breaking them down into simpler overlapping subproblems, solving each subproblem once, and storing their solutions — typically in a table — to avoid redundant computations. It's closely related to recursion with memoization but emphasizes bottom-up tabulation as well.

Why it exists

DP exists to transform exponential-time recursive solutions into polynomial-time solutions by reusing previously computed results. Problems with overlapping subproblems and optimal substructure are ideal for DP.

What problem it solves

It solves recurrence-based optimization and counting problems like knapsack, coin change, longest increasing subsequence, edit distance, and many scheduling/partitioning tasks.

Real-world analogy

Building a skyscraper: complete each floor (subproblem) and reuse knowledge/infrastructure for higher floors instead of rebuilding from scratch each time.

Where it is used in industry

- Resource allocation, planning and scheduling
- Bioinformatics (sequence alignment, edit distance)
- Speech recognition, NLP (Viterbi), recommendation systems


2. Intuition Section

Core idea

If a problem can be described recursively via choices that split the problem into smaller subproblems, and those subproblems repeat, then compute them once and reuse results.

Example: Fibonacci

Naive recursion: fib(n) = fib(n-1) + fib(n-2) → exponential
DP: store fib(0..n) in table and compute sequentially O(n)

Visual: recursion tree with repeated nodes; DP collapses identical nodes by storing results.


3. Core Theory

Two approaches

- Top-down memoization: write recursive solution and cache results in hashmap/array
- Bottom-up tabulation: fill dp table iteratively from base cases up to target

Key properties

- Overlapping subproblems: same subproblem computed multiple times in naive recursion
- Optimal substructure: optimal solution to problem contains optimal solutions to subproblems

State and transition

- Choose minimal sufficient state to represent subproblem (index, remaining capacity, previous value)
- Define recurrence transition expressing solution in terms of smaller states

Complexity

- Time and space depend on number of states and cost per transition; typical DP reduces exponential to polynomial


4. Java Implementation

Classic 0/1 Knapsack (bottom-up)

```java
int knapsack(int[] wt, int[] val, int W) {
    int n = wt.length;
    int[][] dp = new int[n+1][W+1];
    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];
            if (w >= wt[i-1]) dp[i][w] = Math.max(dp[i][w], dp[i-1][w-wt[i-1]] + val[i-1]);
        }
    }
    return dp[n][W];
}
```

Explain: dp[i][w] = max value using first i items with capacity w. Space can be optimized to O(W) by rolling arrays when transitions only depend on previous row.


5. Python Implementation

Top-down memoization example (Fibonacci)

```python
from functools import lru_cache

@lru_cache(None)
def fib(n):
    if n < 2: return n
    return fib(n-1) + fib(n-2)
```

Bottom-up LIS (n log n version exists but classic DP O(n^2) approach)

```python
def lis(nums):
    n = len(nums)
    dp = [1]*n
    res = 0
    for i in range(n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)
        res = max(res, dp[i])
    return res
```


6. Internal Working

State explosion and pruning

- Choose minimal state; avoid redundant state components that increase table size exponentially.

Memoization vs tabulation

- Memoization is often quicker to write and may avoid computing unreachable states; tabulation is often faster in practice and avoids recursion overhead.

Optimization techniques

- Space optimization: rolling arrays, iterative overwrites
- Time optimization: monotonic queues, convex hull trick, bitsets, divide-and-conquer DP optimization


7. Time Complexity Table

| Operation | Complexity |
| --------- | ---------- |
| Fibonacci naive | O(2^n) |
| Fibonacci DP | O(n) |
| Knapsack (0/1 DP) | O(n*W) |
| LIS (DP) | O(n^2) |


8. Space Complexity

- Depends on number of states — can often be reduced via rolling arrays to O(W) or O(n)


9. Common Interview Questions

Beginner

- Fibonacci with memoization
- Climbing stairs, minimum path sum in grid

Intermediate

- Longest increasing subsequence (DP and patience sorting O(n log n))
- Partition equal subset sum (subset-sum DP)

Advanced

- DP on trees (tree DP), DP with bitmasking (TSP), divide and conquer DP optimizations


10. Common Mistakes

Mistake: choosing a state that’s too large (includes unnecessary variables)
Fix: analyze problem to find minimal state; try sample small inputs and trace states

Mistake: forgetting base cases or incorrect transition leading to off-by-one or wrong indexing


11. Real Interview Traps

Trap: expecting simple DP when optimization requires more advanced techniques (e.g., convex hull trick for monotonic costs)

Trap: not recognizing DP shape (overlapping subproblems) and using brute-force search


12. Real World Applications

- Resource allocation, sequence alignment, bandwidth optimization, pricing and inventory planning


13. Common LeetCode Problems

Easy
- Climbing Stairs
- House Robber

Medium
- Longest Increasing Subsequence
- Partition Equal Subset Sum

Hard
- Bitmasking DP for TSP, DP with complex state transitions like some combinatoric counting problems


14. Pattern Recognition

When to use DP

- Problem can be described recursively and subproblems overlap
- Constraints small enough to hold table for states

Decision checklist

- Can you write recurrence? Identify state and transitions
- Is there overlapping subproblems? → memoize


15. Comparison Section

DP vs Greedy

- Greedy picks local optimum hoping for global optimum; DP systematically computes global optimum by exploring states

DP vs Backtracking

- Backtracking explores space and prunes; DP caches results to avoid recomputation of overlapping subproblems


16. 5 Minute Revision

Dynamic Programming

- Identify state and recurrence
- Use memoization or tabulation to reuse solutions
- Optimize space and time with problem-specific techniques


---
