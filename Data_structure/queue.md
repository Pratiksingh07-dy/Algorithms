# 🎟️ Queue

> *"The fairest data structure — whoever arrives first, leaves first."*

---

## What is a Queue?

A Queue is a **linear data structure** that follows the **FIFO** principle — **First In, First Out**.

Think of a real queue at a ticket counter: the person who joins first gets served first. No cutting in line, no going from the back. Orderly, sequential, fair.

Four core operations:
- **Enqueue** — add an element to the **rear** (back)
- **Dequeue** — remove an element from the **front**
- **Peek / Front** — view the front element without removing
- **Is Empty** — check if the queue has any elements

The Queue is the natural data structure for anything that needs to be processed in the order it arrived — network packets, print jobs, BFS traversal, task scheduling.

---

## Visualising a Queue

```
                    ENQUEUE (add at rear)
                         │
                         ▼
FRONT                                           REAR
  │                                              │
  ▼                                              ▼
┌──────┬──────┬──────┬──────┐       ┌──────┬──────┬──────┬──────┬──────┐
│  10  │  20  │  30  │  40  │  →→→  │  10  │  20  │  30  │  40  │  50  │
└──────┴──────┴──────┴──────┘       └──────┴──────┴──────┴──────┴──────┘
  ▲                                    ▲
  │                                    │
DEQUEUE (remove from front)       After enqueue(50)


After dequeue():  returns 10
┌──────┬──────┬──────┬──────┐
│  20  │  30  │  40  │  50  │
└──────┴──────┴──────┴──────┘
  ▲
FRONT
```

---

## Implementation 1 — Using `collections.deque`

Python's `deque` (double-ended queue) is the standard and most efficient queue implementation. Both `append` (enqueue) and `popleft` (dequeue) are O(1).

```python
from collections import deque


class Queue:
    def __init__(self):
        self._data = deque()

    def enqueue(self, item):
        """O(1) — append to right end"""
        self._data.append(item)

    def dequeue(self):
        """O(1) — remove from left end"""
        if self.is_empty():
            raise IndexError("Dequeue from empty queue")
        return self._data.popleft()

    def peek(self):
        """O(1) — view front without removing"""
        if self.is_empty():
            raise IndexError("Peek at empty queue")
        return self._data[0]

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

    def __repr__(self):
        return f"Queue(front → {list(self._data)} ← rear)"


# Usage
q = Queue()
q.enqueue(10)
q.enqueue(20)
q.enqueue(30)
print(q)            # Queue(front → [10, 20, 30] ← rear)
print(q.dequeue())  # 10
print(q.peek())     # 20
print(q.size())     # 2
```

> **Why not use a plain list?** `list.pop(0)` is O(n) — it shifts every element left. `deque.popleft()` is O(1). Always use `deque` for queues in Python.

---

## Implementation 2 — Using a Linked List

True O(1) for all operations with no amortised asterisk.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class Queue:
    def __init__(self):
        self._front = None    # dequeue from here
        self._rear  = None    # enqueue to here
        self._size  = 0

    def enqueue(self, item):
        """O(1) — add to rear using tail pointer"""
        node = Node(item)
        if self._rear:
            self._rear.next = node
        self._rear = node
        if not self._front:
            self._front = node
        self._size += 1

    def dequeue(self):
        """O(1) — remove from front"""
        if self.is_empty():
            raise IndexError("Dequeue from empty queue")
        val = self._front.data
        self._front = self._front.next
        if not self._front:
            self._rear = None
        self._size -= 1
        return val

    def peek(self):
        if self.is_empty():
            raise IndexError("Peek at empty queue")
        return self._front.data

    def is_empty(self):
        return self._front is None

    def size(self):
        return self._size
```

---

## Implementation 3 — Circular Queue (Fixed Size)

A circular queue (ring buffer) uses a fixed-size array and wraps indices around using modulo arithmetic. Used in embedded systems, OS buffers, and audio streaming where memory is fixed.

```python
class CircularQueue:
    def __init__(self, capacity):
        self._data     = [None] * capacity
        self._capacity = capacity
        self._front    = 0
        self._rear     = 0
        self._size     = 0

    def enqueue(self, item):
        if self._size == self._capacity:
            raise OverflowError("Queue is full")
        self._data[self._rear] = item
        self._rear = (self._rear + 1) % self._capacity   # wrap around
        self._size += 1

    def dequeue(self):
        if self.is_empty():
            raise IndexError("Dequeue from empty queue")
        val = self._data[self._front]
        self._front = (self._front + 1) % self._capacity  # wrap around
        self._size -= 1
        return val

    def peek(self):
        if self.is_empty():
            raise IndexError("Queue is empty")
        return self._data[self._front]

    def is_empty(self):
        return self._size == 0

    def is_full(self):
        return self._size == self._capacity
```

```
Circular buffer visualised (capacity = 5):

Initial:  [ _ , _ , _ , _ , _ ]
           ↑F,R

After enqueue(A,B,C):
          [ A , B , C , _ , _ ]
           ↑F           ↑R

After dequeue() → A:
          [ _ , B , C , _ , _ ]
                ↑F       ↑R

After enqueue(D,E):
          [ _ , B , C , D , E ]
                ↑F           ↑R (= 0, wraps)

After enqueue(F) (wraps rear to 0):
          [ F , B , C , D , E ]
           ↑R  ↑F
Queue full!
```

---

## Operations Complexity

| Operation  | deque-based | Linked List | Circular (array) |
|------------|-------------|-------------|------------------|
| Enqueue    | O(1)        | O(1)        | O(1)             |
| Dequeue    | O(1)        | O(1)        | O(1)             |
| Peek       | O(1)        | O(1)        | O(1)             |
| Is Empty   | O(1)        | O(1)        | O(1)             |
| Space      | O(n)        | O(n)        | O(capacity)      |

---

## Queue Variants

### Priority Queue

Elements dequeue in order of priority, not arrival. Implemented with a heap.

```python
import heapq

pq = []
heapq.heappush(pq, (2, "medium priority"))
heapq.heappush(pq, (1, "high priority"))
heapq.heappush(pq, (3, "low priority"))

print(heapq.heappop(pq))   # (1, 'high priority') ← highest priority first
```

### Deque (Double-Ended Queue)

Add and remove from both ends in O(1). The generalisation of both Stack and Queue.

```python
from collections import deque

dq = deque()
dq.append(1)        # add to right
dq.appendleft(0)    # add to left
dq.pop()            # remove from right
dq.popleft()        # remove from left
```

### Monotonic Deque

A deque maintained in monotonic order — used to find sliding window maximum/minimum in O(n).

---

## BFS — The Queue's Killer Application

Breadth-First Search traverses a graph or tree **level by level** — exactly what a queue enables. This is the most important use of a queue in algorithms.

```python
from collections import deque

def bfs(graph, start):
    visited = set()
    queue   = deque([start])
    visited.add(start)
    order   = []

    while queue:
        node = queue.popleft()          # process in arrival order
        order.append(node)

        for neighbour in graph[node]:
            if neighbour not in visited:
                visited.add(neighbour)
                queue.append(neighbour)

    return order


graph = {
    'A': ['B', 'C'],
    'B': ['D', 'E'],
    'C': ['F'],
    'D': [], 'E': [], 'F': []
}

print(bfs(graph, 'A'))   # ['A', 'B', 'C', 'D', 'E', 'F']
```

```
BFS traversal order:

        A           ← Level 0: process A, enqueue B, C
       / \
      B   C         ← Level 1: process B→enqueue D,E; process C→enqueue F
     / \   \
    D   E   F       ← Level 2: process D, E, F

Queue state:
  Start:  [A]
  Pop A:  [B, C]
  Pop B:  [C, D, E]
  Pop C:  [D, E, F]
  Pop D:  [E, F]
  Pop E:  [F]
  Pop F:  []  → done
```

---

## Classic Interview Problems

### 1. Generate binary numbers from 1 to n

```python
def generate_binary(n):
    result = []
    q = deque(["1"])
    for _ in range(n):
        front = q.popleft()
        result.append(front)
        q.append(front + "0")
        q.append(front + "1")
    return result

print(generate_binary(7))
# ['1', '10', '11', '100', '101', '110', '111']
```

### 2. Implement Stack using two Queues

```python
class StackUsingQueues:
    def __init__(self):
        self.q1 = deque()
        self.q2 = deque()

    def push(self, val):
        self.q2.append(val)
        while self.q1:                   # drain q1 into q2
            self.q2.append(self.q1.popleft())
        self.q1, self.q2 = self.q2, self.q1   # swap names

    def pop(self):
        return self.q1.popleft()

    def peek(self):
        return self.q1[0]
```

### 3. Sliding Window Maximum

Find the maximum in every window of size k — O(n) using a monotonic deque.

```python
def sliding_window_max(arr, k):
    dq = deque()    # stores indices, decreasing order of values
    result = []

    for i, val in enumerate(arr):
        # remove indices outside the window
        while dq and dq[0] < i - k + 1:
            dq.popleft()
        # remove indices with smaller values (they can never be the max)
        while dq and arr[dq[-1]] < val:
            dq.pop()
        dq.append(i)
        if i >= k - 1:
            result.append(arr[dq[0]])   # front is always the max

    return result

print(sliding_window_max([1,3,-1,-3,5,3,6,7], 3))
# [3, 3, 5, 5, 6, 7]
```

### 4. Rotting Oranges (multi-source BFS)

```python
def oranges_rotting(grid):
    from collections import deque
    rows, cols = len(grid), len(grid[0])
    queue = deque()
    fresh = 0

    for r in range(rows):
        for c in range(cols):
            if grid[r][c] == 2:
                queue.append((r, c, 0))   # (row, col, minutes)
            elif grid[r][c] == 1:
                fresh += 1

    minutes = 0
    dirs = [(0,1),(0,-1),(1,0),(-1,0)]

    while queue:
        r, c, t = queue.popleft()
        for dr, dc in dirs:
            nr, nc = r+dr, c+dc
            if 0<=nr<rows and 0<=nc<cols and grid[nr][nc]==1:
                grid[nr][nc] = 2
                fresh -= 1
                minutes = max(minutes, t+1)
                queue.append((nr, nc, t+1))

    return minutes if fresh == 0 else -1
```

---

## Queue vs Stack vs Deque

| Feature         | Stack (LIFO)   | Queue (FIFO)   | Deque             |
|-----------------|----------------|----------------|-------------------|
| Add element     | Push to top    | Enqueue to rear| Either end O(1)   |
| Remove element  | Pop from top   | Dequeue from front | Either end O(1)|
| Access          | Top only       | Front only     | Both ends         |
| Real world      | Undo, DFS      | BFS, scheduling| Sliding window    |
| Python type     | list / deque   | deque          | collections.deque |

---

## Real-World Uses

```
1. CPU task scheduling       — round-robin: each process gets a time slice
2. Printer queue             — documents printed in arrival order
3. BFS traversal             — shortest path, level-order tree traversal
4. Network packet buffering  — router queues incoming packets
5. Web server request queue  — handle requests in arrival order
6. Keyboard input buffer     — keystrokes queued for the OS to process
7. Breadth-first crawling    — web crawler visits pages level by level
8. Circular buffer (audio)   — fixed-size ring buffer for audio streaming
```

---

## Pros and Cons

### ✅ Advantages
- O(1) enqueue and dequeue with `deque` or linked list
- Fair and predictable — FIFO ordering is intuitive
- Foundation of BFS and level-order traversal
- Circular variant uses fixed memory — no dynamic allocation

### ❌ Disadvantages
- No random access — can only see the front element
- List-based queue has O(n) dequeue — must use `deque`
- No priority — all elements treated equally (use a priority queue for priority-based processing)

---

## When to Use It

```
Use a Queue when:
  ✓ Order of processing must match order of arrival (FIFO)
  ✓ Implementing BFS or level-order traversal
  ✓ Task scheduling, request handling, buffering
  ✓ Sliding window problems (monotonic deque variant)
  ✓ Producer-consumer problems

Don't use a Queue when:
  ✗ You need LIFO ordering → use a Stack
  ✗ You need priority-based processing → use a Priority Queue (heap)
  ✗ You need random access → use an array or deque
```

---

<p align="center">
  <img src="./assets/Queue-Data-Structures.png" alt="Queue" width="800">
</p>
* reference to Geeksforgeeks

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  QUEUE — AT A GLANCE                                              │
│                                                                   │
│  Principle:  FIFO — First In, First Out                          │
│  Core ops:   Enqueue O(1)  |  Dequeue O(1)  |  Peek O(1)        │
│                                                                   │
│  Implement with:  collections.deque (Python standard)            │
│  Variants:  Circular Queue, Priority Queue, Monotonic Deque      │
│                                                                   │
│  Key patterns:                                                    │
│    • BFS traversal        — queue processes nodes level by level │
│    • Sliding window max   — monotonic deque, O(n)               │
│    • Multi-source BFS     — enqueue all sources at once          │
│    • Stack using 2 queues — classic interview problem            │
│                                                                   │
│  Used in: BFS, OS scheduling, network buffers, audio streaming   │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 13 of daily algorithm commits — next up: Hash Table (Data Structure)*
