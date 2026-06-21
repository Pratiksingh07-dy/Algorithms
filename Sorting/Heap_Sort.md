# 🏔️ Heap Sort

> *"The algorithm that turns an array into a priority queue — then empties it in sorted order."*

---

## What is Heap Sort?

Heap Sort is a **comparison-based, in-place** sorting algorithm built on top of a data structure called a **binary heap**. It runs in guaranteed O(n log n) time in all cases — like Merge Sort — but uses O(1) extra space — like Quick Sort.

The algorithm has two distinct phases:

- **Phase 1 — Build:** Rearrange the array into a valid **max-heap**, where every parent is larger than its children
- **Phase 2 — Extract:** Repeatedly pull the root (the current maximum) to the end of the array, shrink the heap, and restore the heap property

The result: the array fills up from the right with sorted elements.

---

## Understanding the Max-Heap

A **binary heap** is a complete binary tree stored *inside the array itself* — no pointers, no extra memory.

For any node at index `i`:

```
Parent:       (i - 1) // 2
Left child:    2*i + 1
Right child:   2*i + 2
```

**Max-heap property:** Every parent's value ≥ both its children's values.
The largest element is always at index 0 (the root).

**Example — array `[10, 5, 8, 3, 4, 7, 2]` as a tree:**

```
         10          ← index 0 (root)
        /    \
       5       8     ← indices 1, 2
      / \     / \
     3   4   7   2   ← indices 3, 4, 5, 6
```

Array: `[ 10,  5,  8,  3,  4,  7,  2 ]`
Index:    0    1   2   3   4   5   6

---

## The Heapify Operation

`heapify(arr, i, n)` — the core subroutine. Given that node `i`'s children are already valid heaps, it "sifts down" node `i` until the max-heap property is restored.

```
heapify([4, 10, 8, 3, 5, 7, 2], i=0, n=7)

Step 1: Node 0 has value 4. Children: left=10 (idx 1), right=8 (idx 2).
        Largest = 10 (idx 1). 4 < 10 → swap.
        [ 10,  4,  8,  3,  5,  7,  2 ]

Step 2: Now at idx 1, value 4. Children: left=3 (idx 3), right=5 (idx 4).
        Largest = 5 (idx 4). 4 < 5 → swap.
        [ 10,  5,  8,  3,  4,  7,  2 ]

Step 3: Now at idx 4, value 4. No children within heap. Stop.
        Max-heap property restored. ✅
```

Each heapify call touches at most one path from a node to a leaf: **O(log n)** time.

---

## Full Trace

**Input:** `[4, 10, 3, 5, 1, 7, 8]`

### Phase 1 — Build the max-heap (bottom-up)

Start from the last non-leaf node: index `n//2 - 1 = 2`.

```
Initial:   [ 4,  10,  3,  5,  1,  7,  8 ]

Heapify i=2: value=3, children: 7(idx5), 8(idx6). Largest=8 → swap 3↔8
           [ 4,  10,  8,  5,  1,  7,  3 ]
             Tree:        4
                        /   \
                      10     8   ← 8 rose up
                     / \   / \
                    5   1 7   3

Heapify i=1: value=10, children: 5(idx3), 1(idx4). 10 already largest. No swap.
           [ 4,  10,  8,  5,  1,  7,  3 ]

Heapify i=0: value=4, children: 10(idx1), 8(idx2). Largest=10 → swap 4↔10
           [ 10,  4,  8,  5,  1,  7,  3 ]
             Sift 4 down: children 5(idx3), 1(idx4). Largest=5 → swap 4↔5
           [ 10,  5,  8,  4,  1,  7,  3 ]
             Node 4 is a leaf. Stop.

Max-heap:  [ 10,  5,  8,  4,  1,  7,  3 ]

             Tree:        10         ← largest at root ✓
                        /    \
                       5      8
                      / \    / \
                     4   1  7   3
```

### Phase 2 — Extract max, restore heap, repeat

```
Heap size = 7. Sorted region grows from right.

Extract 1: Swap root(10) ↔ last(3). Shrink heap to 6.
           [ 3,  5,  8,  4,  1,  7 | 10 ]
             Heapify root 3: largest child=8(idx2) → swap
           [ 8,  5,  3,  4,  1,  7 | 10 ]
             Sift 3 down: largest child=7(idx5) → swap
           [ 8,  5,  7,  4,  1,  3 | 10 ]

Extract 2: Swap root(8) ↔ last(3). Shrink to 5.
           [ 3,  5,  7,  4,  1 | 8,  10 ]
             Heapify: 3→7→ done
           [ 7,  5,  3,  4,  1 | 8,  10 ]

Extract 3: Swap root(7) ↔ last(1). Shrink to 4.
           [ 1,  5,  3,  4 | 7,  8,  10 ]
             Heapify: 1→5→4 done
           [ 5,  4,  3,  1 | 7,  8,  10 ]

Extract 4: Swap root(5) ↔ last(1). Shrink to 3.
           [ 1,  4,  3 | 5,  7,  8,  10 ]
             Heapify: 1→4 done
           [ 4,  1,  3 | 5,  7,  8,  10 ]

Extract 5: Swap root(4) ↔ last(3). Shrink to 2.
           [ 3,  1 | 4,  5,  7,  8,  10 ]
             Heapify: 3>1. No swap.

Extract 6: Swap root(3) ↔ last(1). Shrink to 1.
           [ 1 | 3,  4,  5,  7,  8,  10 ]

Result:    [ 1,  3,  4,  5,  7,  8,  10 ] ✅
```

> **Key insight:** After each extraction, the heap shrinks by one and the sorted region grows by one. The `|` bar shows the boundary — everything to the right is permanently placed.

---

## The Algorithm

```
Phase 1 — Build max-heap:
  1. Start from the last non-leaf: i = n//2 - 1
  2. Call heapify(arr, i, n) for each i down to 0
  3. After this loop, arr[0] is the largest element

Phase 2 — Extract sorted elements:
  4. Swap arr[0] (max) with arr[end] (last element of heap)
  5. Shrink heap: end -= 1
  6. Call heapify(arr, 0, end) to restore the max-heap
  7. Repeat from step 4 until heap has one element
```

---

## Pseudocode

```
function heap_sort(arr):
    n = len(arr)

    # Phase 1: build max-heap bottom-up
    for i = n//2 - 1 down to 0:
        heapify(arr, i, n)

    # Phase 2: extract elements one by one
    for end = n-1 down to 1:
        swap(arr[0], arr[end])     ← move current max to sorted region
        heapify(arr, 0, end)       ← restore heap property

function heapify(arr, i, n):
    largest = i
    left    = 2*i + 1
    right   = 2*i + 2

    if left  < n and arr[left]  > arr[largest]: largest = left
    if right < n and arr[right] > arr[largest]: largest = right

    if largest != i:
        swap(arr[i], arr[largest])
        heapify(arr, largest, n)   ← sift down recursively
```

---

## Python Implementation

```python
def heap_sort(arr):
    n = len(arr)

    # Phase 1: build max-heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, i, n)

    # Phase 2: extract max elements one by one
    for end in range(n - 1, 0, -1):
        arr[0], arr[end] = arr[end], arr[0]   # move max to sorted region
        heapify(arr, 0, end)                  # restore heap for reduced range

    return arr


def heapify(arr, i, n):
    largest = i
    left  = 2 * i + 1
    right = 2 * i + 2

    if left  < n and arr[left]  > arr[largest]: largest = left
    if right < n and arr[right] > arr[largest]: largest = right

    if largest != i:
        arr[i], arr[largest] = arr[largest], arr[i]
        heapify(arr, largest, n)    # sift down the displaced element


# Example
arr = [4, 10, 3, 5, 1, 7, 8]
print("Before:", arr)
print("After: ", heap_sort(arr))
# After:  [1, 3, 4, 5, 7, 8, 10]
```

> **Tip:** The recursive `heapify` can be rewritten iteratively (a `while` loop sifting down) to avoid stack frames — preferred in production.

---

## Why O(n log n)?

```
Phase 1 — Build heap:
  Intuition: seems like O(n log n) since we call heapify O(n) times.
  Reality: O(n).
  Why? Most nodes are deep in the tree (leaves do no work).
  The sum converges: n/2 × O(1) + n/4 × O(log2) + ... = O(n).

Phase 2 — Extract:
  n-1 extractions × O(log n) heapify each = O(n log n)

Total: O(n) + O(n log n) = O(n log n)
```

The surprising part is that building the heap is only O(n) — a fact many people get wrong in interviews.

---

## Complexity

### Time

| Case         | Complexity      | Why                                               |
|--------------|-----------------|---------------------------------------------------|
| Best case    | **O(n log n)**  | Same structure regardless of input order          |
| Average case | **O(n log n)**  | Always n-1 extractions, each O(log n) heapify     |
| Worst case   | **O(n log n)**  | Guaranteed — no pathological pivot like Quick Sort |

### Space

| Metric          | Complexity | Why                                                |
|-----------------|------------|----------------------------------------------------|
| Auxiliary space | **O(1)**   | Everything happens in-place within the array       |
| Call stack      | **O(log n)**| Recursive heapify depth = tree height              |

> Heap Sort is the only algorithm in this series that guarantees **O(n log n) time AND O(1) auxiliary space**.

---

## Properties at a Glance

| Property         | Value   | Notes                                                       |
|------------------|---------|-------------------------------------------------------------|
| Stable           | ❌ No   | Swapping root to end can reorder equal elements             |
| In-Place         | ✅ Yes  | Sorts entirely within the input array                       |
| Adaptive         | ❌ No   | Always O(n log n) — ignores existing order                  |
| Comparison-Based | ✅ Yes  | All decisions made by comparing elements                    |
| Cache-Friendly   | ❌ No   | Heap traversal jumps around memory — poor locality          |

> **The cache-friendliness problem** is Heap Sort's main weakness in practice. Phase 2 accesses the root (index 0) and the last element repeatedly, but heapify jumps to `2i+1` and `2i+2` — non-sequential memory accesses that cause cache misses. This is why Quick Sort (sequential scans) is typically faster in practice despite the same asymptotic complexity.

---

## The Complete Sorting Algorithm Comparison

| Feature              | Bubble    | Selection | Insertion     | Merge Sort      | Quick Sort        | **Heap Sort**       |
|----------------------|-----------|-----------|---------------|-----------------|-------------------|---------------------|
| Best case            | O(n)      | O(n²)     | O(n)          | O(n log n)      | O(n log n)        | **O(n log n)**      |
| Average case         | O(n²)     | O(n²)     | O(n²)         | O(n log n)      | O(n log n)        | **O(n log n)**      |
| Worst case           | O(n²)     | O(n²)     | O(n²)         | O(n log n)      | ⚠️ O(n²)         | **O(n log n)**      |
| Auxiliary space      | O(1)      | O(1)      | O(1)          | O(n)            | O(log n)          | **O(1)**            |
| Stable               | ✅        | ❌        | ✅            | ✅              | ❌                | ❌                  |
| In-place             | ✅        | ✅        | ✅            | ❌              | ✅                | ✅                  |
| Cache-friendly       | ✗         | ✗         | ✓✓            | ✗               | ✅✅              | ✗                   |
| Guaranteed O(n logn) | ❌        | ❌        | ❌            | ✅              | ❌                | ✅                  |

> **Heap Sort's unique position:** It's the only algorithm with guaranteed O(n log n) AND O(1) extra space. This makes it the fallback in Introsort — C++'s `std::sort` switches to Heap Sort when Quick Sort's recursion depth exceeds 2·log(n), preventing the O(n²) worst case.

---

## Heap Sort vs Merge Sort vs Quick Sort

```
                   Worst case    Space    Stable   Cache
Merge Sort          O(n log n)   O(n)     Yes      Poor
Quick Sort          O(n²) ⚠️    O(log n)  No      Excellent
Heap Sort           O(n log n)   O(1)     No       Poor

→ Need guaranteed O(n log n) + O(1) space?  → Heap Sort
→ Need guaranteed O(n log n) + stability?   → Merge Sort
→ Need fastest average-case?                → Quick Sort
→ All three combined?                       → Introsort (Quick + Heap + Insertion)
```

---

## Real-World Uses

```
1. Priority queues — the heap data structure itself (not the sort) is used
   everywhere: Dijkstra's algorithm, A* pathfinding, task schedulers,
   OS process scheduling, event-driven simulations.

2. Introsort fallback — C++ std::sort switches to Heap Sort when
   Quick Sort's recursion exceeds 2*log(n) to guarantee O(n log n).

3. Memory-constrained sorting — when you cannot allocate O(n) extra
   space (embedded systems, real-time systems), Heap Sort gives O(n log n)
   with zero auxiliary memory.

4. Finding the k largest elements — build a min-heap of size k,
   scan the array once. O(n log k) — faster than full sort.

5. Median maintenance — two heaps (max-heap for lower half, min-heap
   for upper half) maintain a running median in O(log n) per element.
```

---

## Pros and Cons

### ✅ Advantages
- Guaranteed **O(n log n)** in all cases — no worst-case surprise
- **O(1) auxiliary space** — unique among O(n log n) algorithms
- No extra memory allocation — critical for embedded/real-time systems
- The heap structure itself is extremely useful beyond just sorting

### ❌ Disadvantages
- **Not stable** — equal elements can be reordered
- **Poor cache performance** — heap traversal has bad memory locality
- Slower than Quick Sort in practice on random in-memory data (2–5× slower)
- Not adaptive — always O(n log n) even if input is nearly sorted

---

## When to Use It

```
Heap Sort is the right choice when:
  ✓ You need guaranteed O(n log n) with O(1) extra space
  ✓ Memory allocation is forbidden (embedded, real-time systems)
  ✓ You're implementing the fallback in a hybrid sort (à la Introsort)
  ✓ You need the k largest elements efficiently

Heap Sort is the wrong choice when:
  ✗ Raw speed on random data — Quick Sort is faster in practice
  ✗ Stability required — use Merge Sort or Tim Sort
  ✗ Data is nearly sorted — use Insertion Sort
  ✗ Cache performance matters — Heap's random access is a bottleneck
```

---

## Common Interview Questions

**Q: Why is building a heap O(n) and not O(n log n)?**
Because we build bottom-up starting from the last non-leaf. Leaf nodes (half the tree) do zero work. The work per node is proportional to its height, and the sum of all heights in a complete binary tree is O(n). Most nodes are near the bottom doing very little.

**Q: Is Heap Sort stable?**
No. The extraction step swaps the root (current max) with the last heap element, which can move equal elements past each other and break their original order.

**Q: What's the difference between Heap Sort and using a heap to sort?**
Same algorithm — Heap Sort is simply the in-place version. A "heap-based sort" might allocate a separate heap structure (O(n) space) and extract elements from it. Heap Sort avoids that extra allocation by treating the input array itself as the heap.

**Q: Why is Heap Sort slower than Quick Sort in practice if they're both O(n log n)?**
Cache behaviour. Quick Sort's partition step scans the array sequentially — every access is likely cached. Heap Sort's heapify jumps to `2i+1` and `2i+2` — for large arrays these indices land far apart in memory, causing frequent cache misses. The asymptotic complexity is the same; the constant factor is not.

**Q: Where is Heap Sort used in real production code?**
Primarily as the safety net in Introsort (C++ `std::sort`): Quick Sort runs normally, but if recursion depth exceeds 2·log(n), it switches to Heap Sort to guarantee O(n log n) worst case. It's the "insurance policy" algorithm.

**Q: How would you find the k-th largest element efficiently using a heap?**
Build a min-heap of size k from the first k elements, then scan the rest: if `arr[i] > heap[0]`, replace the root and heapify. After scanning, the root is the k-th largest. Time: O(n log k). Better than full sort when k << n.

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  HEAP SORT —                                                      │
│                                                                   │
│  Strategy:  Build max-heap → extract max n-1 times                │
│  Two phases: O(n) build + O(n log n) extraction                   │
│                                                                   │
│  Time  →  Always O(n log n) — best, average, and worst            │
│  Space →  O(1) auxiliary — unique among O(n log n) sorts          │
│                                                                   │
│  Stable: No   In-Place: Yes   Cache-Friendly: No                  │
│                                                                   │
│  Use for: guaranteed O(n log n) + no extra memory                 │
│  Used in: Introsort (C++ std::sort) as the fallback               │
│  Heap itself used in: Dijkstra, A*, priority queues               │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 6 of daily algorithm commits — next up: Binary Search*