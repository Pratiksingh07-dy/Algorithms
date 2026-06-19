# ⚡ Merge Sort

> *"Divide until trivial, then conquer by merging. The first O(n log n) algorithm most programmers ever meet."*

---

## What is Merge Sort?

Merge Sort is a **divide and conquer** algorithm. It breaks the problem into smaller sub-problems, solves each one, and combines the results.

The core insight: **a single-element array is always sorted**. So if you keep splitting until every piece has one element, all you have to do is merge sorted pieces back together — and merging two sorted arrays is easy and fast.

Two phases:
- **Divide** — recursively split the array in half until every sub-array has one element
- **Conquer (Merge)** — repeatedly merge adjacent sorted sub-arrays into larger sorted arrays

Unlike Bubble, Selection, and Insertion Sort, Merge Sort is **not in-place** and **guarantees O(n log n)** in all cases — best, average, and worst.

---

## The Problem

**Input:** An unsorted array

```
[38, 27, 43, 3, 9, 82, 10]
```

**Output:** Elements in ascending order

```
[3, 9, 10, 27, 38, 43, 82]
```

---

## Visualising the Algorithm

### Phase 1 — Split (top-down)

```
[38, 27, 43, 3, 9, 82, 10]
          /              \
  [38, 27, 43]       [3, 9, 82, 10]
     /     \            /        \
 [38, 27]  [43]     [3, 9]    [82, 10]
   /    \              /   \    /    \
 [38]  [27]          [3]  [9] [82] [10]
```

Every leaf is a single element — trivially sorted.

---

### Phase 2 — Merge (bottom-up)

Now merge adjacent sorted pairs at each level:

```
── Level 1: merge pairs of single elements ──────────────────────
  [38] + [27]  →  compare 38 vs 27  →  [27, 38]
  [3]  + [9]   →  compare 3  vs 9   →  [3, 9]
  [82] + [10]  →  compare 82 vs 10  →  [10, 82]
  [43] stays as [43] (odd one out, joins next level)

── Level 2: merge pairs of 2-element arrays ─────────────────────
  [27, 38] + [43]  →
    compare 27 vs 43 → take 27
    compare 38 vs 43 → take 38
    remainder  [43]  → take 43
    result: [27, 38, 43]

  [3, 9] + [10, 82]  →
    compare 3  vs 10 → take 3
    compare 9  vs 10 → take 9
    compare 10 vs -- → take 10
    remainder  [82]  → take 82
    result: [3, 9, 10, 82]

── Level 3: final merge ─────────────────────────────────────────
  [27, 38, 43] + [3, 9, 10, 82]  →
    compare 27 vs 3  → take 3
    compare 27 vs 9  → take 9
    compare 27 vs 10 → take 10
    compare 27 vs 27 → take 27
    compare 38 vs 38 → take 38
    compare 43 vs 43 → take 43
    remainder   [82] → take 82
    result: [3, 9, 10, 27, 38, 43, 82]  ✅
```

> **Key insight:** Each merge step scans both halves exactly once and produces a sorted output. This is why the overall complexity is O(n log n) — log n levels of splitting, each level doing O(n) total work across all merges at that level.

---

## The Algorithm

```
Divide:
  1. Find the midpoint of the array
  2. Recursively sort the left half
  3. Recursively sort the right half

Conquer (Merge):
  4. Compare the front elements of both halves
  5. Take the smaller one and place it in the output
  6. Repeat until one half is exhausted
  7. Copy the remaining elements from the other half
```

---

## Pseudocode

```
function merge_sort(arr):
    if len(arr) <= 1:
        return arr               ← base case: already sorted

    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])    ← sort left half
    right = merge_sort(arr[mid:])    ← sort right half

    return merge(left, right)        ← merge sorted halves


function merge(left, right):
    result = []
    i, j = 0, 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i]); i++
        else:
            result.append(right[j]); j++

    result += left[i:]     ← copy remainder
    result += right[j:]
    return result
```

---

## Python Implementation

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr

    mid = len(arr) // 2
    left  = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])

    return merge(left, right)


def merge(left, right):
    result = []
    i = j = 0

    while i < len(left) and j < len(right):
        if left[i] <= right[j]:         # ← <= keeps it stable
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1

    result.extend(left[i:])             # copy remaining elements
    result.extend(right[j:])
    return result


# Example
arr = [38, 27, 43, 3, 9, 82, 10]
print("Before:", arr)
print("After: ", merge_sort(arr))
# After:  [3, 9, 10, 27, 38, 43, 82]
```

> **On stability:** Using `<=` (not `<`) in the comparison ensures that when two elements are equal, the left half's element is taken first — preserving their original relative order. This is what makes Merge Sort stable.

---

## Why O(n log n)?

```
Array size n = 8:

Split levels:
  Level 0:  1 array  of size 8   →  split into 2
  Level 1:  2 arrays of size 4   →  split into 4
  Level 2:  4 arrays of size 2   →  split into 8
  Level 3:  8 arrays of size 1   →  base case

  Total levels = log₂(8) = 3   →   log n levels

Merge work per level:
  Each level merges all n elements in total.
  Level 3→2: merge 4 pairs of size 1 → 4×2 = 8 comparisons = O(n)
  Level 2→1: merge 2 pairs of size 2 → 2×4 = 8 comparisons = O(n)
  Level 1→0: merge 1 pair  of size 4 → 1×8 = 8 comparisons = O(n)

  Total = log n levels × O(n) work per level = O(n log n) ✅
```

---

## Complexity

### Time

| Case         | Complexity      | Why                                               |
|--------------|-----------------|---------------------------------------------------|
| Best case    | **O(n log n)**  | Always splits log n times; always merges n total  |
| Average case | **O(n log n)**  | Same structure regardless of input order          |
| Worst case   | **O(n log n)**  | No pathological case — guaranteed performance     |

### Space

| Metric          | Complexity  | Why                                             |
|-----------------|-------------|--------------------------------------------------|
| Auxiliary space | **O(n)**    | Temporary arrays needed during each merge step  |
| Call stack      | **O(log n)**| Recursion depth equals the number of split levels |

> The O(n) space cost is Merge Sort's main trade-off. For datasets that don't fit in memory this matters — but for most use cases the predictable O(n log n) time is worth it.

---

## Properties at a Glance

| Property         | Value   | Notes                                                      |
|------------------|---------|------------------------------------------------------------|
| Stable           | ✅ Yes  | Equal elements maintain relative order (with `<=` compare) |
| In-Place         | ❌ No   | Requires O(n) extra memory for temporary arrays            |
| Adaptive         | ❌ No   | Always O(n log n) — doesn't exploit existing order         |
| Divide & Conquer | ✅ Yes  | Core strategy — split, recurse, merge                      |
| Parallelisable   | ✅ Yes  | Independent sub-problems can run on separate threads       |
| External sort    | ✅ Yes  | Works excellently for data too large to fit in memory      |

---

## The Sorting Algorithm Showdown

| Feature              | Bubble       | Selection    | Insertion         | **Merge Sort**      |
|----------------------|--------------|--------------|-------------------|---------------------|
| Best case            | O(n)         | O(n²)        | O(n)              | **O(n log n)**      |
| Average case         | O(n²)        | O(n²)        | O(n²)             | **O(n log n)**      |
| Worst case           | O(n²)        | O(n²)        | O(n²)             | **O(n log n)**      |
| Space                | O(1)         | O(1)         | O(1)              | O(n)                |
| Stable               | ✅           | ❌           | ✅                | ✅                  |
| Adaptive             | ✅           | ❌           | ✅                | ❌                  |
| Good for large data  | ❌           | ❌           | ❌                | ✅                  |

> Merge Sort is the first algorithm in this series that's actually used in production — Python's `list.sort()` (Tim Sort) and Java's `Arrays.sort()` for objects both use Merge Sort as their core strategy.

---

## Real-World Uses

**Merge Sort is used when:**

```
1. Linked lists — no random access needed; merge is naturally O(n)
2. External sorting — data too large to fit in RAM
   (split into chunks, sort each, merge across disk/network)
3. Stable sort required for non-primitive types
   (Java's Arrays.sort for objects, Python's sorted() both use merge-based sorts)
4. Counting inversions — counting how "unsorted" an array is
5. Parallel sorting — independent halves computed on separate CPU cores
```

---

## Pros and Cons

### ✅ Advantages
- Guaranteed **O(n log n)** — no worst-case surprises like Quick Sort
- **Stable** — safe for records where order matters
- Excellent for **linked lists** (O(1) merge, no random access needed)
- Best algorithm for **external sorting** (disk-based, streamed data)
- Naturally **parallelisable** — left and right halves are independent

### ❌ Disadvantages
- Requires **O(n) extra memory** — not in-place
- Slower than Quick Sort in practice on random in-memory data (constant factors)
- **Not adaptive** — doesn't speed up on nearly-sorted input
- Recursion overhead can be significant for very small arrays (this is why Tim Sort switches to Insertion Sort below ~32 elements)

---

## When to Use It

```
Merge Sort is the right choice when:
  ✓ You need a guaranteed O(n log n) — Quick Sort's worst case is O(n²)
  ✓ You need a stable sort for objects / records
  ✓ You're sorting a linked list (ideal fit)
  ✓ Data doesn't fit in memory (external sort)
  ✓ You want to parallelise the sort across multiple threads

Merge Sort is the wrong choice when:
  ✗ Memory is tight — O(n) extra space is too costly
  ✗ Data is small or nearly sorted → use Insertion Sort
  ✗ Raw speed on random in-memory data → Quick Sort is faster in practice
```

---

## Common Interview Questions

**Q: Why is Merge Sort's time complexity O(n log n)?**
There are log n levels of splitting (each halving the size), and at each level the merge step touches all n elements in total. So: log n levels × O(n) work per level = O(n log n).

**Q: Is Merge Sort stable?**
Yes — as long as the merge step takes the left element first when two elements are equal (using `<=`, not `<`). This preserves the original relative order.

**Q: How is Merge Sort different from Quick Sort?**
Merge Sort always splits in half and does the work during the merge (bottom-up). Quick Sort does the work during the split (choosing a pivot) and the sub-arrays are already in place. Merge Sort guarantees O(n log n); Quick Sort averages O(n log n) but degrades to O(n²) in the worst case. Merge Sort needs O(n) extra space; Quick Sort is in-place.

**Q: Why does Python use Merge Sort for `sorted()`?**
Python uses Tim Sort, which is a hybrid of Merge Sort and Insertion Sort. It uses Insertion Sort for small runs (≤ 32 elements) and Merge Sort for combining runs. The stable guarantee is essential because Python's sort is used on objects and tuples where stability matters.

**Q: Can Merge Sort be done in-place?**
Technically yes, but in-place merge algorithms are complex and have worse constant factors. In practice, the standard O(n) auxiliary space version is always preferred.

---

## Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│  MERGE SORT — AT A GLANCE                                       │
│                                                                 │
│  Strategy:   Divide in half → recurse → merge sorted halves     │
│  Levels:     log n (split depth)                                │
│                                                                 │
│  Time  →  Always O(n log n) — best, average, and worst          │
│  Space →  O(n) auxiliary + O(log n) call stack                  │
│                                                                 │
│  Stable: Yes   In-Place: No   Adaptive: No                      │
│                                                                 │
│  Use for: large data, linked lists, external sort, stable sort  │
│  Used by: Python's Tim Sort, Java Arrays.sort (objects)         │
└─────────────────────────────────────────────────────────────────┘
```

---

*Day 4 of daily algorithm commits — next up: Quick Sort*