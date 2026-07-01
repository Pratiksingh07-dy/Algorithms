# 📚 Stack

> *"The last thing you put in is the first thing you get out. Simple rule — enormous power."*

---

## What is a Stack?

A Stack is a **linear data structure** that follows the **LIFO** principle — **Last In, First Out**.

Think of a stack of plates: you add plates to the top and remove them from the top. You can never access a plate in the middle without removing everything above it first.

Three core operations, all O(1):
- **Push** — add an element to the top
- **Pop** — remove the element from the top
- **Peek / Top** — view the top element without removing it

The restriction is the power. By limiting access to one end only, the Stack becomes a precise tool for tracking state, managing function calls, and solving a surprising range of problems elegantly.

---

## Visualising a Stack

```
── push(10) ──     ── push(20) ──     ── push(30) ──     ── pop() ──
                                       ┌──────┐
                    ┌──────┐           │  30  │  ← TOP        ┌──────┐
    ┌──────┐        │  20  │  ← TOP    ├──────┤               │  20  │ ← TOP
    │  10  │ ← TOP  ├──────┤           │  20  │               ├──────┤
    └──────┘        │  10  │           ├──────┤               │  10  │
                    └──────┘           │  10  │               └──────┘
                                       └──────┘
                                                          returns 30
```

---

## Implementation 1 — Using a Python List

Python's built-in list is a perfect stack. `append()` is push, `pop()` is pop — both O(1) amortised.

```python
class Stack:
    def __init__(self):
        self._data = []

    def push(self, item):
        """O(1) amortised — append to end of list"""
        self._data.append(item)

    def pop(self):
        """O(1) — remove from end of list"""
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        return self._data.pop()

    def peek(self):
        """O(1) — view top without removing"""
        if self.is_empty():
            raise IndexError("Peek at empty stack")
        return self._data[-1]

    def is_empty(self):
        return len(self._data) == 0

    def size(self):
        return len(self._data)

    def __repr__(self):
        return f"Stack({self._data}) ← top"


# Usage
s = Stack()
s.push(10)
s.push(20)
s.push(30)
print(s)           # Stack([10, 20, 30]) ← top
print(s.pop())     # 30
print(s.peek())    # 20
print(s.size())    # 2
```

---

## Implementation 2 — Using a Linked List

More memory overhead but truly O(1) push/pop with no reallocation.

```python
class Node:
    def __init__(self, data):
        self.data = data
        self.next = None


class Stack:
    def __init__(self):
        self._top = None
        self._size = 0

    def push(self, item):
        """O(1) — new node becomes the top"""
        node = Node(item)
        node.next = self._top
        self._top = node
        self._size += 1

    def pop(self):
        """O(1) — unlink the top node"""
        if self.is_empty():
            raise IndexError("Pop from empty stack")
        val = self._top.data
        self._top = self._top.next
        self._size -= 1
        return val

    def peek(self):
        """O(1) — read top node's data"""
        if self.is_empty():
            raise IndexError("Peek at empty stack")
        return self._top.data

    def is_empty(self):
        return self._top is None

    def size(self):
        return self._size
```

---

## Operations Complexity

| Operation  | List-based | Linked List-based |
|------------|-----------|-------------------|
| Push       | O(1) amort | O(1)              |
| Pop        | O(1)       | O(1)              |
| Peek       | O(1)       | O(1)              |
| Is Empty   | O(1)       | O(1)              |
| Size       | O(1)       | O(1)              |
| Search     | O(n)       | O(n)              |
| Space      | O(n)       | O(n) + pointer overhead |

---

## How the Call Stack Works

Every function call in every programming language uses a stack under the hood.

```python
def c():
    return 1 + 1

def b():
    return c()

def a():
    return b()

a()
```

```
Call Stack during execution:

a() is called:
┌──────────────────┐
│   a() frame      │  ← top (currently executing)
└──────────────────┘

a() calls b():
┌──────────────────┐
│   b() frame      │  ← top
├──────────────────┤
│   a() frame      │
└──────────────────┘

b() calls c():
┌──────────────────┐
│   c() frame      │  ← top
├──────────────────┤
│   b() frame      │
├──────────────────┤
│   a() frame      │
└──────────────────┘

c() returns → popped
b() returns → popped
a() returns → popped
Stack empty.
```

Recursion too deep? Stack frames accumulate until memory runs out → **Stack Overflow**.

---

## Classic Problems

### 1. Valid Parentheses

Check if brackets are correctly matched and nested.

```python
def is_valid(s):
    stack = []
    pairs = {')': '(', ']': '[', '}': '{'}

    for ch in s:
        if ch in '([{':
            stack.append(ch)
        elif ch in ')]}':
            if not stack or stack[-1] != pairs[ch]:
                return False
            stack.pop()

    return len(stack) == 0


print(is_valid("({[]})"))   # True
print(is_valid("({[})"))    # False
```

```
Input: "({[]})"
Push:   (  →  stack: ['(']
Push:   {  →  stack: ['(', '{']
Push:   [  →  stack: ['(', '{', '[']
Match:  ]  →  top is '[' ✓  →  stack: ['(', '{']
Match:  }  →  top is '{' ✓  →  stack: ['(']
Match:  )  →  top is '(' ✓  →  stack: []
Empty stack → valid ✅
```

### 2. Evaluate Reverse Polish Notation (RPN)

```python
def eval_rpn(tokens):
    stack = []
    ops = {
        '+': lambda a, b: a + b,
        '-': lambda a, b: a - b,
        '*': lambda a, b: a * b,
        '/': lambda a, b: int(a / b),
    }
    for token in tokens:
        if token in ops:
            b, a = stack.pop(), stack.pop()
            stack.append(ops[token](a, b))
        else:
            stack.append(int(token))
    return stack[0]


print(eval_rpn(["2","1","+","3","*"]))   # (2+1)*3 = 9
print(eval_rpn(["4","13","5","/","+"]))  # 4+(13/5) = 6
```

### 3. Next Greater Element

For each element, find the next element in the array that is greater.

```python
def next_greater(arr):
    n = len(arr)
    result = [-1] * n
    stack = []                      # stores indices

    for i in range(n):
        while stack and arr[stack[-1]] < arr[i]:
            idx = stack.pop()
            result[idx] = arr[i]    # arr[i] is next greater for arr[idx]
        stack.append(i)

    return result


print(next_greater([4, 5, 2, 10, 8]))
# → [5, 10, 10, -1, -1]
```

### 4. Min Stack (get minimum in O(1))

```python
class MinStack:
    def __init__(self):
        self.stack = []
        self.min_stack = []          # tracks current minimum at each level

    def push(self, val):
        self.stack.append(val)
        min_val = min(val, self.min_stack[-1] if self.min_stack else val)
        self.min_stack.append(min_val)

    def pop(self):
        self.min_stack.pop()
        return self.stack.pop()

    def get_min(self):
        return self.min_stack[-1]    # O(1) — always current minimum


s = MinStack()
s.push(5); s.push(3); s.push(7); s.push(2)
print(s.get_min())   # 2
s.pop()
print(s.get_min())   # 3
```

---

## The Monotonic Stack Pattern

A **monotonic stack** maintains elements in strictly increasing or decreasing order. It's a powerful pattern for problems involving "next greater", "previous smaller", "largest rectangle", and "trapping rain water".

```
Monotonic Increasing Stack:
  Elements are kept in increasing order from bottom to top.
  When a new element arrives, pop everything larger than it first.

  arr  = [2, 1, 5, 6, 2, 3]
  i=0:  push 2   →  stack: [2]
  i=1:  2 > 1, pop 2. push 1  →  stack: [1]
  i=2:  1 < 5, push 5  →  stack: [1, 5]
  i=3:  5 < 6, push 6  →  stack: [1, 5, 6]
  i=4:  6 > 2, pop 6. 5 > 2, pop 5. 1 < 2, push 2  →  stack: [1, 2]
  i=5:  2 < 3, push 3  →  stack: [1, 2, 3]
```

This pattern solves in O(n) problems that feel like they need O(n²).

---

## Stack vs Queue

| Feature          | Stack (LIFO)          | Queue (FIFO)              |
|------------------|-----------------------|---------------------------|
| Order            | Last In, First Out    | First In, First Out       |
| Add element      | Push to **top**       | Enqueue to **rear**       |
| Remove element   | Pop from **top**      | Dequeue from **front**    |
| Real world       | Undo, call stack      | Printer queue, BFS        |
| Implementation   | List / Linked List    | Deque / Linked List       |

---

## Real-World Uses

```
1. Function call stack      — every program uses this; recursion depends on it
2. Undo / redo              — text editors push actions, pop on Ctrl+Z
3. Browser back button      — visited pages pushed; back = pop
4. Expression evaluation    — compilers evaluate arithmetic via stacks
5. Syntax parsing           — bracket matching, HTML tag validation
6. DFS traversal            — iterative depth-first search uses an explicit stack
7. Backtracking             — maze solving, N-Queens: push state, pop on dead end
8. Memory management        — stack memory for local variables in programs
```

---

## Pros and Cons

### ✅ Advantages
- All core operations O(1)
- Simple, predictable behaviour — LIFO is easy to reason about
- Fundamental building block — used in dozens of algorithms
- Low memory overhead (list-based implementation)

### ❌ Disadvantages
- Only access to the top element — no random access
- O(n) search — must pop everything to find a buried element
- Fixed access pattern — not suitable when you need to access arbitrary elements
- Stack overflow risk with deep recursion

---

## When to Use It

```
Use a Stack when:
  ✓ You need to reverse a sequence
  ✓ You need to track the "most recent" thing (last opened, last called)
  ✓ Matching pairs: parentheses, tags, brackets
  ✓ Implementing DFS or backtracking iteratively
  ✓ Expression evaluation or syntax parsing
  ✓ Undo/redo functionality

Don't use a Stack when:
  ✗ You need FIFO ordering → use a Queue
  ✗ You need access to arbitrary elements → use an array or deque
  ✗ You need to search efficiently → use a hash table or BST
```

---

<p align="center">
  <img src="./assets/pushstack.jpg" alt="stack" width="800">
</p>
* credits to GeeksforGeeks

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  STACK — AT A GLANCE                                              │
│                                                                   │
│  Principle:  LIFO — Last In, First Out                            │
│  Core ops:   Push O(1)  |  Pop O(1)  |  Peek O(1)                 │
│                                                                   │
│  Implement with:  Python list (simple) or Linked List (pure O(1)) │
│                                                                   │
│  Key patterns:                                                    │
│    • Valid parentheses  — push open, match on close               │
│    • Min/Max stack      — auxiliary stack tracks extremes         │
│    • Monotonic stack    — O(n) solution for next greater element  │
│    • Iterative DFS      — explicit stack replaces recursion       │
│                                                                   │
│  Used in: call stack, undo systems, parsers, DFS, backtracking    │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 12 of daily algorithm commits — next up: Queue (Data Structure)*
