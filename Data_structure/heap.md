# 🏔️ Heap (Data Structure)

> *"A tree that always knows its most important element — and tells you in O(1)."*


---

## What is a Heap?

A **Heap** is a specialised **complete binary tree** that satisfies the **heap property**:

- **Max-Heap** — every parent is ≥ its children. Root = largest element.
- **Min-Heap** — every parent is ≤ its children. Root = smallest element.

The heap is NOT fully sorted — it only guarantees the root is the extreme (max or min). This partial order is exactly what makes it powerful: you can find the maximum (or minimum) in **O(1)**, and inserting or extracting it takes only **O(log n)**.

> The heap is also the backing structure for **Heap Sort** and **Priority Queues** — two of the most important applications in computer science.

---

## Heap vs Binary Search Tree


```
Max-Heap:                      BST:
        90                           50
       /  \                         /  \
      70   80                      30   70
     / \  / \                     / \ / \
   50 60 65  75                  20 40 60 80

Heap: parent ≥ both children    BST: left < parent < right
      (no left-right ordering)       (full left-right ordering)
      Root = max, O(1)               No O(1) max/min
      Insert/Extract = O(log n)      Search any key = O(log n)
```

---

## Storing a Heap as an Array

A complete binary tree maps perfectly into an array — no pointers needed.


```
Max-Heap tree:               Array representation:
        90                   index: 0   1   2   3   4   5   6
       /  \                  value:[90, 70, 80, 50, 60, 65, 75]
      70   80
     / \  / \
   50 60 65  75

Index relationships:
  Parent of i     →  (i - 1) // 2
  Left child of i →  2*i + 1
  Right child of i→  2*i + 2

Examples:
  Parent of index 3 (value 50) → (3-1)//2 = 1 (value 70) ✓
  Left child of index 1 (value 70) → 2*1+1 = 3 (value 50) ✓
  Right child of index 1 (value 70) → 2*1+2 = 4 (value 60) ✓
```

---

## Core Operations


### Sift Up (Bubble Up) — used after Insert

```
Insert 95 into max-heap:

Step 1: Add to end
  [90, 70, 80, 50, 60, 65, 75, 95]
                                ↑ new

Step 2: Compare with parent (index 3, value 50): 95 > 50 → swap
  [90, 70, 80, 95, 60, 65, 75, 50]

Step 3: Compare with parent (index 1, value 70): 95 > 70 → swap
  [90, 95, 80, 70, 60, 65, 75, 50]

Step 4: Compare with parent (index 0, value 90): 95 > 90 → swap
  [95, 90, 80, 70, 60, 65, 75, 50]

Step 5: At root. Done. ✅
```

### Sift Down (Heapify Down) — used after Extract

```
Extract max (90) from max-heap [90, 70, 80, 50, 60, 65, 75]:

Step 1: Swap root with last element, remove last
  [75, 70, 80, 50, 60, 65] ← 90 removed

Step 2: Sift 75 down. Children: 70 (idx 1), 80 (idx 2). Largest = 80 → swap
  [80, 70, 75, 50, 60, 65]

Step 3: Sift 75 down. Children: 65 (idx 5). 75 > 65. No swap. Done. ✅
  [80, 70, 75, 50, 60, 65]
```

---

## Python Implementation (Min-Heap)

```python
class MinHeap:
    def __init__(self):
        self._data = []

    def push(self, val):
        """O(log n) — add element and sift up"""
        self._data.append(val)
        self._sift_up(len(self._data) - 1)

    def pop(self):
        """O(log n) — remove and return minimum (root)"""
        if not self._data:
            raise IndexError("Pop from empty heap")
        self._swap(0, len(self._data) - 1)
        val = self._data.pop()
        self._sift_down(0)
        return val

    def peek(self):
        """O(1) — view minimum without removing"""
        if not self._data:
            raise IndexError("Peek at empty heap")
        return self._data[0]

    def _sift_up(self, i):
        while i > 0:
            parent = (i - 1) // 2
            if self._data[parent] > self._data[i]:
                self._swap(parent, i)
                i = parent
            else:
                break

    def _sift_down(self, i):
        n = len(self._data)
        while True:
            smallest = i
            left, right = 2*i+1, 2*i+2
            if left < n and self._data[left] < self._data[smallest]:
                smallest = left
            if right < n and self._data[right] < self._data[smallest]:
                smallest = right
            if smallest == i:
                break
            self._swap(i, smallest)
            i = smallest

    def _swap(self, i, j):
        self._data[i], self._data[j] = self._data[j], self._data[i]

    def __len__(self):
        return len(self._data)

    def __repr__(self):
        return f"MinHeap({self._data})"


# Usage
h = MinHeap()
h.push(10); h.push(4); h.push(7); h.push(1); h.push(9)
print(h)           # MinHeap([1, 4, 7, 10, 9])
print(h.peek())    # 1  ← minimum always at root
print(h.pop())     # 1
print(h.peek())    # 4
```

---

## Python's `heapq` Module

Python provides a **min-heap** implementation in the standard library. For a max-heap, negate the values.

```python
import heapq

# ── Min-Heap ──────────────────────────────────────────────────────
h = []
heapq.heappush(h, 10)
heapq.heappush(h, 4)
heapq.heappush(h, 7)
heapq.heappush(h, 1)
print(h)                       # [1, 4, 7, 10]  (heap-ordered array)
print(heapq.heappop(h))        # 1   ← smallest
print(heapq.heappop(h))        # 4

# Build heap from list in O(n) — faster than n pushes
nums = [10, 4, 7, 1, 9, 3]
heapq.heapify(nums)            # in-place, O(n)
print(nums)                    # [1, 4, 3, 10, 9, 7]

# ── Max-Heap (negate values) ──────────────────────────────────────
max_heap = []
for val in [10, 4, 7, 1, 9]:
    heapq.heappush(max_heap, -val)     # store negated
print(-heapq.heappop(max_heap))        # 10 ← largest

# ── Useful heapq utilities ────────────────────────────────────────
nums = [10, 4, 7, 1, 9, 3]
print(heapq.nlargest(3, nums))    # [10, 9, 7]  — O(n log k)
print(heapq.nsmallest(3, nums))   # [1, 3, 4]   — O(n log k)

# ── Priority Queue with (priority, value) tuples ──────────────────
pq = []
heapq.heappush(pq, (2, "medium"))
heapq.heappush(pq, (1, "urgent"))
heapq.heappush(pq, (3, "low"))
print(heapq.heappop(pq))          # (1, 'urgent')
```

---

## Complexity

### Time

| Operation       | Complexity  | Notes                             |
|-----------------|-------------|-----------------------------------|
| Peek (min/max)  | **O(1)**    | Root is always the extreme        |
| Push (insert)   | **O(log n)**| Sift up at most height levels     |
| Pop (extract)   | **O(log n)**| Sift down at most height levels   |
| Build heap      | **O(n)**    | Bottom-up heapify, not O(n log n) |
| Search          | O(n)        | Heap is not fully sorted          |
| k largest/smallest | O(n log k)| heapq.nlargest / nsmallest       |

### Space

| Metric  | Complexity | Notes                     |
|---------|-----------|---------------------------|
| Storage | **O(n)**  | Array of n elements       |

---

## Properties at a Glance

| Property           | Value   | Notes                                        |
|--------------------|---------|----------------------------------------------|
| Complete binary tree| ✅ Yes | Always fills left to right at each level     |
| Fully sorted       | ❌ No  | Only root is guaranteed to be min or max     |
| Stable             | ❌ No  | Equal elements can be reordered              |
| In-place           | ✅ Yes | Array-based, no extra pointers               |
| Search by value    | O(n)   | Not a search structure — use BST or HashMap  |

---

## Classic Heap Patterns

### Pattern 1 — K Largest Elements

```python
def k_largest(nums, k):
    # Min-heap of size k: root is the smallest of the k largest
    heap = nums[:k]
    heapq.heapify(heap)
    for num in nums[k:]:
        if num > heap[0]:
            heapq.heapreplace(heap, num)   # O(log k)
    return sorted(heap, reverse=True)

print(k_largest([3,1,5,12,2,11,7], 3))   # [12, 11, 7]
```

### Pattern 2 — Merge K Sorted Lists

```python
def merge_k_sorted(lists):
    heap, result = [], []
    for i, lst in enumerate(lists):
        if lst:
            heapq.heappush(heap, (lst[0], i, 0))   # (value, list_idx, elem_idx)
    while heap:
        val, li, ei = heapq.heappop(heap)
        result.append(val)
        if ei + 1 < len(lists[li]):
            heapq.heappush(heap, (lists[li][ei+1], li, ei+1))
    return result

print(merge_k_sorted([[1,4,7],[2,5,8],[3,6,9]]))
# [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### Pattern 3 — Running Median

Maintain a max-heap for the lower half and a min-heap for the upper half.

```python
class MedianFinder:
    def __init__(self):
        self.lo = []   # max-heap (negate values)
        self.hi = []   # min-heap

    def add(self, num):
        heapq.heappush(self.lo, -num)               # always push to lo first
        heapq.heappush(self.hi, -heapq.heappop(self.lo))  # balance
        if len(self.hi) > len(self.lo):             # keep lo >= hi in size
            heapq.heappush(self.lo, -heapq.heappop(self.hi))

    def median(self):
        if len(self.lo) > len(self.hi):
            return -self.lo[0]
        return (-self.lo[0] + self.hi[0]) / 2
```

---

## Heap vs Other Structures


| Feature            | Heap         | BST (balanced) | Sorted Array |
|--------------------|-------------|---------------|--------------|
| Find min/max       | **O(1)**    | O(log n)      | O(1)         |
| Insert             | **O(log n)**| O(log n)      | O(n)         |
| Delete min/max     | **O(log n)**| O(log n)      | O(1)         |
| Delete arbitrary   | O(n)        | O(log n)      | O(n)         |
| Search by value    | O(n)        | O(log n)      | O(log n)     |
| Build from list    | **O(n)**    | O(n log n)    | O(n log n)   |
| Sorted iteration   | O(n log n)  | O(n)          | O(n)         |

---

## Real-World Uses

```
1. Priority Queues      — OS task scheduling, hospital triage, Dijkstra's algorithm
2. Heap Sort            — in-place O(n log n) sorting algorithm
3. K-way merge          — merging k sorted files/streams efficiently
4. Median maintenance   — streaming data, running statistics
5. Dijkstra's algorithm — min-heap extracts the closest unvisited node
6. A* pathfinding       — priority queue on estimated cost
7. Event simulation     — events processed in time order (min-heap on timestamp)
8. Top-K problems       — leaderboards, trending items, log analysis
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  HEAP — AT A GLANCE                                               │
│                                                                   │
│  Structure:  Complete binary tree stored as an array             │
│  Property:   Parent ≤ children (min-heap) / ≥ children (max-heap)│
│                                                                   │
│  Peek    → O(1)    — root is always min or max                   │
│  Push    → O(log n)— sift up                                     │
│  Pop     → O(log n)— sift down                                   │
│  Build   → O(n)    — bottom-up heapify                           │
│                                                                   │
│  Python: heapq module (min-heap); negate for max-heap            │
│                                                                   │
│  Key patterns:                                                    │
│    K largest/smallest  — heap of size k, O(n log k)              │
│    Merge k sorted lists — heap of (value, list_idx)              │
│    Running median      — two heaps (max + min)                   │
│                                                                   │
│  Used in: priority queues, Dijkstra, A*, Heap Sort, top-K        │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 17 of daily algorithm commits — next up: Trie (Data Structure)*
