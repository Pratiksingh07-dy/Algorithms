#  Binary Search Tree

> *"A sorted array you can grow. A linked list you can search in O(log n). A BST gives you both."*


---

## What is a Binary Search Tree?

A Binary Search Tree (BST) is a **node-based binary tree** where every node satisfies the **BST property**:

```
For every node N:
  All values in N's LEFT  subtree  <  N.value
  All values in N's RIGHT subtree  >  N.value
```

This ordering is maintained at every node — not just the root. It's what allows **O(log n) search, insert, and delete** on a balanced BST, compared to O(n) for a plain linked list.

---

## Anatomy of a BST


```
                    50              ← Root
                  /    \
                30       70        ← Internal nodes
               /  \     /  \
             20   40   60   80     ← Internal nodes
            /       \
           10       45             ← Leaf nodes

BST property verified:
  Node 50: left subtree max = 45 < 50 ✓, right subtree min = 60 > 50 ✓
  Node 30: left subtree max = 20 < 30 ✓, right subtree max = 45 < 50 ✓
  Node 70: left subtree min = 60 > 50 ✓, right subtree min = 80 > 70 ✓
```

Key terms:
- **Root** — the topmost node (no parent)
- **Leaf** — a node with no children
- **Height** — longest path from root to a leaf
- **Depth** — distance from the root to a given node
- **Balanced BST** — height ≈ log n (e.g. AVL tree, Red-Black tree)

---

## Node Structure

```python
class Node:
    def __init__(self, val):
        self.val   = val
        self.left  = None
        self.right = None
```

---

## Full BST Implementation

```python
class BST:
    def __init__(self):
        self.root = None

    # ── Search ─────────────────────────────────────────────────────

    def search(self, val):
        """O(h) — h is height; O(log n) balanced, O(n) worst"""
        return self._search(self.root, val)

    def _search(self, node, val):
        if node is None:
            return False
        if val == node.val:
            return True
        elif val < node.val:
            return self._search(node.left, val)
        else:
            return self._search(node.right, val)

    # ── Insert ─────────────────────────────────────────────────────

    def insert(self, val):
        """O(h) — recurse to the correct leaf position"""
        self.root = self._insert(self.root, val)

    def _insert(self, node, val):
        if node is None:
            return Node(val)           # found the insertion point
        if val < node.val:
            node.left  = self._insert(node.left, val)
        elif val > node.val:
            node.right = self._insert(node.right, val)
        # equal: BSTs typically ignore duplicates
        return node

    # ── Delete ─────────────────────────────────────────────────────

    def delete(self, val):
        """O(h) — three cases: leaf, one child, two children"""
        self.root = self._delete(self.root, val)

    def _delete(self, node, val):
        if node is None:
            return None

        if val < node.val:
            node.left  = self._delete(node.left, val)
        elif val > node.val:
            node.right = self._delete(node.right, val)
        else:
            # Case 1: Leaf node — just remove it
            if not node.left and not node.right:
                return None
            # Case 2: One child — replace with the child
            if not node.left:
                return node.right
            if not node.right:
                return node.left
            # Case 3: Two children — replace with in-order successor
            successor      = self._min_node(node.right)
            node.val       = successor.val
            node.right     = self._delete(node.right, successor.val)

        return node

    def _min_node(self, node):
        """Find the leftmost (smallest) node"""
        while node.left:
            node = node.left
        return node

    # ── Min / Max ──────────────────────────────────────────────────

    def minimum(self):
        """O(h) — leftmost node"""
        if not self.root:
            return None
        return self._min_node(self.root).val

    def maximum(self):
        """O(h) — rightmost node"""
        node = self.root
        while node.right:
            node = node.right
        return node.val

    # ── Height ─────────────────────────────────────────────────────

    def height(self):
        return self._height(self.root)

    def _height(self, node):
        if node is None:
            return -1
        return 1 + max(self._height(node.left), self._height(node.right))
```

---

## The Three Delete Cases


```
Case 1 — Delete a LEAF (node 10):
  Before:           After:
      20               20
     /  \             /  \
   10   30          NULL  30
  Simply remove it — no rewiring needed.


Case 2 — Delete node with ONE CHILD (node 30, has right child 40):
  Before:           After:
      50               50
     /                /
   30               40
     \
      40
  Replace the node with its only child.


Case 3 — Delete node with TWO CHILDREN (node 50 with children 30 and 70):
  Before:           After:
      50               60       ← in-order successor of 50
     /  \             /  \
   30   70          30   70
        /                  \
       60                  80
       (60 is leftmost node of right subtree = in-order successor)

  Steps:
    1. Find in-order successor: leftmost of right subtree = 60
    2. Copy successor's value into the node: node.val = 60
    3. Delete successor from right subtree
```

---

## Tree Traversals


All traversals are **O(n)** — every node is visited exactly once.

```
Tree:
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```

### In-order (Left → Root → Right)
Visits nodes in **sorted ascending order** — the most useful traversal for a BST.

```python
def inorder(node, result=[]):
    if node:
        inorder(node.left, result)
        result.append(node.val)
        inorder(node.right, result)
    return result

# Result: [1, 2, 3, 4, 5, 6, 7]  ← sorted!
```

### Pre-order (Root → Left → Right)
Used to **copy or serialise** a tree — root always comes first.

```python
def preorder(node, result=[]):
    if node:
        result.append(node.val)
        preorder(node.left, result)
        preorder(node.right, result)
    return result

# Result: [4, 2, 1, 3, 6, 5, 7]
```

### Post-order (Left → Right → Root)
Used to **delete a tree** or evaluate expression trees — children processed before parent.

```python
def postorder(node, result=[]):
    if node:
        postorder(node.left, result)
        postorder(node.right, result)
        result.append(node.val)
    return result

# Result: [1, 3, 2, 5, 7, 6, 4]
```

### Level-order (Breadth-First)
Visits nodes **level by level** — requires a queue.

```python
from collections import deque

def levelorder(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        node = queue.popleft()
        result.append(node.val)
        if node.left:  queue.append(node.left)
        if node.right: queue.append(node.right)
    return result

# Result: [4, 2, 6, 1, 3, 5, 7]
```

---

## Complexity


### Time

| Operation | Balanced BST  | Unbalanced (worst) | Sorted insert → |
|-----------|---------------|--------------------|-----------------|
| Search    | **O(log n)**  | O(n)               | O(n)            |
| Insert    | **O(log n)**  | O(n)               | O(n)            |
| Delete    | **O(log n)**  | O(n)               | O(n)            |
| Min / Max | **O(log n)**  | O(n)               | O(n)            |
| In-order  | O(n)          | O(n)               | O(n)            |

### Space

| Metric        | Complexity | Notes                          |
|---------------|-----------|--------------------------------|
| Storage       | **O(n)**  | One node per element           |
| Recursive ops | O(h)      | h = height = O(log n) balanced |

---

## The Degenerate Case

The worst enemy of a BST: **inserting sorted data**.

```
Insert 1, 2, 3, 4, 5 in order:

1                              Looks like a linked list!
 \                             Height = n-1
  2                            All operations = O(n)
   \
    3
     \
      4
       \
        5
```

![Degenerate BST vs Balanced BST](./assets/bst-degenerate-vs-balanced.png)

This is why **self-balancing BSTs** exist:
- **AVL Tree** — strictly balanced (height diff ≤ 1 at every node), faster lookups
- **Red-Black Tree** — loosely balanced, faster insertions (used in C++ `std::map`, Java `TreeMap`)
- **B-Tree** — generalisation for disk storage (used in databases, filesystems)

---

## Classic Interview Problems

### 1. Validate a BST

```python
def is_valid_bst(node, min_val=float('-inf'), max_val=float('inf')):
    if not node:
        return True
    if not (min_val < node.val < max_val):
        return False
    return (is_valid_bst(node.left,  min_val, node.val) and
            is_valid_bst(node.right, node.val, max_val))
```

### 2. Lowest Common Ancestor (LCA)

```python
def lca(root, p, q):
    if p.val < root.val and q.val < root.val:
        return lca(root.left, p, q)   # both in left subtree
    if p.val > root.val and q.val > root.val:
        return lca(root.right, p, q)  # both in right subtree
    return root                        # split point = LCA
```

### 3. Kth Smallest Element

```python
def kth_smallest(root, k):
    stack, node = [], root
    while stack or node:
        while node:
            stack.append(node)
            node = node.left
        node = stack.pop()
        k -= 1
        if k == 0:
            return node.val
        node = node.right
```

### 4. Convert Sorted Array to Balanced BST

```python
def sorted_to_bst(nums):
    if not nums:
        return None
    mid  = len(nums) // 2
    node = Node(nums[mid])
    node.left  = sorted_to_bst(nums[:mid])
    node.right = sorted_to_bst(nums[mid+1:])
    return node
```

---

## BST vs Other Structures

| Feature           | BST (balanced) | Hash Table | Sorted Array |
|-------------------|---------------|------------|--------------|
| Search            | O(log n)      | O(1) avg   | O(log n)     |
| Insert            | O(log n)      | O(1) avg   | O(n)         |
| Delete            | O(log n)      | O(1) avg   | O(n)         |
| Min / Max         | **O(log n)**  | O(n)       | O(1)         |
| Range query       | **O(log n+k)**| O(n)       | O(log n+k)   |
| Sorted iteration  | **O(n)**      | O(n log n) | O(n)         |
| Ordered           | ✅ Yes        | ❌ No      | ✅ Yes       |

> Use a BST when you need **ordered operations** (range queries, sorted iteration, floor/ceiling). Use a Hash Table when you only need **exact key lookup**.

---

## Real-World Uses

```
1. std::map / TreeMap         — C++/Java sorted maps use Red-Black trees (BST variant)
2. Database indices            — B-Trees (BST generalisation) power SQL indexes
3. Filesystems                 — directory entries in NTFS, HFS+ use B-Trees
4. Priority queues             — binary heaps are a specialised BST
5. Auto-complete               — trie (prefix tree) is a BST variant for strings
6. Interval scheduling         — interval trees extend BSTs for range overlap queries
7. Git version graph           — DAG traversal uses tree algorithms
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  BINARY SEARCH TREE — AT A GLANCE                                 │
│                                                                   │
│  Property:  left < node < right (at every node)                   │
│                                                                   │
│  Time  →  O(log n) balanced  |  O(n) degenerate (sorted insert)   │
│  Space →  O(n) nodes  +  O(h) stack for recursive ops             │
│                                                                   │
│  Traversals:                                                      │
│    In-order   → sorted output (most useful)                       │
│    Pre-order  → copy / serialise tree                             │
│    Post-order → delete / evaluate bottom-up                       │
│    Level-order→ BFS, level-by-level                               │
│                                                                   │
│  Worst case: inserting sorted data → linked list → O(n)           │
│  Solution:   use AVL or Red-Black tree for guaranteed O(log n)    │
│                                                                   │
│  Choose BST over Hash Table when: range queries, ordered output   │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 15 of daily algorithm commits — next up: Graph (Data Structure)*
