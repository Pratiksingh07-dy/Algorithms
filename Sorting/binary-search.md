# 🔍 Binary Search

> *"Why check every element when you can eliminate half the array with a single comparison?"*

---

## What is Binary Search?

Binary Search is a **search algorithm** that finds the position of a target value in a **sorted array** by repeatedly halving the search space.

At each step it looks at the middle element:
- If it equals the target → found, return the index
- If it's less than the target → the target must be in the right half
- If it's greater than the target → the target must be the left half

Each comparison eliminates half the remaining elements. An array of 1,000,000 elements needs at most **20 comparisons**. That's the power of O(log n).

> **Critical prerequisite:** The array must be sorted. Binary Search on an unsorted array produces wrong answers, not just slow ones.

---

## The Problem

**Input:** A sorted array and a target value

```
arr    = [2, 5, 8, 12, 16, 23, 38, 45, 67, 91]
target = 23
```

**Output:** Index of the target, or -1 if not found

```
4   (arr[5] = 23)  ← 0-indexed: answer is index 5
```

---

## Visualising Binary Search

```
arr = [2, 5, 8, 12, 16, 23, 38, 45, 67, 91]
idx =  0  1  2   3   4   5   6   7   8   9
target = 23

── Step 1 ────────────────────────────────────────────────
lo=0, hi=9, mid=4  →  arr[4]=16
16 < 23  →  target is in the RIGHT half
[ 2   5   8  12  16 | 23  38  45  67  91 ]
                  ^              ^^^^^^^^
                 mid          search here

── Step 2 ────────────────────────────────────────────────
lo=5, hi=9, mid=7  →  arr[7]=45
45 > 23  →  target is in the LEFT half
[ 23  38  45  45  67  91 ]
      ^    ^
   search  mid

── Step 3 ────────────────────────────────────────────────
lo=5, hi=6, mid=5  →  arr[5]=23
23 == 23  →  FOUND at index 5 ✅

Total comparisons: 3  (vs 10 for linear search)
```

---

## Trace Table

| Step | lo | hi | mid | arr[mid] | Decision          |
|------|----|----|-----|----------|-------------------|
| 1    | 0  | 9  | 4   | 16       | 16 < 23 → go right |
| 2    | 5  | 9  | 7   | 45       | 45 > 23 → go left  |
| 3    | 5  | 6  | 5   | 23       | 23 == 23 → **found!** |

---

## The Algorithm

```
1.  Set lo = 0, hi = n - 1
2.  While lo ≤ hi:
      a. Compute mid = (lo + hi) // 2
      b. If arr[mid] == target → return mid
      c. If arr[mid] < target  → lo = mid + 1  (search right half)
      d. If arr[mid] > target  → hi = mid - 1  (search left half)
3.  Return -1  (target not found)
```

---

## Pseudocode

```
function binary_search(arr, target):
    lo = 0
    hi = len(arr) - 1

    while lo <= hi:
        mid = (lo + hi) // 2       ← integer division

        if arr[mid] == target:
            return mid              ← found
        elif arr[mid] < target:
            lo = mid + 1            ← discard left half
        else:
            hi = mid - 1            ← discard right half

    return -1                       ← not found
```

---

## Python Implementation

```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1

    while lo <= hi:
        mid = (lo + hi) // 2        # safe in Python (no integer overflow)

        if arr[mid] == target:
            return mid
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1

    return -1


# Example
arr = [2, 5, 8, 12, 16, 23, 38, 45, 67, 91]
print(binary_search(arr, 23))   # → 5
print(binary_search(arr, 10))   # → -1
```

### Recursive version

```python
def binary_search_recursive(arr, target, lo=0, hi=None):
    if hi is None:
        hi = len(arr) - 1

    if lo > hi:
        return -1                   # base case: not found

    mid = (lo + hi) // 2

    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search_recursive(arr, target, mid + 1, hi)
    else:
        return binary_search_recursive(arr, target, lo, mid - 1)
```

> **Prefer iterative over recursive** — the recursive version uses O(log n) call stack space for no benefit. The iterative version is O(1) space.

---

## The Integer Overflow Bug

A classic subtle bug when computing the midpoint:

```python
# ❌ Wrong in languages with fixed-size integers (C, Java, C++)
mid = (lo + hi) // 2    # lo + hi can overflow if both are large

# ✅ Safe in all languages
mid = lo + (hi - lo) // 2
```

In Python integers don't overflow, so `(lo + hi) // 2` is fine. In C/Java/C++ — always use `lo + (hi - lo) // 2`.

---

## Why O(log n)?

```
Array size:   n = 1,024

After step 1: n/2   =  512 elements remain
After step 2: n/4   =  256 elements remain
After step 3: n/8   =  128 elements remain
...
After step k: n/2^k =    1 element remains

Solve n/2^k = 1  →  k = log₂(n)

For n = 1,024   →  k = 10 comparisons
For n = 1M      →  k = 20 comparisons
For n = 1B      →  k = 30 comparisons
```

This is why Binary Search is described as "absurdly fast" — doubling the array size adds just **one** comparison.

---

## Complexity

### Time

| Case         | Complexity   | When it happens                          |
|--------------|-------------|------------------------------------------|
| Best case    | **O(1)**    | Target is exactly at the midpoint        |
| Average case | **O(log n)**| Target found after several halvings      |
| Worst case   | **O(log n)**| Target not present, or at extreme end    |

### Space

| Version    | Complexity   | Why                                    |
|------------|-------------|----------------------------------------|
| Iterative  | **O(1)**    | Only `lo`, `hi`, `mid` variables       |
| Recursive  | **O(log n)**| Call stack depth = number of halvings  |

---

## Properties at a Glance

| Property             | Value   | Notes                                         |
|----------------------|---------|-----------------------------------------------|
| Requires sorted array | ✅ Yes | Will give wrong results on unsorted input     |
| In-Place             | ✅ Yes  | No auxiliary data structure                   |
| Comparison-Based     | ✅ Yes  | Decisions made by `<`, `>`, `==`              |
| Works on linked lists | ❌ No  | Needs O(1) random access — arrays only        |

---

## Variants

### 1. Find first occurrence (leftmost)

When the array has duplicates, standard binary search returns an arbitrary match. To find the first:

```python
def first_occurrence(arr, target):
    lo, hi, result = 0, len(arr) - 1, -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            result = mid
            hi = mid - 1       # ← keep searching LEFT even after a match
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return result
```

### 2. Find last occurrence (rightmost)

```python
def last_occurrence(arr, target):
    lo, hi, result = 0, len(arr) - 1, -1
    while lo <= hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] == target:
            result = mid
            lo = mid + 1       # ← keep searching RIGHT even after a match
        elif arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid - 1
    return result
```

### 3. Find insertion point (lower bound)

Find the leftmost index where target could be inserted to keep sorted order:

```python
def lower_bound(arr, target):
    lo, hi = 0, len(arr)
    while lo < hi:
        mid = lo + (hi - lo) // 2
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid
    return lo
```

> Python's `bisect` module provides `bisect_left` (lower bound) and `bisect_right` (upper bound) as built-ins — use them in production.

---

## Binary Search vs Linear Search

| Feature              | Linear Search    | Binary Search      |
|----------------------|------------------|--------------------|
| Requires sorted array | ❌ No           | ✅ Yes             |
| Time complexity      | O(n)             | O(log n)           |
| Space complexity     | O(1)             | O(1)               |
| Works on linked list | ✅ Yes           | ❌ No              |
| Best for             | Small / unsorted | Large sorted arrays |

```
n = 1,000,000 elements:
  Linear search  → up to 1,000,000 comparisons
  Binary search  → at most 20 comparisons
```

---

## Real-World Uses

```
1. Dictionary / phone book lookup     — classic binary search scenario
2. Database index lookup              — B-trees use binary search at each node
3. Git bisect                         — find the commit that introduced a bug
4. Numeric root finding               — binary search on a continuous function
5. Finding a threshold                — "first day temperature > 30°" in sorted data
6. Word autocomplete / spell check    — searching sorted word lists
7. IP routing tables                  — longest prefix match via binary search
8. Debugging segfaults                — comment out half the code, test, repeat
```

---

## Pros and Cons

### ✅ Advantages
- Extremely fast — O(log n) makes it practical on millions of elements
- Simple to implement
- O(1) space (iterative version)
- The foundation of countless advanced algorithms and data structures

### ❌ Disadvantages
- Requires a **sorted array** — if data is unsorted, you must sort first (O(n log n))
- Only works on data structures with O(1) random access (arrays, not linked lists)
- Finding the midpoint of a sorted linked list is O(n), so Binary Search doesn't help
- Edge cases (duplicates, insertion points) require careful variant implementation

---

## When to Use It

```
Binary Search is the right choice when:
  ✓ The array is already sorted (or you can afford to sort it once)
  ✓ You need to search repeatedly — sort once, search many times in O(log n)
  ✓ The array is large (n > ~20) — logarithmic gain becomes significant
  ✓ You need first/last occurrence or an insertion point

Binary Search is the wrong choice when:
  ✗ The array is unsorted and you only search once → just use Linear Search
  ✗ The data structure is a linked list → no O(1) random access
  ✗ Data changes frequently → re-sorting cost may outweigh search savings
```

---

## Common Interview Questions

**Q: What happens if you run Binary Search on an unsorted array?**
It produces incorrect results — potentially returning the wrong index or -1 even when the target is present. There is no error; it just silently gives wrong answers.

**Q: Why is `mid = lo + (hi - lo) // 2` safer than `(lo + hi) // 2`?**
In languages with fixed-size integers (C, Java), `lo + hi` can overflow if both values are near `INT_MAX`. `lo + (hi - lo) // 2` avoids the intermediate sum exceeding the integer limit. Not an issue in Python.

**Q: How do you find the first or last occurrence of a duplicate value?**
After finding a match, don't return immediately — record the index and continue searching left (for first) or right (for last) by adjusting `hi = mid - 1` or `lo = mid + 1`.

**Q: Can Binary Search be applied to non-integer search spaces?**
Yes. Any monotonic function can be binary searched. "Find the minimum speed to finish a task in k hours" — binary search over the speed space. This pattern (binary search on the answer) is a common hard LeetCode technique.

**Q: How many comparisons does Binary Search need for n = 1,000,000?**
At most ⌈log₂(1,000,000)⌉ = 20 comparisons.

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  BINARY SEARCH — AT A GLANCE                                      │
│                                                                   │
│  Strategy:  Eliminate half the search space each step             │
│  Prerequisite: array must be sorted                               │
│                                                                   │
│  Time  →  O(log n) — halves the problem each comparison          │
│  Space →  O(1) iterative  |  O(log n) recursive                  │
│                                                                   │
│  Variants: first occurrence, last occurrence, lower bound         │
│  Pitfall: integer overflow in mid calculation (use lo+(hi-lo)//2) │
│                                                                   │
│  1M elements → 20 comparisons. 1B elements → 30 comparisons.     │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 7 of daily algorithm commits — next up: Linear Search*
