# 🧠 Dynamic Programming

> *"Those who cannot remember the past are condemned to recompute it."*

---

## What is Dynamic Programming?

Dynamic Programming (DP) is an **algorithmic technique** for solving problems by breaking them into **overlapping subproblems**, solving each subproblem **exactly once**, and storing the result to avoid redundant work.

It's not a data structure — it's a **problem-solving strategy** that turns exponential algorithms into polynomial ones by trading memory for speed.

Two conditions must hold for DP to apply:
1. **Optimal substructure** — the optimal solution to the problem contains optimal solutions to its subproblems
2. **Overlapping subproblems** — the same subproblems are solved multiple times in a naive recursive approach

---

## The Core Idea — Fibonacci

<p align="center">
  <img src="./assets/dp-overview.png" alt="DP Overview — Naive vs Memoization" width="900">
</p>

Fibonacci without DP recomputes the same values exponentially. DP stores each result the first time and retrieves it in O(1) thereafter.

---

## Two Approaches

<p align="center">
  <img src="./assets/dp-approaches.png" alt="Top-Down Memoization vs Bottom-Up Tabulation" width="900">
</p>

### Approach 1 — Top-Down (Memoization)

Start from the original problem, recurse down, cache results.

```python
def fib(n, memo={}):
    if n in memo:
        return memo[n]           # ← O(1) cache hit
    if n <= 1:
        return n
    memo[n] = fib(n-1, memo) + fib(n-2, memo)
    return memo[n]

print(fib(50))   # 12586269025 — instant
```

```
Call tree (n=5):
  fib(5)
    fib(4)
      fib(3)
        fib(2)
          fib(1) → 1
          fib(0) → 0
        returns 1 → cached
      fib(2) → cache hit! ✓
    returns 3 → cached
  fib(3) → cache hit! ✓
returns 5
```

### Approach 2 — Bottom-Up (Tabulation)

Start from base cases, fill a table iteratively up to the answer.

```python
def fib(n):
    if n <= 1:
        return n
    dp = [0] * (n + 1)
    dp[0], dp[1] = 0, 1
    for i in range(2, n + 1):
        dp[i] = dp[i-1] + dp[i-2]   # each cell uses previous two
    return dp[n]

print(fib(50))   # 12586269025
```

**Space optimised** — only keep last two values:

```python
def fib(n):
    a, b = 0, 1
    for _ in range(n):
        a, b = b, a + b
    return a
```

---

## Top-Down vs Bottom-Up

| Feature              | Top-Down (Memoization)     | Bottom-Up (Tabulation)      |
|----------------------|----------------------------|-----------------------------|
| Style                | Recursive                  | Iterative                   |
| Starting point       | Original problem           | Base cases                  |
| Memory               | Only needed subproblems    | Full table (or optimised)   |
| Stack risk           | Stack overflow for large n | No recursion, no risk       |
| Easier to write      | ✅ Yes (follows recursion) | Takes more thought upfront  |
| Faster in practice   | Slightly slower (call overhead) | ✅ Yes                |
| When to use          | When not all subproblems needed | When all subproblems needed |

---

## The DP Framework — 5 Steps

```
1. DEFINE the subproblem
   What does dp[i] mean? What does dp[i][j] mean?

2. IDENTIFY the recurrence relation
   How does dp[i] relate to smaller subproblems?

3. SET base cases
   What are the smallest inputs with known answers?

4. DETERMINE the order of computation
   Which subproblems must be solved before others?

5. EXTRACT the answer
   Is it dp[n]? max(dp)? dp[m][n]?
```

---

## Classic Problem 1 — 0/1 Knapsack

**Problem:** Given items with weights and values, and a bag of capacity W — what's the maximum value you can carry?

```
items  = [(weight=2, value=3), (weight=3, value=4), (weight=4, value=5), (weight=5, value=6)]
W      = 8

Subproblem: dp[i][w] = max value using first i items with weight limit w

Recurrence:
  If item i is too heavy:   dp[i][w] = dp[i-1][w]
  If item i fits:           dp[i][w] = max(dp[i-1][w],          ← skip item i
                                           dp[i-1][w-weight[i]] + value[i])  ← take item i
```

```python
def knapsack(weights, values, W):
    n = len(weights)
    dp = [[0] * (W + 1) for _ in range(n + 1)]

    for i in range(1, n + 1):
        for w in range(W + 1):
            dp[i][w] = dp[i-1][w]                          # skip item
            if weights[i-1] <= w:
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - weights[i-1]] + values[i-1])   # take item

    return dp[n][W]


weights = [2, 3, 4, 5]
values  = [3, 4, 5, 6]
print(knapsack(weights, values, 8))   # 10  (items 0+1+? → 3+4+? check all)
```

Time: O(n × W)   Space: O(n × W) → optimisable to O(W) with 1D array

---

## Classic Problem 2 — Longest Common Subsequence (LCS)

**Problem:** Find the length of the longest subsequence common to two strings.

```
s1 = "ABCBDAB"
s2 = "BDCAB"
LCS = "BCAB" → length 4
```

```python
def lcs(s1, s2):
    m, n = len(s1), len(s2)
    dp = [[0] * (n + 1) for _ in range(m + 1)]

    for i in range(1, m + 1):
        for j in range(1, n + 1):
            if s1[i-1] == s2[j-1]:
                dp[i][j] = dp[i-1][j-1] + 1       # characters match
            else:
                dp[i][j] = max(dp[i-1][j], dp[i][j-1])  # take best without one

    return dp[m][n]

print(lcs("ABCBDAB", "BDCAB"))   # 4
```

```
DP Table (partial):
       ""  B  D  C  A  B
    "" [0, 0, 0, 0, 0, 0]
    A  [0, 0, 0, 0, 1, 1]
    B  [0, 1, 1, 1, 1, 2]
    C  [0, 1, 1, 2, 2, 2]
    B  [0, 1, 1, 2, 2, 3]
    D  [0, 1, 2, 2, 2, 3]
    A  [0, 1, 2, 2, 3, 3]
    B  [0, 1, 2, 2, 3, 4]  ← answer
```

---

## Classic Problem 3 — Coin Change

**Problem:** Given coin denominations and a target amount, find the minimum number of coins needed.

```
coins  = [1, 5, 6, 9]
amount = 11
Answer = 2  (5+6)
```

```python
def coin_change(coins, amount):
    dp = [float('inf')] * (amount + 1)
    dp[0] = 0                              # base case: 0 coins to make amount 0

    for amt in range(1, amount + 1):
        for coin in coins:
            if coin <= amt:
                dp[amt] = min(dp[amt], dp[amt - coin] + 1)

    return dp[amount] if dp[amount] != float('inf') else -1


print(coin_change([1, 5, 6, 9], 11))   # 2
```

```
dp[0] = 0
dp[1] = dp[0]+1 = 1   (coin 1)
dp[5] = dp[0]+1 = 1   (coin 5)
dp[6] = dp[0]+1 = 1   (coin 6)
dp[9] = dp[0]+1 = 1   (coin 9)
dp[10]= dp[5]+1 = 2   (coin 5)
dp[11]= dp[5]+1 = 2   (coin 6, dp[5]=1) ← answer
```

---

## Classic Problem 4 — Longest Increasing Subsequence (LIS)

```python
def lis(nums):
    n = len(nums)
    dp = [1] * n   # every element is an LIS of length 1 by itself

    for i in range(1, n):
        for j in range(i):
            if nums[j] < nums[i]:
                dp[i] = max(dp[i], dp[j] + 1)

    return max(dp)

print(lis([10, 9, 2, 5, 3, 7, 101, 18]))   # 4  → [2,3,7,18] or [2,5,7,101]
```

> **O(n log n) variant** exists using binary search + patience sorting.

---

## Classic Problem 5 — Edit Distance (Levenshtein)

**Problem:** Minimum operations (insert, delete, replace) to convert word1 to word2.

```python
def edit_distance(w1, w2):
    m, n = len(w1), len(w2)
    dp = [[0] * (n+1) for _ in range(m+1)]

    for i in range(m+1): dp[i][0] = i    # delete all chars of w1
    for j in range(n+1): dp[0][j] = j    # insert all chars of w2

    for i in range(1, m+1):
        for j in range(1, n+1):
            if w1[i-1] == w2[j-1]:
                dp[i][j] = dp[i-1][j-1]                            # no operation
            else:
                dp[i][j] = 1 + min(dp[i-1][j],   # delete from w1
                                   dp[i][j-1],   # insert into w1
                                   dp[i-1][j-1]) # replace

    return dp[m][n]

print(edit_distance("horse", "ros"))   # 3
```

---

## DP Patterns Cheat Sheet

```
Pattern               Problem Type                     Example
─────────────────────────────────────────────────────────────────────────
Linear DP             1D array, each state uses prev   Fibonacci, Coin Change
Kadane's Algorithm    Max subarray sum                 Max profit, max subarray
2D Grid DP            Move in a grid (right/down)      Unique paths, min path sum
Knapsack DP           Pick items with constraints      0/1 knapsack, subset sum
Interval DP           Solve over intervals [i..j]      Matrix chain, burst balloons
String DP             Two string comparison            LCS, Edit Distance, LPS
Tree DP               DP on tree nodes                 Max path sum, house robber III
Bitmask DP            State = subset of elements       TSP, assignment problems
```

---

## Complexity

| Problem                | Time        | Space      |
|------------------------|-------------|------------|
| Fibonacci              | O(n)        | O(1) opt   |
| 0/1 Knapsack           | O(n × W)    | O(W) opt   |
| LCS                    | O(m × n)    | O(min(m,n))|
| Coin Change            | O(n × amt)  | O(amt)     |
| LIS                    | O(n²) / O(n log n) | O(n) |
| Edit Distance          | O(m × n)    | O(min(m,n))|

---

## Real-World Uses

```
1. Spell checkers         — Edit distance for nearest word suggestions
2. DNA sequence alignment — LCS applied to genome strings (bioinformatics)
3. Git diff               — LCS to find changed lines between commits
4. Resource scheduling    — Knapsack-style allocation of jobs/bandwidth
5. Natural language       — Viterbi algorithm (HMM decoding) uses DP
6. Game AI                — Minimax with memoization (chess, checkers)
7. Route optimisation     — Bellman-Ford shortest path is pure DP
8. Compiler optimisation  — Register allocation, instruction scheduling
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  DYNAMIC PROGRAMMING — AT A GLANCE                                │
│                                                                   │
│  Apply when: overlapping subproblems + optimal substructure      │
│                                                                   │
│  Top-Down:    recursive + memo dict     — easier to derive       │
│  Bottom-Up:   iterative + dp table     — faster, no stack risk   │
│                                                                   │
│  Framework: define dp[i] → recurrence → base cases → order → ans│
│                                                                   │
│  Classic problems:                                                │
│    Fibonacci · Knapsack · LCS · Coin Change · LIS · Edit Distance│
│                                                                   │
│  Used in: spell check, Git diff, DNA alignment, game AI, routing │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 19 of daily algorithm commits — next up: Dijkstra's Algorithm*
