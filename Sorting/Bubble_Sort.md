# 🫧 Bubble Sort

> *"The simplest sorting algorithm — and the best one for building intuition."*

---
![Python](https://img.shields.io/badge/Language-Python-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Beginner-green)
![Category](https://img.shields.io/badge/Category-Sorting-orange)   

## What is Bubble Sort?

Bubble Sort is a comparison-based sorting algorithm that repeatedly walks through an array, compares adjacent elements, and swaps them if they're in the wrong order. After each full pass, the largest unsorted element "bubbles up" to its correct position at the end.

The algorithm keeps making passes until no swaps are needed — which means the array is sorted.

---

## The Problem

**Input:** An unsorted array of numbers

```
[5, 3, 8, 4, 2]
```

**Output:** The same elements in ascending order

```
[2, 3, 4, 5, 8]
```

---

## Visualising the Algorithm

Here's what happens step-by-step. Elements being compared are marked with `^`, and sorted positions are marked `✓`:

```
Initial:  [ 5   3   8   4   2 ]

── Pass 1 ──────────────────────────────────────────────
          [ 5   3   8   4   2 ]
             ^   ^              compare 5,3 → swap
          [ 3   5   8   4   2 ]
                 ^   ^          compare 5,8 → ok
          [ 3   5   8   4   2 ]
                     ^   ^      compare 8,4 → swap
          [ 3   5   4   8   2 ]
                         ^   ^  compare 8,2 → swap
          [ 3   5   4   2   8 ]
                             ✓  8 is in place

── Pass 2 ──────────────────────────────────────────────
          [ 3   5   4   2   8 ]
             ^   ^              compare 3,5 → ok
          [ 3   5   4   2   8 ]
                 ^   ^          compare 5,4 → swap
          [ 3   4   5   2   8 ]
                     ^   ^      compare 5,2 → swap
          [ 3   4   2   5   8 ]
                         ✓   ✓  5 and 8 in place

── Pass 3 ──────────────────────────────────────────────
          [ 3   4   2   5   8 ]
             ^   ^              compare 3,4 → ok
          [ 3   4   2   5   8 ]
                 ^   ^          compare 4,2 → swap
          [ 3   2   4   5   8 ]
                     ✓   ✓   ✓  4, 5, 8 in place

── Pass 4 ──────────────────────────────────────────────
          [ 3   2   4   5   8 ]
             ^   ^              compare 3,2 → swap
          [ 2   3   4   5   8 ]
             ✓   ✓   ✓   ✓   ✓  Array fully sorted! ✅
```

> **Key insight:** After each pass, the largest unsorted element is guaranteed to be in its final position. So each pass needs one fewer comparison than the last.

---

## The Algorithm (5 Steps)

```
1.  Start at the beginning of the array
2.  Compare the current element with its right neighbour
3.  If left > right → swap them
4.  Move one position right and repeat
5.  After each full pass, the last unsorted element is settled
    → Repeat until no swaps occur in a pass
```

---

## Pseudocode

```
for i = 0 to n-1:
    swapped = false
    for j = 0 to n-i-2:
        if arr[j] > arr[j+1]:
            swap(arr[j], arr[j+1])
            swapped = true
    if not swapped:
        break          ← early exit if already sorted
```

---

## Python Implementation

```python
def bubble_sort(arr):
    n = len(arr)

    for i in range(n):
        swapped = False

        for j in range(0, n - i - 1):
            if arr[j] > arr[j + 1]:
                arr[j], arr[j + 1] = arr[j + 1], arr[j]
                swapped = True

        if not swapped:   # ← optimisation: stop early if sorted
            break

    return arr


# Example
arr = [5, 3, 8, 4, 2]
print("Before:", arr)
print("After: ", bubble_sort(arr))
# After:  [2, 3, 4, 5, 8]
```

> **Optimisation:** The `swapped` flag lets us detect an already-sorted array in a single pass — achieving **O(n)** best-case time.

---

## Complexity

### Time

| Case         | Complexity | When it happens                              |
|--------------|-----------|----------------------------------------------|
| Best case    | **O(n)**  | Array is already sorted (swapped flag kicks in) |
| Average case | **O(n²)** | Elements are in random order                 |
| Worst case   | **O(n²)** | Array is sorted in reverse                   |

### Space

| Metric         | Complexity | Why                                         |
|----------------|-----------|----------------------------------------------|
| Auxiliary space | **O(1)** | Swaps happen in-place, no extra array needed |

---

## Properties at a Glance

| Property         | Value  | Notes                                      |
|------------------|--------|--------------------------------------------|
| Stable           | ✅ Yes  | Equal elements keep their relative order |
| In-Place         | ✅ Yes  | No extra memory needed                    |
| Adaptive         | ✅ Yes  | Optimised version exits early             |
| Comparison-Based | ✅ Yes  | Decisions made by comparing pairs         |
| Recursive        | ❌ No   | Iterative by nature                       |

---

## Pros and Cons

### ✅ Advantages
- Dead simple to understand and implement
- Requires zero extra memory
- Stable — safe for records where order matters
- Detects sorted arrays efficiently with the `swapped` flag
- Perfect for learning how sorting works

### ❌ Disadvantages
- Terrible performance on large inputs — O(n²)
- Enormous number of comparisons even for small gains
- Not used in production — Quick Sort, Merge Sort, and Tim Sort are all faster

---

## When to Use It

```
Bubble Sort is the right choice when:
  ✓ You're learning sorting fundamentals
  ✓ The dataset is tiny (n < 20 or so)
  ✓ You need something quick to implement and correctness > speed
  ✓ You want to detect whether an array is already sorted cheaply

Bubble Sort is the wrong choice when:
  ✗ You have more than a few hundred elements
  ✗ Performance matters
  ✗ You're building anything production-facing
```

---

## Common Interview Questions

**Q: Is Bubble Sort stable?**
Yes — equal elements are never swapped, so they maintain their original relative order.

**Q: Is it in-place?**
Yes — swaps happen directly in the array. Space complexity is O(1).

**Q: Why is it called "Bubble Sort"?**
After each pass, the largest unsorted value has "bubbled up" to its correct position at the end — just like a bubble rising to the surface.

**Q: Can Bubble Sort achieve O(n)?**
Yes — in the best case (already-sorted input), the optimised version exits after a single pass with zero swaps.

**Q: Is it used in industry?**
Rarely, if ever. In practice you'd reach for Tim Sort (Python's built-in), Merge Sort, or Quick Sort.

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│  BUBBLE SORT — AT A GLANCE                              │
│                                                         │
│  Strategy:   Compare adjacent pairs, swap if needed     │
│  Passes:     n-1 in worst case                          │
│                                                         │
│  Time  →  Best: O(n)  |  Avg/Worst: O(n²)               │
│  Space →  O(1) — in-place                               │
│                                                         │
│  Stable: Yes   Adaptive: Yes (with swapped flag)        │
│                                                         │
│  Use for: learning   Avoid for: real datasets           │
└─────────────────────────────────────────────────────────┘
```

---

*Day 1 of daily algorithm commits — next up: Selection Sort / Insertion Sort / Binary Search*