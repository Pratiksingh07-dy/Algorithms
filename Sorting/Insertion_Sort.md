# 🃏 Insertion Sort

> *"Like sorting cards in your hand — pick one up, slide it left until it fits."*

---

## What is Insertion Sort?

Insertion Sort builds a sorted array **one element at a time** by picking each new element and inserting it into its correct position within the already-sorted portion.

Think of sorting playing cards: you pick up one card at a time from the table and slide it into the right spot among the cards already in your hand. You never reshuffle the whole hand — you just make room and insert.

- Everything to the **left** of the current position → sorted
- Everything to the **right** → not yet touched
- The current element is "picked up" and shifted left until it finds its home

---

## The Problem

**Input:** An unsorted array

```
[12, 11, 13, 5, 6]
```

**Output:** Elements in ascending order

```
[5, 6, 11, 12, 13]
```

---

## Visualising the Algorithm

`[key]` = the element currently being inserted. `→` = element shifted right. Sorted region shown with `✓`.

```
Initial:  [ 12   11   13    5    6 ]

── Pass 1: insert arr[1] = 11 ────────────────────────────
  Pick up:  key = 11
  Compare:  11 < 12  →  shift 12 right
            [  _   12   13    5    6 ]   (gap opened)
  Insert:   [ 11   12   13    5    6 ]
              ✓    ✓

── Pass 2: insert arr[2] = 13 ────────────────────────────
  Pick up:  key = 13
  Compare:  13 ≥ 12  →  no shift needed
  Insert:   [ 11   12   13    5    6 ]   (already in place)
              ✓    ✓    ✓

── Pass 3: insert arr[3] = 5 ─────────────────────────────
  Pick up:  key = 5
  Compare:  5 < 13  →  shift 13 right
            [ 11   12    _   13    6 ]
  Compare:  5 < 12  →  shift 12 right
            [ 11    _   12   13    6 ]
  Compare:  5 < 11  →  shift 11 right
            [  _   11   12   13    6 ]
  Insert:   [  5   11   12   13    6 ]
              ✓    ✓    ✓    ✓

── Pass 4: insert arr[4] = 6 ─────────────────────────────
  Pick up:  key = 6
  Compare:  6 < 13  →  shift 13 right
            [  5   11   12    _   13 ]
  Compare:  6 < 12  →  shift 12 right
            [  5   11    _   12   13 ]
  Compare:  6 < 11  →  shift 11 right
            [  5    _   11   12   13 ]
  Compare:  6 ≥ 5   →  stop
  Insert:   [  5    6   11   12   13 ]
              ✓    ✓    ✓    ✓    ✓   Array fully sorted! ✅
```

> **Key insight:** Insertion Sort only moves elements as far as necessary. On a nearly-sorted array, almost no shifting happens — this is why it achieves **O(n)** in the best case and is the algorithm of choice for small or nearly-sorted data.

---

## The Algorithm (5 Steps)

```
1.  Start from index 1 (the first element is trivially "sorted")
2.  Pick up the current element — call it the key
3.  Compare the key with each element to its left
4.  Shift elements right as long as they are greater than the key
5.  Drop the key into the gap that opened up
    → Move to the next index and repeat
```

---

## Pseudocode

```
for i = 1 to n-1:
    key = arr[i]          ← pick up the element
    j = i - 1

    while j >= 0 and arr[j] > key:
        arr[j+1] = arr[j]  ← shift element right
        j = j - 1

    arr[j+1] = key         ← drop into the gap
```

---

## Python Implementation

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]          # pick up the current element
        j = i - 1

        # shift elements right while they are greater than key
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1

        arr[j + 1] = key      # drop key into its correct position

    return arr


# Example
arr = [12, 11, 13, 5, 6]
print("Before:", arr)
print("After: ", insertion_sort(arr))
# After:  [5, 6, 11, 12, 13]
```

> **Note:** The inner `while` loop exits early the moment it finds an element ≤ key. On partially sorted input this means very few shifts — this is what makes Insertion Sort adaptive.

---

## Complexity

### Time

| Case         | Complexity | When it happens                                         |
|--------------|-----------|----------------------------------------------------------|
| Best case    | **O(n)**  | Array is already sorted — inner loop never runs         |
| Average case | **O(n²)** | Random order — roughly half the elements shift each pass |
| Worst case   | **O(n²)** | Array is reverse sorted — every element shifts all the way left |

### Space

| Metric          | Complexity | Why                          |
|-----------------|-----------|-------------------------------|
| Auxiliary space | **O(1)**  | Sorted in-place; only `key` and `j` as temp variables |

---

## Properties at a Glance

| Property         | Value   | Notes                                                     |
|------------------|---------|-----------------------------------------------------------|
| Stable           | ✅ Yes  | Equal elements are never swapped past each other          |
| In-Place         | ✅ Yes  | No additional data structure needed                       |
| Adaptive         | ✅ Yes  | Fewer operations on partially sorted input                |
| Online           | ✅ Yes  | Can sort a stream — process elements one at a time        |
| Comparison-Based | ✅ Yes  | Uses comparisons to find the correct position             |
| Recursive        | ❌ No   | Iterative (but a recursive version exists)                |

> **Online algorithm** — a unique property. Insertion Sort can process incoming elements one at a time without needing the full array upfront. Bubble Sort and Selection Sort cannot do this.

---

## The Big Three Compared

| Feature              | Bubble Sort      | Selection Sort   | Insertion Sort         |
|----------------------|------------------|------------------|------------------------|
| Best case            | O(n)             | O(n²)            | **O(n)**               |
| Average case         | O(n²)            | O(n²)            | O(n²)                  |
| Worst case           | O(n²)            | O(n²)            | O(n²)                  |
| Swaps / writes       | Up to O(n²)      | At most n-1      | Up to O(n²) shifts     |
| Stable               | ✅ Yes           | ❌ No            | ✅ Yes                  |
| Adaptive             | ✅ Yes           | ❌ No            | ✅ Yes                  |
| Online               | ❌ No            | ❌ No            | ✅ Yes                  |
| Best real-world use  | Teaching / tiny  | Min writes       | **Nearly sorted data** |

> **Winner for practical small-array use:** Insertion Sort — it's adaptive, stable, online, and outperforms the others on nearly sorted data. This is exactly why Python's Tim Sort and Java's Arrays.sort use Insertion Sort internally for small sub-arrays.

---

## Pros and Cons

### ✅ Advantages
- Simple to implement and easy to reason about
- **O(n) best case** — blazing fast on nearly sorted data
- Stable and in-place
- **Online** — works as elements arrive, no full array needed
- Excellent for small arrays (n < 20–30)
- Used internally by Tim Sort and Intro Sort as the base case

### ❌ Disadvantages
- O(n²) average and worst case — slow on large random arrays
- More shifts than Selection Sort's swaps in the worst case
- Not suitable for large unsorted datasets on its own

---

## When to Use It

```
Insertion Sort is the right choice when:
  ✓ The array is small (n ≤ ~30)
  ✓ The data is nearly sorted (few inversions)
  ✓ You need a stable sort with O(1) space
  ✓ Elements arrive one at a time (online scenario)
  ✓ You're implementing the base case of a hybrid sorter

Insertion Sort is the wrong choice when:
  ✗ The dataset is large and randomly ordered
  ✗ You need guaranteed O(n log n) — use Merge Sort or Heap Sort
```

---

## Common Interview Questions

**Q: Is Insertion Sort stable?**
Yes — equal elements are never moved past each other. The condition `arr[j] > key` (strict greater-than) ensures equal elements are not shifted.

**Q: What makes Insertion Sort adaptive?**
The inner while loop exits as soon as it finds an element ≤ the key. If the array is already sorted, no shifts happen — just one comparison per pass, giving O(n) total.

**Q: What does "online algorithm" mean?**
It means Insertion Sort can process elements as they arrive, one by one, maintaining a sorted list at every step — without needing the full input upfront. Selection Sort and Bubble Sort need the full array first.

**Q: How does Insertion Sort compare to Selection Sort in terms of writes?**
Insertion Sort shifts multiple elements per pass (potentially O(n²) total writes). Selection Sort makes at most n-1 swaps. So if writes are expensive (e.g. flash memory), Selection Sort is better — but if the input is nearly sorted, Insertion Sort's early-exit makes it far fewer writes in practice.

**Q: Why do real-world sorting algorithms use Insertion Sort?**
Tim Sort (Python, Java) and Intro Sort (C++ STL) switch to Insertion Sort for small sub-arrays because the overhead of recursive divide-and-conquer algorithms outweighs their asymptotic advantage at small n. Insertion Sort's cache-friendliness and low constant factor make it the fastest in practice for n ≤ ~32.

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────┐
│  INSERTION SORT — AT A GLANCE                           │
│                                                         │
│  Strategy:   Pick, shift left, drop into gap            │
│  Passes:     n-1                                        │
│                                                         │
│  Time  →  Best: O(n)  |  Avg/Worst: O(n²)               │
│  Space →  O(1) — in-place                               │
│                                                         │
│  Stable: Yes   Adaptive: Yes   Online: Yes              │
│                                                         │
│  Use for: small / nearly sorted arrays, streaming data  │
│  Used by: Tim Sort, Intro Sort as the base-case sorter  │
└─────────────────────────────────────────────────────────┘
```

---

*Day 2 of daily algorithm commits — next up: Merge Sort*