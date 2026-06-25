# 🦘 Jump Search

> *"Too slow for Linear, too limited for Binary — Jump Search lives in the sweet spot between them."*

---

## What is Jump Search?

Jump Search is a search algorithm for **sorted arrays** that sits between Linear Search and Binary Search in both complexity and concept.

Instead of checking every element (Linear) or jumping to the middle (Binary), Jump Search moves through the array in **fixed-size blocks** — jumping ahead by √n steps at a time. Once it overshoots the target, it steps backward one block and does a short Linear Search within that block.

Two phases:
- **Jump phase** — leap forward by `step = √n` until `arr[step] ≥ target`
- **Linear phase** — scan backward within the identified block to find the target

The result: **O(√n)** — better than O(n), worse than O(log n), but with one key advantage: it only moves **forward**, never backward during the jump phase. This makes it ideal for storage media where backward seeking is expensive (magnetic tapes, spinning disks).

---

## The Problem

**Input:** A sorted array and a target value

```
arr    = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
target = 55
n      = 15
step   = √15 ≈ 3
```

**Output:** Index of the target, or -1 if not found

```
10   (arr[10] = 55)
```

---

## Visualising Jump Search

```
arr  = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
idx  =  0  1  2  3  4  5  6   7   8   9  10  11   12   13   14
target = 55,  step = √15 ≈ 3

── Jump Phase ────────────────────────────────────────────────────
Jump 1: check index 0 + 3 = 3  →  arr[3]  =  2  →  2  < 55 → jump
        [ 0   1   1  [2]  3   5   8  13  21  34  55  89 ... ]
                      ^
Jump 2: check index 3 + 3 = 6  →  arr[6]  =  8  →  8  < 55 → jump
        [ 0   1   1   2   3   5  [8] 13  21  34  55  89 ... ]
                                  ^
Jump 3: check index 6 + 3 = 9  →  arr[9]  = 34  →  34 < 55 → jump
        [ 0   1   1   2   3   5   8  13  21 [34] 55  89 ... ]
                                              ^
Jump 4: check index 9 + 3 = 12 →  arr[12] = 144 →  144 > 55 → STOP

── Linear Phase ──────────────────────────────────────────────────
Overshoot! Step back and scan [prev..current] = [9..12)
        [ ...  34  55  89 144 ... ]
                ^   ^   ^   ^
              idx=9  10  11  12

  arr[9]  = 34  →  34 ≠ 55 → continue
  arr[10] = 55  →  55 = 55 → FOUND at index 10 ✅

Total jumps: 4   (vs max 15 linear checks)
Linear phase: 2 comparisons
```

---

## Why √n is the Optimal Step Size

Let `m` = the step size (block size).

```
Jump phase cost:   n/m  jumps  (to scan the whole array)
Linear phase cost: m    comparisons (one block scan at most)

Total worst case:  n/m + m

Minimise by taking the derivative with respect to m:
  d/dm (n/m + m) = -n/m² + 1 = 0
  m² = n
  m  = √n

At m = √n:
  Total comparisons = n/√n + √n = √n + √n = 2√n = O(√n) ✅
```

This is a beautiful result: **the optimal block size is exactly the square root of the array size**.

---

## Trace Table

| Step | Phase  | Index  | arr[idx] | Decision              |
|------|--------|--------|----------|-----------------------|
| 1    | Jump   | 3      | 2        | 2 < 55 → jump again   |
| 2    | Jump   | 6      | 8        | 8 < 55 → jump again   |
| 3    | Jump   | 9      | 34       | 34 < 55 → jump again  |
| 4    | Jump   | 12     | 144      | 144 > 55 → stop, scan |
| 5    | Linear | 9      | 34       | 34 ≠ 55 → next        |
| 6    | Linear | 10     | 55       | **Found!** ✅         |

---

## The Algorithm

```
1.  Compute step = √n
2.  Set prev = 0, curr = step
3.  Jump phase: while arr[min(curr, n)-1] < target:
        prev = curr
        curr += step
        if prev >= n: return -1  (overshot the array)
4.  Linear phase: scan from arr[prev] to arr[min(curr, n)]
5.  If match found → return index
6.  Return -1
```

---

## Pseudocode

```
function jump_search(arr, target):
    n    = len(arr)
    step = floor(√n)
    prev = 0

    # Jump phase: find block where target may exist
    while arr[min(step, n) - 1] < target:
        prev = step
        step += floor(√n)
        if prev >= n:
            return -1       ← overshot the entire array

    # Linear phase: scan within the identified block
    while arr[prev] < target:
        prev += 1
        if prev == min(step, n):
            return -1       ← target not in this block

    if arr[prev] == target:
        return prev

    return -1
```

---

## Python Implementation

```python
import math

def jump_search(arr, target):
    n    = len(arr)
    step = int(math.sqrt(n))
    prev = 0

    # Jump phase
    while arr[min(step, n) - 1] < target:
        prev  = step
        step += int(math.sqrt(n))
        if prev >= n:
            return -1

    # Linear phase: scan backwards from the overshoot point
    while arr[prev] < target:
        prev += 1
        if prev == min(step, n):
            return -1

    if arr[prev] == target:
        return prev

    return -1


# Example
arr = [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377]
print(jump_search(arr, 55))   # → 10
print(jump_search(arr, 10))   # → -1
```

---

## Complexity

### Time

| Case         | Complexity   | When it happens                              |
|--------------|-------------|----------------------------------------------|
| Best case    | **O(1)**    | Target is the first element checked          |
| Average case | **O(√n)**   | Target found after several jumps + short scan|
| Worst case   | **O(√n)**   | Target at end or not present                 |

### Space

| Metric          | Complexity | Why                         |
|-----------------|-----------|------------------------------|
| Auxiliary space | **O(1)**  | Only a handful of variables  |

---

## Properties at a Glance

| Property              | Value   | Notes                                              |
|-----------------------|---------|----------------------------------------------------|
| Requires sorted array | ✅ Yes  | Will give wrong results on unsorted input          |
| In-Place              | ✅ Yes  | No extra data structure needed                     |
| Jump direction        | ➡️ Forward only | Never steps backward during jump phase   |
| Works on linked lists | ⚠️ Limited | Can jump by step if forward traversal is possible |

---

## Jump Search vs Linear vs Binary

| Feature               | Linear Search | Jump Search | Binary Search |
|-----------------------|---------------|-------------|---------------|
| Requires sorted array | ❌ No         | ✅ Yes      | ✅ Yes        |
| Time complexity       | O(n)          | **O(√n)**   | O(log n)      |
| Space complexity      | O(1)          | O(1)        | O(1)          |
| Direction of traversal| Forward       | Forward only| Both ways     |
| Works on linked lists | ✅ Yes        | ⚠️ Partial  | ❌ No         |
| Practical use case    | Unsorted data | Sequential media | Large sorted arrays |

```
Comparison for n = 10,000:
  Linear Search → up to 10,000 comparisons
  Jump Search   → up to √10,000 = 100 comparisons  ← 100× faster than linear
  Binary Search → up to log₂(10,000) ≈ 14 comparisons
```

---

## The Key Advantage: Forward-Only Traversal

Binary Search jumps forward and backward — if your storage device has high backward-seek cost (like a magnetic tape drive), Binary Search is expensive. Jump Search only ever moves forward in the jump phase, making it the algorithm of choice for:

```
Magnetic tapes     — backward rewind is physically slow (mechanical)
Rotating disk HDDs — reverse seeks involve disk rotation + head movement
Sequential streams — data arriving in order, no random access
Append-only logs   — forward scanning is all that's possible
```

For in-memory arrays, this advantage disappears and Binary Search is almost always faster. But Jump Search exists precisely for these edge cases.

---

## Choosing the Right Search Algorithm

```
Unsorted data?
  → Linear Search  O(n)

Sorted array, small n (< 20)?
  → Linear Search  O(n)   ← cache advantage, no overhead

Sorted array, large n, RAM-based?
  → Binary Search  O(log n)  ← fastest

Sorted array, forward-only storage (tape, disk)?
  → Jump Search  O(√n)  ← better than linear, no backward seeks

Sorted array, need all duplicates?
  → Binary Search (first/last occurrence variants)

Hash-based data (dict/set)?
  → Hash lookup  O(1)  ← beats all of the above
```

---

## Real-World Uses

```
1. Magnetic tape search       — classic use case; O(√n) with forward-only seeks
2. Sequential log file search — find first entry past a timestamp
3. Filesystem block search    — when backward block seeking is expensive
4. Teaching O(√n) complexity  — a clean example of an intermediate algorithm class
5. Jump pointers in skip lists — skip lists generalize this idea to multiple levels
```

---

## Pros and Cons

### ✅ Advantages
- Faster than Linear Search — O(√n) vs O(n)
- Forward-only movement — ideal for sequential storage media
- Simple to implement
- O(1) extra space
- Good teaching example of trading between O(n) and O(log n)

### ❌ Disadvantages
- Slower than Binary Search — O(√n) vs O(log n)
- Requires a sorted array
- The step size must be tuned (√n is optimal but fixed)
- For in-memory arrays, Binary Search is almost always the better choice
- Not adaptive — performance doesn't improve on nearly-sorted or nearly-found data

---

## Common Interview Questions

**Q: Why is √n the optimal block size for Jump Search?**
Total comparisons = n/m (jumps) + m (linear scan). Minimising n/m + m by differentiation gives m = √n. At that size both terms equal √n, giving 2√n total — the minimum possible.

**Q: When would you prefer Jump Search over Binary Search?**
When the data is on a storage medium where backward seeking is expensive — magnetic tapes, rotating disk HDDs, or any append-only sequential stream. Jump Search only ever moves forward in its main phase.

**Q: What is the worst case for Jump Search?**
The target is either the last element or not present. In that case the jump phase takes n/√n = √n jumps, and the linear phase scans a full block of √n elements: 2√n comparisons total = O(√n).

**Q: Can Jump Search find duplicate values?**
Standard Jump Search returns one occurrence. To find all duplicates, after finding the target, scan linearly in both directions until the value changes — similar to how you'd extend a Binary Search result.

**Q: How does Jump Search relate to Skip Lists?**
Skip Lists generalize the Jump Search idea across multiple levels. Level 1 jumps by √n, level 2 jumps by n^(2/3), and so on — giving O(log n) expected time with forward-only traversal. Jump Search is essentially a one-level skip list.

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  JUMP SEARCH — AT A GLANCE                                        │
│                                                                   │
│  Strategy:  Jump by √n blocks, then linear scan within block     │
│  Prerequisite: array must be sorted                               │
│                                                                   │
│  Time  →  O(√n) — between O(n) and O(log n)                     │
│  Space →  O(1)                                                   │
│                                                                   │
│  Optimal block size: √n  (minimises total comparisons)            │
│  Key advantage: forward-only traversal (great for tape/disk)      │
│                                                                   │
│  Use for: sorted data on sequential/forward-only storage          │
│  Avoid for: in-memory arrays → Binary Search is faster            │
│  Related to: Skip Lists (multi-level Jump Search)                 │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 9 of daily algorithm commits — next up: Interpolation Search*
