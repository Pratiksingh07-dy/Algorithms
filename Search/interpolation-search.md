# 📐 Interpolation Search

> *"Binary Search always checks the middle. Interpolation Search asks — where is the target most likely to be?"*

---

## What is Interpolation Search?

Interpolation Search is an improvement over Binary Search for **uniformly distributed sorted arrays**. Instead of always jumping to the midpoint, it estimates the probable position of the target using a formula derived from linear interpolation — the same idea as reading a dictionary and opening it closer to 'Z' when looking for "zebra", not right in the middle.

If the values are spread evenly, this estimation lands very close to the target immediately — giving **O(log log n)** average time instead of Binary Search's O(log n).

Two conditions required:
- Array must be **sorted**
- Values should be **uniformly distributed** (the algorithm degrades to O(n) on skewed data)

---

## The Core Formula

Binary Search always picks:
```
mid = (lo + hi) // 2
```

Interpolation Search estimates the position proportionally:
```
pos = lo + ((target - arr[lo]) / (arr[hi] - arr[lo])) × (hi - lo)
```

**Reading the formula:**
- `(target - arr[lo]) / (arr[hi] - arr[lo])` — what fraction of the value range is the target?
- Multiply that fraction by `(hi - lo)` — apply the same fraction to the index range
- Add `lo` — offset from the start of the range

If values are perfectly uniform, this lands exactly on the target on the first try.

---

## The Problem

**Input:** A sorted, uniformly distributed array and a target

```
arr    = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
target = 70
n      = 10
```

**Output:** Index of the target, or -1 if not found

```
6   (arr[6] = 70)
```

---

## Visualising Interpolation Search

```
arr    = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
idx    =   0   1   2   3   4   5   6   7   8    9
target = 70

── Step 1 ────────────────────────────────────────────────────────
lo=0, hi=9
arr[lo]=10, arr[hi]=100

pos = 0 + ((70 - 10) / (100 - 10)) × (9 - 0)
    = 0 + (60 / 90) × 9
    = 0 + 0.666 × 9
    = 0 + 6
    = 6

arr[6] = 70 == 70 → FOUND at index 6 ✅

Total comparisons: 1  (Binary Search would take 3)
```

---

## Trace on a harder example

```
arr    = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
target = 85

── Step 1 ────────────────────────────────────────────────────────
lo=0, hi=9, arr[lo]=10, arr[hi]=100

pos = 0 + ((85 - 10) / (100 - 10)) × (9 - 0)
    = 0 + (75 / 90) × 9
    = 0 + 0.833 × 9
    = 7 (floor)

arr[7] = 80 → 80 < 85  →  lo = 8

── Step 2 ────────────────────────────────────────────────────────
lo=8, hi=9, arr[lo]=90, arr[hi]=100

pos = 8 + ((85 - 90) / (100 - 90)) × (9 - 8)
    = 8 + (-5 / 10) × 1
    = 8 + (-0.5)
    = 7 (floor) → clamp to lo=8

arr[8] = 90 → 90 > 85  →  hi = 7

── lo > hi: search space exhausted → return -1

Target 85 not in array.
```

> **Notice:** When `target < arr[pos]` or `target > arr[pos]`, the search space narrows — similar to Binary Search but asymmetrically based on the estimated position.

---

## The Algorithm

```
1. While lo ≤ hi AND target is within range [arr[lo], arr[hi]]:
     a. Estimate position using the interpolation formula
     b. If arr[pos] == target → return pos
     c. If arr[pos] < target  → lo = pos + 1  (search right)
     d. If arr[pos] > target  → hi = pos - 1  (search left)
2. Return -1 if not found
```

---

## Pseudocode

```
function interpolation_search(arr, target):
    lo = 0
    hi = len(arr) - 1

    while lo <= hi AND arr[lo] <= target <= arr[hi]:

        if lo == hi:
            if arr[lo] == target: return lo
            return -1

        # Interpolation formula
        pos = lo + int((target - arr[lo]) / (arr[hi] - arr[lo]) * (hi - lo))

        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1

    return -1
```

---

## Python Implementation

```python
def interpolation_search(arr, target):
    lo, hi = 0, len(arr) - 1

    while lo <= hi and arr[lo] <= target <= arr[hi]:

        if lo == hi:
            return lo if arr[lo] == target else -1

        # Estimate position by linear interpolation
        pos = lo + int(
            (target - arr[lo]) / (arr[hi] - arr[lo]) * (hi - lo)
        )

        if arr[pos] == target:
            return pos
        elif arr[pos] < target:
            lo = pos + 1
        else:
            hi = pos - 1

    return -1


# Example
arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
print(interpolation_search(arr, 70))   # → 6
print(interpolation_search(arr, 25))   # → -1
```

> **Guard clause:** `arr[lo] <= target <= arr[hi]` catches out-of-range targets immediately and avoids division-by-zero when `arr[lo] == arr[hi]`.

---

## Why O(log log n) for Uniform Data?

```
Binary Search halves the search space each step:
  n → n/2 → n/4 → ... → 1    takes log₂(n) steps

Interpolation Search on uniform data:
  The formula lands close to the target, leaving a search space
  proportional to √(range) each time.
  n → √n → √(√n) → ... → 1   takes log(log(n)) steps

For n = 1,000,000,000 (1 billion):
  Binary Search       → log₂(1B)       ≈ 30 comparisons
  Interpolation Search→ log(log(1B))   ≈  5 comparisons
```

The catch: this only holds when values are uniformly distributed. On skewed data (e.g. exponentially growing values), the formula overshoots repeatedly and degrades to O(n).

---

## Complexity

### Time

| Case         | Complexity       | When it happens                            |
|--------------|------------------|--------------------------------------------|
| Best case    | **O(1)**         | Formula lands exactly on the target        |
| Average case | **O(log log n)** | Values are uniformly distributed           |
| Worst case   | **O(n)**         | Values are heavily skewed (exponential etc)|

### Space

| Metric          | Complexity | Why                        |
|-----------------|-----------|------------------------------|
| Auxiliary space | **O(1)**  | Only lo, hi, pos variables   |

---

## Properties at a Glance

| Property              | Value    | Notes                                               |
|-----------------------|----------|-----------------------------------------------------|
| Requires sorted array | ✅ Yes   | Must be sorted for the formula to make sense        |
| Requires uniform data | ⚠️ Ideal | Works on skewed data but degrades to O(n)          |
| In-Place              | ✅ Yes   | No extra memory                                     |
| Stable                | ✅ Yes   | Returns one index; no reordering                    |

---

## The Four Search Algorithms Compared

| Feature               | Linear   | Jump     | Binary     | **Interpolation**  |
|-----------------------|----------|----------|------------|--------------------|
| Requires sorted       | ❌       | ✅       | ✅         | ✅                 |
| Requires uniform data | ❌       | ❌       | ❌         | ⚠️ For best case  |
| Time (best)           | O(1)     | O(1)     | O(1)       | **O(1)**           |
| Time (average)        | O(n)     | O(√n)    | O(log n)   | **O(log log n)**   |
| Time (worst)          | O(n)     | O(√n)    | O(log n)   | O(n)               |
| Space                 | O(1)     | O(1)     | O(1)       | O(1)               |
| Forward-only          | ✅       | ✅       | ❌         | ❌                 |

---

## When the Formula Breaks Down

```
Uniform distribution (Interpolation wins):
  arr = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
  Values equally spaced → formula estimates perfectly

Exponential distribution (Interpolation degrades):
  arr = [1, 2, 4, 8, 16, 32, 64, 128, 256, 512]
  Searching for 200:
  pos = 0 + ((200-1)/(512-1)) × 9 = 0 + 0.39 × 9 = 3
  arr[3] = 8 → way off. Search space barely shrinks → O(n)

All same values except one (worst case):
  arr = [1, 1, 1, 1, 1, 1, 1, 1, 1, 2]  target = 2
  Every step: pos = lo + (2-1)/(2-1) × (hi-lo) = hi
  Scans the entire array → O(n)
```

---

## Real-World Uses

```
1. Phone directories / sorted name lists   — names roughly uniformly distributed
2. Database index range scans              — numeric keys often uniformly spaced
3. Timestamp-indexed logs                  — log entries arrive at a roughly constant rate
4. Sensor data retrieval                   — readings sampled at regular intervals
5. In-memory dictionary lookup             — if keys are dense integers
6. Scientific datasets                     — physical measurements often uniform
```

---

## Pros and Cons

### ✅ Advantages
- O(log log n) average case on uniform data — faster than Binary Search
- O(1) extra space
- Falls back gracefully — still correct on non-uniform data, just slower
- Simple formula, easy to implement

### ❌ Disadvantages
- O(n) worst case — worse than Binary Search's guaranteed O(log n)
- Requires uniform distribution to realise the speed advantage
- Division by `(arr[hi] - arr[lo])` requires care — must handle equal values
- Not well-known — in practice Binary Search is almost always used instead

---

## When to Use It

```
Interpolation Search is the right choice when:
  ✓ The array is sorted and values are uniformly distributed
  ✓ The array is very large and uniform (phone book, sensor logs)
  ✓ O(log log n) matters — extreme performance requirements

Interpolation Search is the wrong choice when:
  ✗ Data is skewed or clustered → worst case O(n), use Binary Search
  ✗ You don't know the distribution → Binary Search is safer
  ✗ Array is small → the formula overhead isn't worth it
```

---

## Common Interview Questions

**Q: What is the average time complexity of Interpolation Search and when does it apply?**
O(log log n) — when the array is sorted and values are uniformly distributed. Each step the search space shrinks by approximately a square root factor instead of half.

**Q: When does Interpolation Search degrade to O(n)?**
When values are heavily skewed — for example exponentially distributed, or when many elements share the same value. The formula repeatedly lands far from the target, barely shrinking the search space.

**Q: How is Interpolation Search different from Binary Search?**
Binary Search always picks the midpoint (`(lo+hi)//2`). Interpolation Search estimates the position proportionally based on the target's value relative to the value range — like how you'd open a dictionary closer to the end for a word starting with 'Z'.

**Q: What happens if all elements are the same?**
Division by zero in `arr[hi] - arr[lo]`. Handle with a guard: if `arr[lo] == arr[hi]`, check only `arr[lo]` and return accordingly.

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  INTERPOLATION SEARCH — AT A GLANCE                               │
│                                                                   │
│  Strategy:  Estimate position by value proportion, not midpoint  │
│  Formula:   pos = lo + (target-arr[lo])/(arr[hi]-arr[lo])*(hi-lo)│
│                                                                   │
│  Time  →  O(log log n) avg (uniform)  |  O(n) worst (skewed)    │
│  Space →  O(1)                                                   │
│                                                                   │
│  Best for: large, sorted, uniformly distributed arrays            │
│  Avoid for: skewed data, unknown distribution → use Binary Search │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 10 of daily algorithm commits — next up: Linked List (Data Structure)*
