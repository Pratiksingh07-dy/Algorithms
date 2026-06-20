# ⚡ Quick Sort

> *"The fastest sorting algorithm in practice — and the one with the most ways to go wrong."*

---

## What is Quick Sort?

Quick Sort is a **divide and conquer** algorithm, like Merge Sort — but the strategy is flipped:

- **Merge Sort** splits blindly down the middle, then does all the work during the merge
- **Quick Sort** does all the work during the split (the **partition** step), then recurses with nothing left to merge

The key idea: pick an element called the **pivot**, rearrange the array so that every element smaller than the pivot ends up to its left, and every element larger ends up to its right — then the pivot is in its final sorted position forever. Recurse on both sides.

No merge step. No extra array. Just partitioning.

---

## The Problem

**Input:** An unsorted array

```
[10, 80, 30, 90, 40, 50, 70]
```

**Output:** Elements in ascending order

```
[10, 30, 40, 50, 70, 80, 90]
```

---

## The Partition Step (Lomuto Scheme)

This is the heart of Quick Sort. Choose the last element as pivot, then scan left to right:

```
Partition [10, 80, 30, 90, 40, 50, 70], pivot = 70 (index 6)

i = -1  (boundary between left zone and right zone)
j scans from index 0 to 5:

  j=0: arr[0]=10 ≤ 70  → i=0, no swap needed (i==j)
       [ 10 | 80  30  90  40  50 | 70 ]
         ✓

  j=1: arr[1]=80 > 70  → skip
       [ 10 | 80  30  90  40  50 | 70 ]

  j=2: arr[2]=30 ≤ 70  → i=1, swap arr[1] ↔ arr[2]
       [ 10  30 | 80  90  40  50 | 70 ]
              ✓

  j=3: arr[3]=90 > 70  → skip

  j=4: arr[4]=40 ≤ 70  → i=2, swap arr[2] ↔ arr[4]
       [ 10  30  40 | 90  80  50 | 70 ]
                  ✓

  j=5: arr[5]=50 ≤ 70  → i=3, swap arr[3] ↔ arr[5]
       [ 10  30  40  50 | 90  80 | 70 ]
                      ✓

Place pivot: swap arr[i+1] ↔ arr[6]  →  swap arr[4] ↔ arr[6]
       [ 10  30  40  50 | 70 | 90  80 ]
                         ✓ pivot in final position!

Left sub-array  [10, 30, 40, 50] → all < 70 ✓
Right sub-array [90, 80]         → all > 70 ✓
```

---

## Full Trace (Recursive)

```
Initial:  [10, 80, 30, 90, 40, 50, 70]

── Partition [0..6], pivot=70 ──────────────────────────────
  Result: [10, 30, 40, 50, 70, 90, 80]
                               ✓  pivot at index 4

  ┌── Recurse left:  [10, 30, 40, 50] (indices 0..3)
  │   Partition [0..3], pivot=50
  │   Result: [10, 30, 40, 50]  → pivot 50 at index 3
  │
  │   ┌── Recurse left:  [10, 30, 40] (0..2), pivot=40
  │   │   Result: [10, 30, 40]  → pivot 40 at index 2
  │   │   ┌── [10, 30] pivot=30 → [10, 30] ✓
  │   │   └── [] empty → done
  │   │
  │   └── Recurse right: [] empty → done
  │
  └── Recurse right: [90, 80] (5..6), pivot=80
      Result: [80, 90]  → pivot 80 at index 5  ✓

Final: [10, 30, 40, 50, 70, 80, 90] ✅
```

> **Key insight:** Every pivot call places exactly **one element** permanently. With n elements you need n partition calls — but each level of recursion does O(n) total work across all its partition calls, and there are O(log n) levels on average.

---

## The Algorithm

```
Divide:
  1. Choose a pivot element from the array
  2. Partition: rearrange so all elements < pivot are on the left,
                             all elements > pivot are on the right
  3. The pivot is now in its final sorted position

Conquer:
  4. Recursively apply Quick Sort to the left sub-array
  5. Recursively apply Quick Sort to the right sub-array
  (No merge step needed — everything is already in place)
```

---

## Pseudocode (Lomuto Partition)

```
function quick_sort(arr, lo, hi):
    if lo < hi:
        p = partition(arr, lo, hi)   ← partition returns pivot's final index
        quick_sort(arr, lo, p - 1)   ← recurse left
        quick_sort(arr, p + 1, hi)   ← recurse right


function partition(arr, lo, hi):
    pivot = arr[hi]     ← choose last element as pivot
    i = lo - 1          ← boundary of left zone

    for j = lo to hi - 1:
        if arr[j] <= pivot:
            i++
            swap(arr[i], arr[j])    ← grow the left zone

    swap(arr[i+1], arr[hi])         ← place pivot between zones
    return i + 1                    ← pivot's final index
```

---

## Python Implementation

```python
def quick_sort(arr, lo=0, hi=None):
    if hi is None:
        hi = len(arr) - 1

    if lo < hi:
        p = partition(arr, lo, hi)
        quick_sort(arr, lo, p - 1)
        quick_sort(arr, p + 1, hi)

    return arr


def partition(arr, lo, hi):
    pivot = arr[hi]
    i = lo - 1                          # boundary: everything ≤ i is in the left zone

    for j in range(lo, hi):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]

    arr[i + 1], arr[hi] = arr[hi], arr[i + 1]   # place pivot
    return i + 1


# Example
arr = [10, 80, 30, 90, 40, 50, 70]
print("Before:", arr)
print("After: ", quick_sort(arr))
# After:  [10, 30, 40, 50, 70, 80, 90]
```

---

## Lomuto vs Hoare — Two Partition Schemes

There are two classic partition algorithms. Understanding both is an interview staple.

```
┌─────────────────┬──────────────────────────────┬──────────────────────────────┐
│                 │ Lomuto Partition              │ Hoare Partition              │
├─────────────────┼──────────────────────────────┼──────────────────────────────┤
│ Pivot position  │ Last element                 │ First element (or middle)    │
│ Pointers        │ One (j scans forward)        │ Two (i from left, j from right)│
│ Swaps           │ More swaps on average        │ ~3× fewer swaps              │
│ Returned index  │ Final pivot position         │ Partition boundary (not pivot)│
│ Stability       │ Unstable                     │ Unstable                     │
│ Simplicity      │ Simpler to read/implement    │ Harder but faster            │
│ Used for        │ Teaching, interviews          │ Production implementations   │
└─────────────────┴──────────────────────────────┴──────────────────────────────┘
```

Most interview questions use Lomuto because it's cleaner to trace. Real implementations (C++ STL, pdqsort) use Hoare-style or hybrid variants.

---

## Pivot Selection Strategies

The pivot choice is everything in Quick Sort. A bad pivot leads to O(n²).

```
Strategy          How                         Best for             Risk
─────────────────────────────────────────────────────────────────────────────
Last element      pivot = arr[hi]             Teaching, simple     O(n²) on sorted input
First element     pivot = arr[lo]             Simple               O(n²) on sorted input
Random element    pivot = arr[random(lo,hi)]  General use          Low — hard to exploit
Median of three   median(first, mid, last)    Production           Very low worst case
Ninther           median of medians (3×3)     Large datasets       Near-impossible to hit O(n²)
```

> **Why random pivot matters:** If your pivot is always the smallest or largest element (e.g. always picking `arr[hi]` on an already-sorted array), the partition produces one sub-array of size n-1 and one of size 0 — every time. That's O(n²). A random pivot makes this astronomically unlikely.

---

## Complexity

### Time

| Case         | Complexity      | When it happens                                            |
|--------------|-----------------|------------------------------------------------------------|
| Best case    | **O(n log n)**  | Pivot always lands at the midpoint — balanced splits       |
| Average case | **O(n log n)**  | Random input, random pivot — expected balanced splits      |
| Worst case   | **O(n²)**       | Pivot always smallest or largest — maximally unbalanced    |

### Space

| Metric          | Complexity    | Why                                                   |
|-----------------|---------------|-------------------------------------------------------|
| Auxiliary space | **O(1)**      | Partition is in-place — no extra arrays               |
| Call stack      | **O(log n)**  | Average recursion depth (O(n) in worst case)          |

---

## Properties at a Glance

| Property         | Value   | Notes                                                       |
|------------------|---------|-------------------------------------------------------------|
| Stable           | ❌ No   | Partition swaps can move equal elements past each other     |
| In-Place         | ✅ Yes  | No auxiliary array — only call stack space                  |
| Adaptive         | ❌ No   | Doesn't benefit from partially sorted input                 |
| Divide & Conquer | ✅ Yes  | Split (partition), recurse, no merge                        |
| Cache-friendly   | ✅ Yes  | Accesses memory sequentially during partition scan          |

> **Why Quick Sort beats Merge Sort in practice:** Despite the same average complexity, Quick Sort's in-place partition is far more cache-friendly than Merge Sort's auxiliary array writes. The constant factor is much smaller. On random in-memory data, Quick Sort is typically 2–3× faster than Merge Sort in wall-clock time.

---

## The Full Sorting Algorithm Comparison

| Feature              | Bubble    | Selection | Insertion     | Merge Sort      | **Quick Sort**         |
|----------------------|-----------|-----------|---------------|-----------------|------------------------|
| Best case            | O(n)      | O(n²)     | O(n)          | O(n log n)      | O(n log n)             |
| Average case         | O(n²)     | O(n²)     | O(n²)         | O(n log n)      | **O(n log n)**         |
| Worst case           | O(n²)     | O(n²)     | O(n²)         | O(n log n)      | ⚠️ O(n²)              |
| Space                | O(1)      | O(1)      | O(1)          | O(n)            | O(log n) stack         |
| Stable               | ✅        | ❌        | ✅            | ✅              | ❌                     |
| In-place             | ✅        | ✅        | ✅            | ❌              | ✅                     |
| Cache-friendly       | ✗         | ✗         | ✓             | ✗               | ✅✅                   |
| Real-world speed     | Slow      | Slow      | Fast (small n)| Fast            | **Fastest (large n)**  |

---

## Pros and Cons

### ✅ Advantages
- **Fastest in practice** on large random datasets — best cache behaviour of any O(n log n) sort
- **In-place** — no O(n) auxiliary memory like Merge Sort
- Average case O(n log n) with very small constant factor
- Tail-call optimisable — the smaller sub-array recurses first (limits stack depth)
- The algorithm behind C++ `std::sort`, Python's Tim Sort (partially), and most production sorters

### ❌ Disadvantages
- **O(n²) worst case** — sorted or reverse-sorted input with a naive pivot choice
- **Not stable** — equal elements can be reordered
- Worst-case call stack depth O(n) can cause stack overflow on large sorted inputs
- Harder to reason about correctness than Merge Sort

---

## When to Use It

```
Quick Sort is the right choice when:
  ✓ You need the fastest average-case in-memory sort
  ✓ Memory is tight — no O(n) auxiliary space available
  ✓ You use a random or median-of-three pivot (avoiding worst case)
  ✓ Data is random and large (cache-friendliness dominates)

Quick Sort is the wrong choice when:
  ✗ You need guaranteed O(n log n) — Merge Sort or Heap Sort instead
  ✗ You need a stable sort — Merge Sort or Tim Sort instead
  ✗ Data is nearly sorted and you can't randomise — Insertion Sort instead
  ✗ Memory access patterns are irregular (e.g. linked lists) — Merge Sort instead
```

---

## Common Interview Questions

**Q: What is the worst case for Quick Sort and when does it occur?**
O(n²), when the pivot is always the smallest or largest element of the range — creating partitions of size 0 and n-1 at every step. This happens on already-sorted (or reverse-sorted) input with a naive last-element pivot. Random pivot or median-of-three makes this practically impossible.

**Q: Is Quick Sort stable?**
No. The partition step can swap equal elements past each other, changing their original relative order. Use Merge Sort or Tim Sort when stability matters.

**Q: How does Quick Sort differ from Merge Sort?**
Merge Sort splits blindly and does work during the merge (bottom-up). Quick Sort does work during the partition (top-down) and has nothing to merge. Quick Sort is in-place and cache-friendly; Merge Sort needs O(n) extra space but guarantees O(n log n) worst case.

**Q: What is the space complexity of Quick Sort?**
O(1) auxiliary (in-place partition) plus O(log n) call stack on average. In the worst case the stack grows to O(n). To guard against this, always recurse on the *smaller* sub-array first (this guarantees O(log n) max stack depth).

**Q: What sorting algorithm does Python use?**
Tim Sort — a hybrid of Merge Sort (for large runs) and Insertion Sort (for small sub-arrays, ≤32 elements). It's stable and guarantees O(n log n). For raw performance on C-level numeric arrays (numpy), Quick Sort variants are used.

**Q: What is Introsort?**
Introsort is the algorithm behind C++'s `std::sort`. It starts as Quick Sort, switches to Heap Sort if recursion depth exceeds 2·log(n) (avoiding O(n²) worst case), and switches to Insertion Sort for small sub-arrays. Best of all three worlds.

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  QUICK SORT — AT A GLANCE                                         │
│                                                                   │
│  Strategy:   Choose pivot → partition in-place → recurse both     │
│  Key step:   Partition (Lomuto or Hoare scheme)                   │
│                                                                   │
│  Time  →  Average: O(n log n)  |  Worst: O(n²)                    │
│  Space →  O(1) auxiliary  +  O(log n) call stack                  │
│                                                                   │
│  Stable: No   In-Place: Yes   Cache-Friendly: Yes                 │
│                                                                   │
│  Use for: large random in-memory data, tight memory budgets       │
│  Avoid:   sorted input w/ naive pivot, stability requirements     │
│  Evolved into: Introsort (C++ STL), pdqsort (Rust std)            │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 5 of daily algorithm commits — next up: Heap Sort*