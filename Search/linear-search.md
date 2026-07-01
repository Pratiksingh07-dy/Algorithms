# 🔦 Linear Search

> *"The simplest search algorithm — and the only one that works on unsorted data."*

---

## What is Linear Search?

Linear Search (also called **Sequential Search**) scans through an array element by element from the beginning until it finds the target — or reaches the end.

No tricks. No prerequisites. No sorted order required. Just check each element, one by one.

It's the algorithm you use every time you scan a list with your eyes. For small arrays or unsorted data it's perfectly valid — and sometimes the only option.

---

## The Problem

**Input:** An array (sorted or unsorted) and a target value

```
arr    = [64, 34, 25, 12, 22, 11, 90]
target = 22
```

**Output:** Index of the target, or -1 if not found

```
4   (arr[4] = 22)
```

---

## Visualising Linear Search

```
arr    = [64, 34, 25, 12, 22, 11, 90]
target = 22

Step 1: arr[0] = 64 → 64 ≠ 22 → continue
        [64*  34   25   12   22   11   90]
          ^

Step 2: arr[1] = 34 → 34 ≠ 22 → continue
        [64   34*  25   12   22   11   90]
               ^

Step 3: arr[2] = 25 → 25 ≠ 22 → continue
        [64   34   25*  12   22   11   90]
                    ^

Step 4: arr[3] = 12 → 12 ≠ 22 → continue
        [64   34   25   12*  22   11   90]
                         ^

Step 5: arr[4] = 22 → 22 == 22 → FOUND at index 4 ✅
        [64   34   25   12   22*  11   90]
                          ^
```

---

## The Algorithm

```
1.  Start at index 0
2.  Compare current element with target
3.  If equal → return current index
4.  Move to next element
5.  Repeat until end of array
6.  If not found → return -1
```

---

## Pseudocode

```
function linear_search(arr, target):
    for i = 0 to len(arr) - 1:
        if arr[i] == target:
            return i         ← found, return index
    return -1                ← not found
```

---

## Python Implementation

```python
def linear_search(arr, target):
    for i, val in enumerate(arr):
        if val == target:
            return i
    return -1


# Example
arr = [64, 34, 25, 12, 22, 11, 90]
print(linear_search(arr, 22))   # → 4
print(linear_search(arr, 99))   # → -1
```

### Find all occurrences

```python
def linear_search_all(arr, target):
    return [i for i, val in enumerate(arr) if val == target]

arr = [5, 3, 7, 3, 8, 3]
print(linear_search_all(arr, 3))  # → [1, 3, 5]
```

### Search with a condition (predicate search)

```python
def linear_search_if(arr, predicate):
    for i, val in enumerate(arr):
        if predicate(val):
            return i
    return -1

# Find first even number
arr = [7, 3, 5, 8, 11, 6]
print(linear_search_if(arr, lambda x: x % 2 == 0))  # → 3
```

---

## Optimisation: Sentinel Search

A subtle but effective trick — append the target to the end of the array before searching. This eliminates the bounds check (`i < n`) from every iteration, reducing the number of comparisons per step from 2 to 1.

```python
def sentinel_search(arr, target):
    n = len(arr)
    arr.append(target)          # place sentinel at the end
    i = 0
    while arr[i] != target:
        i += 1
    arr.pop()                   # restore original array

    if i < n:
        return i                # found within original bounds
    return -1                   # only the sentinel was hit


# Example
arr = [64, 34, 25, 12, 22, 11, 90]
print(sentinel_search(arr, 22))   # → 4
print(sentinel_search(arr, 99))   # → -1
```

```
Normal loop:    while i < n and arr[i] != target:  ← 2 checks per step
Sentinel loop:  while arr[i] != target:            ← 1 check per step (sentinel guarantees termination)
```

The difference is small but measurable in tight inner loops over large arrays.

---

## Optimisation: Move to Front (MTF)

For arrays that are searched repeatedly, move each found element to the front. Frequently searched items rise to the top over time — turning a slow worst-case into a fast average-case on access patterns with hot elements.

```python
def linear_search_mtf(arr, target):
    for i in range(len(arr)):
        if arr[i] == target:
            arr.insert(0, arr.pop(i))   # move to front
            return 0
    return -1
```

---

## Complexity

### Time

| Case         | Complexity | When it happens                       |
|--------------|-----------|---------------------------------------|
| Best case    | **O(1)**  | Target is the first element           |
| Average case | **O(n)**  | Target is somewhere in the middle     |
| Worst case   | **O(n)**  | Target is last, or not present at all |

### Space

| Version        | Complexity | Why                         |
|----------------|-----------|------------------------------|
| Standard       | **O(1)**  | Only loop index variable     |
| Find all       | **O(k)**  | k = number of matches        |

---

## Properties at a Glance

| Property              | Value   | Notes                                       |
|-----------------------|---------|---------------------------------------------|
| Requires sorted array | ❌ No  | Works on any ordering                       |
| Works on linked lists | ✅ Yes | No random access needed                     |
| In-Place              | ✅ Yes | No extra memory                             |
| Can find duplicates   | ✅ Yes | Scan all / collect all variants             |
| Supports predicates   | ✅ Yes | Any condition, not just equality            |

---

## Linear Search vs Binary Search

| Feature               | Linear Search   | Binary Search      |
|-----------------------|-----------------|--------------------|
| Requires sorted array | ❌ No           | ✅ Yes             |
| Time (worst case)     | O(n)            | O(log n)           |
| Works on linked lists | ✅ Yes          | ❌ No              |
| Find first/last dup   | ✅ Easy         | Needs variant      |
| Search by condition   | ✅ Any predicate| ❌ Equality only   |
| Best for              | Small / unsorted| Large sorted arrays |

```
Break-even point (roughly):
  n ≤ ~20       → Linear Search is competitive (cache advantage, no sort cost)
  n > ~20       → Binary Search wins if array is sorted
  One-off search → sort cost (O(n log n)) often outweighs the search gain
  Many searches  → sort once, binary search every time
```

---

## Real-World Uses

```
1. Finding an item in a small list   — overhead of sorting > savings of binary search
2. Searching unsorted data           — the only option without sorting first
3. Linked list search                — no random access; must scan sequentially
4. Database full table scans         — when no index exists, sequential scan is used
5. Pattern matching in strings       — naive substring search is linear
6. Searching objects by property     — predicate search: find first user with age > 30
7. Finding max/min                   — scan once, track best seen (O(n) unavoidable)
8. First/last element matching rule  — log files, event streams
```

---

## Pros and Cons

### ✅ Advantages
- Works on **unsorted** arrays — no preprocessing needed
- Works on **linked lists** and any sequential data structure
- Simple to implement and reason about
- Can search by **any condition** (predicate), not just equality
- Finds **all occurrences** easily
- Excellent for **small arrays** where cache effects dominate

### ❌ Disadvantages
- O(n) time — impractical for large sorted datasets
- Every element must be visited in the worst case
- No improvement possible without changing the data structure (sorting or indexing)

---

## When to Use It

```
Linear Search is the right choice when:
  ✓ The array is unsorted and sorting isn't worth it
  ✓ The array is small (n ≤ ~20)
  ✓ You're searching a linked list
  ✓ You need to search by a custom condition (predicate)
  ✓ You need all occurrences, not just one
  ✓ It's a one-off search (sorting cost > search savings)

Linear Search is the wrong choice when:
  ✗ The array is large and sorted → use Binary Search
  ✗ You're searching the same array repeatedly → sort once, binary search
  ✗ The data is in a hash table → O(1) lookup is better
```

---

## Common Interview Questions

**Q: When is Linear Search better than Binary Search?**
When the array is unsorted (sorting costs O(n log n), so Binary Search only pays off with multiple searches), when the array is small (cache effects favour sequential access), or when searching a linked list where random access is O(n) anyway.

**Q: What is the time complexity of finding the minimum element in an unsorted array?**
O(n) — you must scan every element. There's no way around it without sorting or indexing first.

**Q: Can Linear Search be used on a 2D array?**
Yes — flatten the traversal: scan row by row, column by column. Time complexity is O(m × n) where m and n are the dimensions.

**Q: What is sentinel search and why is it faster?**
A sentinel is a copy of the target placed at the end of the array before searching. It guarantees the loop terminates without a bounds check, reducing comparisons per iteration from 2 to 1. On large arrays in tight loops, this gives a measurable speedup.

**Q: How do you find the second largest element using Linear Search?**
One pass, two tracked variables:
```python
def second_largest(arr):
    first = second = float('-inf')
    for x in arr:
        if x > first:
            second, first = first, x
        elif x > second and x != first:
            second = x
    return second
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  LINEAR SEARCH — AT A GLANCE                                      │
│                                                                   │
│  Strategy:  Scan left to right, check each element               │
│  Prerequisite: none — works on any array                          │
│                                                                   │
│  Time  →  O(n) worst/average  |  O(1) best                      │
│  Space →  O(1)                                                   │
│                                                                   │
│  Works on: unsorted arrays, linked lists, custom predicates       │
│  Optimisations: sentinel search (fewer checks), move-to-front     │
│                                                                   │
│  Use for: small data, unsorted data, one-off searches             │
│  Avoid for: large sorted arrays → binary search is O(log n)       │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 8 of daily algorithm commits — next up: Jump Search*
