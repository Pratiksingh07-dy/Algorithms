# 🔗 Linked List

> *"An array remembers where everything is. A linked list remembers only where to go next."*

---

## What is a Linked List?

A Linked List is a **linear data structure** where elements (called **nodes**) are stored in memory and connected through **pointers** — each node holds a value and a reference to the next node.

Unlike arrays, linked list nodes are **not stored contiguously in memory**. The structure is built dynamically: add a node anywhere, connect the pointers, done. No shifting, no resizing, no pre-allocation.

Each node contains:
- **Data** — the value stored
- **Next** — a pointer to the next node (and `prev` in doubly linked lists)

The list is accessed through the **head** pointer — lose the head, lose the entire list.

---

## Types of Linked Lists

### 1. Singly Linked List
Each node points only to the next node. Traversal is one-directional.

```
HEAD
 │
 ▼
┌──────┬──────┐   ┌──────┬──────┐   ┌──────┬──────┐   ┌──────┬──────┐
│  10  │  ──────►  │  20  │  ──────►  │  30  │  ──────►  │  40  │ NULL │
└──────┴──────┘   └──────┴──────┘   └──────┴──────┘   └──────┴──────┘
  Node 1              Node 2              Node 3              Node 4
```

### 2. Doubly Linked List
Each node points to both the next and previous node. Traversal is bidirectional.

```
HEAD                                                              TAIL
 │                                                                 │
 ▼                                                                 ▼
┌──────┬──────┬──────┐   ┌──────┬──────┬──────┐   ┌──────┬──────┬──────┐
│ NULL │  10  │  ──────►  │ ◄──  │  20  │  ──────►  │ ◄──  │  30  │ NULL │
└──────┴──────┴──────┘   └──────┴──────┴──────┘   └──────┴──────┴──────┘
  prev  data   next         prev  data   next         prev  data   next
```

### 3. Circular Linked List
The last node's `next` pointer points back to the head — no NULL terminal.

```
 ┌──────────────────────────────────────────────┐
 │                                              │
 ▼                                              │
┌──────┬──────┐   ┌──────┬──────┐   ┌──────┬──┴───┐
│  10  │  ──────►  │  20  │  ──────►  │  30  │  ──► │
└──────┴──────┘   └──────┴──────┘   └──────┴──────┘
```

---

## Node Structure

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None        # singly linked
        # self.prev = None      # add for doubly linked
```

---

## Singly Linked List — Full Implementation

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class LinkedList:
    def __init__(self):
        self.head = None

    # ── Insertion ─────────────────────────────────────────────────

    def insert_at_head(self, data):
        """O(1) — new node becomes the new head"""
        new_node = Node(data)
        new_node.next = self.head
        self.head = new_node

    def insert_at_tail(self, data):
        """O(n) — must traverse to reach the last node"""
        new_node = Node(data)
        if not self.head:
            self.head = new_node
            return
        curr = self.head
        while curr.next:
            curr = curr.next
        curr.next = new_node

    def insert_at_position(self, data, pos):
        """O(n) — traverse to pos-1, rewire pointers"""
        if pos == 0:
            self.insert_at_head(data)
            return
        new_node = Node(data)
        curr = self.head
        for _ in range(pos - 1):
            if not curr:
                raise IndexError("Position out of range")
            curr = curr.next
        new_node.next = curr.next
        curr.next = new_node

    # ── Deletion ──────────────────────────────────────────────────

    def delete_head(self):
        """O(1) — move head pointer forward"""
        if not self.head:
            return
        self.head = self.head.next

    def delete_by_value(self, data):
        """O(n) — find node with value, rewire around it"""
        if not self.head:
            return
        if self.head.data == data:
            self.head = self.head.next
            return
        curr = self.head
        while curr.next:
            if curr.next.data == data:
                curr.next = curr.next.next   # skip the node
                return
            curr = curr.next

    def delete_at_position(self, pos):
        """O(n) — traverse to pos-1, skip the node at pos"""
        if pos == 0:
            self.delete_head()
            return
        curr = self.head
        for _ in range(pos - 1):
            if not curr:
                raise IndexError("Position out of range")
            curr = curr.next
        if not curr.next:
            raise IndexError("Position out of range")
        curr.next = curr.next.next

    # ── Search ────────────────────────────────────────────────────

    def search(self, data):
        """O(n) — scan from head until found or end"""
        curr = self.head
        idx = 0
        while curr:
            if curr.data == data:
                return idx
            curr = curr.next
            idx += 1
        return -1

    # ── Traversal ─────────────────────────────────────────────────

    def display(self):
        """O(n) — print all nodes"""
        elements = []
        curr = self.head
        while curr:
            elements.append(str(curr.data))
            curr = curr.next
        print(" → ".join(elements) + " → NULL")

    def to_list(self):
        result, curr = [], self.head
        while curr:
            result.append(curr.data)
            curr = curr.next
        return result

    def length(self):
        """O(n) — count all nodes"""
        count, curr = 0, self.head
        while curr:
            count += 1
            curr = curr.next
        return count

    # ── Reversal ──────────────────────────────────────────────────

    def reverse(self):
        """O(n) time, O(1) space — classic three-pointer technique"""
        prev, curr = None, self.head
        while curr:
            next_node  = curr.next   # save next
            curr.next  = prev        # reverse pointer
            prev       = curr        # advance prev
            curr       = next_node   # advance curr
        self.head = prev
```

---

## Visualising Pointer Rewiring

### Insert at position 2 (between 20 and 30)

```
Before:
HEAD → [10] → [20] → [30] → [40] → NULL
                              ↑
                         insert [25] here

Step 1: Traverse to position 1 (node with value 20)
Step 2: new_node.next = curr.next   →  [25] → [30]
Step 3: curr.next     = new_node    →  [20] → [25]

After:
HEAD → [10] → [20] → [25] → [30] → [40] → NULL
```

### Delete node with value 30

```
Before:
HEAD → [10] → [20] → [30] → [40] → NULL
                       ↑
                    delete this

Step 1: Find node before target: curr = [20]
Step 2: curr.next = curr.next.next  →  [20] → [40]
        Node [30] is now unreferenced → garbage collected

After:
HEAD → [10] → [20] → [40] → NULL
```

### Reverse a linked list

```
Before:
HEAD → [10] → [20] → [30] → NULL

Iteration 1: prev=NULL,   curr=[10]  →  [10] → NULL,  prev=[10]
Iteration 2: prev=[10],   curr=[20]  →  [20] → [10],  prev=[20]
Iteration 3: prev=[20],   curr=[30]  →  [30] → [20],  prev=[30]
curr = NULL → stop. head = prev = [30]

After:
HEAD → [30] → [20] → [10] → NULL
```

---

## Operations Complexity

| Operation          | Singly LL  | Doubly LL  | Array      |
|--------------------|-----------|-----------|------------|
| Access by index    | **O(n)**  | **O(n)**  | O(1)       |
| Insert at head     | **O(1)**  | **O(1)**  | O(n)       |
| Insert at tail     | O(n)*     | **O(1)**** | O(1) amort|
| Insert at middle   | O(n)      | O(n)      | O(n)       |
| Delete at head     | **O(1)**  | **O(1)**  | O(n)       |
| Delete at tail     | O(n)      | **O(1)**  | O(1) amort |
| Delete at middle   | O(n)      | O(n)      | O(n)       |
| Search             | O(n)      | O(n)      | O(n)       |
| Binary search      | ❌ N/A    | ❌ N/A    | O(log n)   |

> *Singly LL tail insert is O(1) if you maintain a `tail` pointer.
> **Doubly LL has O(1) tail operations because `prev` lets you unlink the tail instantly.

---

## Doubly Linked List Implementation (Key Differences)

```python
class DNode:
    def __init__(self, data):
        self.data = data
        self.prev = None
        self.next = None


class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None

    def insert_at_tail(self, data):
        """O(1) — tail pointer means no traversal needed"""
        new_node = DNode(data)
        if not self.tail:
            self.head = self.tail = new_node
            return
        new_node.prev = self.tail
        self.tail.next = new_node
        self.tail = new_node

    def delete_tail(self):
        """O(1) — prev pointer lets us unlink without traversal"""
        if not self.tail:
            return
        if self.head == self.tail:
            self.head = self.tail = None
            return
        self.tail = self.tail.prev
        self.tail.next = None
```

---

## Classic Interview Problems

### 1. Detect a cycle (Floyd's algorithm)

```python
def has_cycle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next          # moves 1 step
        fast = fast.next.next     # moves 2 steps
        if slow == fast:
            return True           # they met → cycle exists
    return False
```

```
Normal list:  slow and fast never meet → fast hits NULL
Cyclic list:  fast laps slow → they meet somewhere in the cycle
```

### 2. Find the middle node

```python
def find_middle(head):
    slow = fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow    # when fast reaches end, slow is at middle
```

### 3. Merge two sorted linked lists

```python
def merge_sorted(l1, l2):
    dummy = Node(0)
    curr = dummy
    while l1 and l2:
        if l1.data <= l2.data:
            curr.next, l1 = l1, l1.next
        else:
            curr.next, l2 = l2, l2.next
        curr = curr.next
    curr.next = l1 or l2
    return dummy.next
```

### 4. Find nth node from end

```python
def nth_from_end(head, n):
    fast = slow = head
    for _ in range(n):           # advance fast by n steps
        fast = fast.next
    while fast:                  # move both until fast hits NULL
        slow = slow.next
        fast = fast.next
    return slow                  # slow is now n from end
```

---

## Linked List vs Array

| Feature                  | Array              | Linked List           |
|--------------------------|--------------------|-----------------------|
| Memory layout            | Contiguous         | Scattered (pointers)  |
| Access by index          | O(1) ✅            | O(n) ❌               |
| Insert / delete at head  | O(n) ❌            | O(1) ✅               |
| Insert / delete at tail  | O(1) amort ✅      | O(1) w/ tail ptr ✅   |
| Memory overhead          | Low (data only)    | Extra pointer per node|
| Cache performance        | Excellent ✅       | Poor (pointer chasing)|
| Dynamic size             | ❌ (fixed or copy)  | ✅                   |
| Binary search            | ✅ O(log n)        | ❌ O(n)               |

---

## Real-World Uses

```
1. Implementing stacks and queues   — O(1) push/pop at head or tail
2. Browser history                  — doubly linked: back and forward navigation
3. Music playlists                  — circular linked list for looping
4. LRU Cache                        — doubly linked list + hash map
5. Memory allocators                — free block lists in heap memory managers
6. Undo/redo systems                — each action is a node; walk backward/forward
7. Polynomial arithmetic            — each term is a node (coefficient + exponent)
8. OS process scheduling            — circular linked list for round-robin scheduling
```

---

## Pros and Cons

### ✅ Advantages
- O(1) insert and delete at head (and tail with a tail pointer)
- Dynamic size — grows and shrinks at runtime with no copying
- No wasted pre-allocated memory
- Efficient for frequent insertions/deletions in the middle (given a pointer)

### ❌ Disadvantages
- O(n) access by index — no random access
- Extra memory per node for the pointer(s)
- Poor cache performance — nodes scattered in memory
- No binary search — must scan sequentially
- Easy to corrupt — a misplaced pointer breaks the entire list

---

## When to Use It

```
Use a Linked List when:
  ✓ Frequent insertions/deletions at head or tail
  ✓ Size changes often and is unpredictable
  ✓ Implementing stacks, queues, deques
  ✓ You have a pointer to the node you want to delete (O(1))

Use an Array when:
  ✗ You need fast index-based access
  ✗ You need binary search
  ✗ Cache performance matters
  ✗ Memory overhead of pointers is a concern
```

---
<p align="center">
  <img src="./assets/linked-list.png" alt="linked list" width="800">
</p>


## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  LINKED LIST — AT A GLANCE                                        │
│                                                                   │
│  Structure:  Nodes connected by pointers (next / prev)           │
│  Types:      Singly, Doubly, Circular                            │
│                                                                   │
│  Insert/Delete at head →  O(1)                                   │
│  Insert/Delete at tail →  O(1) with tail pointer                 │
│  Access by index       →  O(n)  ← main weakness vs array        │
│  Search                →  O(n)                                   │
│                                                                   │
│  Key techniques: Floyd's cycle detection, slow/fast pointers,    │
│                  dummy head node, tail pointer optimisation       │
│                                                                   │
│  Used in: LRU cache, browser history, OS schedulers, queues      │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 11 of daily algorithm commits — next up: Stack (Data Structure)*
