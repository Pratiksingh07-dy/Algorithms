# 🔗 Union-Find (Disjoint Set Union)

> *"Two operations. Near-constant time. The most elegant data structure for connectivity problems."*

---

## What is Union-Find?

Union-Find (also called **Disjoint Set Union / DSU**) is a data structure that tracks a collection of elements partitioned into **disjoint (non-overlapping) sets**. It supports two core operations extremely efficiently:

- **find(x)** — which set does element x belong to? Returns the **root** (representative) of x's set
- **union(x, y)** — merge the sets containing x and y into one

It answers the question: **"Are these two elements in the same connected component?"** in nearly O(1) time.

---

## Visualisation

<p align="center">
  <img src="./assets/union-find-overview.png" alt="Union-Find — Initial State, After Unions, Path Compression" width="900">
</p>

---

## The Core Idea

Every element starts as its own parent (its own set). A `union` operation links two roots together. A `find` operation traverses parent pointers up to the root.

```
Initial:  parent = [0, 1, 2, 3, 4, 5, 6]
           Each element is its own root

After union(0,1):  parent[1] = 0    → 0 and 1 in same set
After union(1,2):  parent[2] = 0    → 0, 1, 2 in same set
After union(3,4):  parent[4] = 3    → 3 and 4 in same set

find(2):  2 → parent[2]=0 → parent[0]=0 → root = 0
find(4):  4 → parent[4]=3 → parent[3]=3 → root = 3

connected(2, 4)?  find(2)=0 ≠ find(4)=3  → NO
connected(0, 2)?  find(0)=0 = find(2)=0  → YES
```

---

## Implementation — Basic

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))   # parent[i] = i initially

    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False            # already in same set
        self.parent[ry] = rx        # merge: ry's root points to rx
        return True

    def connected(self, x, y):
        return self.find(x) == self.find(y)
```

This works but can degrade to **O(n)** per operation if we keep building a chain (like a linked list). Two optimisations fix this completely.

---

## Optimisation 1 — Union by Rank

Always attach the **smaller tree** under the **larger tree's root**. This keeps trees shallow.

```python
class UnionFind:
    def __init__(self, n):
        self.parent = list(range(n))
        self.rank   = [0] * n       # approximation of tree height

    def find(self, x):
        while self.parent[x] != x:
            x = self.parent[x]
        return x

    def union(self, x, y):
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False

        # Attach smaller rank tree under larger rank tree
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx         # swap so rx always has >= rank
        self.parent[ry] = rx        # ry's root points to rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1      # same rank: one level deeper

        return True
```

```
Without rank (worst case — insert 1,2,3,4,5 in order):
  1 → 2 → 3 → 4 → 5      height = n-1

With union by rank:
        1
      / | \
     2  3  4               height = O(log n)
        |
        5
```

---

## Optimisation 2 — Path Compression

During `find`, make every node on the path point **directly to the root**. Future finds on these nodes are O(1).

```python
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])  # compress path
    return self.parent[x]
```

```
Before find(5):               After find(5) with compression:
  root(1)                           root(1)
    │                             / | | \ \
    2                            2  3  4  5  (all direct)
    │
    3
    │
    4
    │
    5

Next time: find(5) = 1 directly — O(1)!
```

---

## Full Optimised Implementation

```python
class UnionFind:
    def __init__(self, n):
        self.parent     = list(range(n))
        self.rank       = [0] * n
        self.components = n          # track number of disjoint sets

    def find(self, x):
        """O(α(n)) — path compression"""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])   # compress
        return self.parent[x]

    def union(self, x, y):
        """O(α(n)) — union by rank"""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False            # already connected

        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        self.components -= 1        # one fewer component
        return True

    def connected(self, x, y):
        """O(α(n)) — are x and y in the same set?"""
        return self.find(x) == self.find(y)

    def count(self):
        """O(1) — number of disjoint sets"""
        return self.components


# Usage
uf = UnionFind(7)
uf.union(0, 1)
uf.union(1, 2)
uf.union(3, 4)
uf.union(5, 6)

print(uf.connected(0, 2))    # True   (0→1→2 merged)
print(uf.connected(0, 3))    # False  (different components)
print(uf.count())             # 3      ({0,1,2}, {3,4}, {5,6})

uf.union(2, 3)
print(uf.connected(0, 4))    # True   (now all merged)
print(uf.count())             # 2
```

---

## Complexity

### Time

| Operation       | Naive  | With Rank | With Rank + Compression |
|-----------------|--------|-----------|-------------------------|
| find(x)         | O(n)   | O(log n)  | **O(α(n)) ≈ O(1)**      |
| union(x, y)     | O(n)   | O(log n)  | **O(α(n)) ≈ O(1)**      |
| connected(x, y) | O(n)   | O(log n)  | **O(α(n)) ≈ O(1)**      |
| initialise      | O(n)   | O(n)      | O(n)                    |

> **α(n)** is the **inverse Ackermann function** — a function that grows so incredibly slowly that for any practical input size (even n = 10^80), α(n) ≤ 4. It is effectively constant.

### Space

| Metric  | Complexity | Notes                        |
|---------|-----------|-------------------------------|
| Storage | **O(n)**  | parent[ ] and rank[ ] arrays  |

---

## Properties at a Glance

| Property              | Value   | Notes                                       |
|-----------------------|---------|---------------------------------------------|
| Dynamic connectivity  | ✅ Yes  | Add edges incrementally                     |
| Delete edges          | ❌ No   | Standard DSU doesn't support edge deletion  |
| Find connected components | ✅ Yes | count() gives number of components       |
| Path queries          | ❌ No   | Only answers "same set?" not "what's the path?" |
| Weighted variant      | ✅ Yes  | Store edge weights in DSU for advanced uses |

---

## Classic Problems

### 1. Number of Connected Components in Graph

```python
def count_components(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        uf.union(u, v)
    return uf.count()

print(count_components(5, [[0,1],[1,2],[3,4]]))   # 2
```

### 2. Detect Cycle in Undirected Graph

```python
def has_cycle(n, edges):
    uf = UnionFind(n)
    for u, v in edges:
        if not uf.union(u, v):   # union returns False if already connected
            return True           # adding this edge creates a cycle
    return False

print(has_cycle(4, [[0,1],[1,2],[2,3],[3,0]]))   # True
print(has_cycle(4, [[0,1],[1,2],[2,3]]))          # False
```

### 3. Kruskal's Minimum Spanning Tree

```python
def kruskal_mst(n, edges):
    """edges = [(weight, u, v)] — find MST using Union-Find"""
    edges.sort()                  # process lightest edges first
    uf  = UnionFind(n)
    mst = []
    total_weight = 0

    for weight, u, v in edges:
        if uf.union(u, v):        # only add if it doesn't form a cycle
            mst.append((u, v, weight))
            total_weight += weight
            if len(mst) == n - 1:
                break             # MST complete: n-1 edges

    return mst, total_weight

edges = [(1,0,1),(4,0,2),(3,1,2),(2,1,3),(5,2,3)]
mst, cost = kruskal_mst(4, edges)
print(mst)    # [(0,1,1),(1,3,2),(1,2,3)]
print(cost)   # 6
```

### 4. Accounts Merge (LeetCode 721)

```python
def accounts_merge(accounts):
    from collections import defaultdict
    uf      = UnionFind(len(accounts))
    email_to_id = {}

    for i, account in enumerate(accounts):
        for email in account[1:]:
            if email in email_to_id:
                uf.union(i, email_to_id[email])   # merge accounts with shared email
            else:
                email_to_id[email] = i

    groups = defaultdict(list)
    for email, i in email_to_id.items():
        groups[uf.find(i)].append(email)

    return [[accounts[root][0]] + sorted(emails)
            for root, emails in groups.items()]
```

### 5. Redundant Connection

```python
def find_redundant_connection(edges):
    n  = len(edges)
    uf = UnionFind(n + 1)
    for u, v in edges:
        if not uf.union(u, v):
            return [u, v]         # this edge creates the first cycle
    return []
```

---

## Union-Find vs BFS/DFS for Connectivity

| Feature                    | Union-Find          | BFS / DFS           |
|----------------------------|---------------------|---------------------|
| Dynamic edge additions     | ✅ O(α(n)) per edge | ❌ Re-run O(V+E)    |
| Static graph connectivity  | ✅ O(n α(n)) total  | ✅ O(V+E) one-time  |
| Find actual path           | ❌ No               | ✅ Yes              |
| Detect cycle               | ✅ Yes              | ✅ Yes              |
| Delete edges               | ❌ No               | ✅ Yes (re-run)     |
| Space                      | O(n)                | O(V+E)              |
| Best use                   | Incremental unions  | Full graph traversal|

> **Rule of thumb:** If edges are added one at a time and you only need connectivity queries → Union-Find. If you need paths or work on a fully known graph → BFS/DFS.

---

## Real-World Uses

```
1. Kruskal's MST           — build minimum spanning tree edge by edge
2. Network connectivity    — detect if two computers are in the same network
3. Percolation theory      — physics simulation (does water flow through?)
4. Image segmentation      — group adjacent pixels with similar colours
5. Social network clusters — detect friend circles / communities
6. Compiler optimisation   — equivalence classes in register allocation
7. Online connectivity     — detect connected components as edges arrive
8. Least common ancestor   — offline LCA queries using DSU (Tarjan's LCA)
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  UNION-FIND — AT A GLANCE                                         │
│                                                                   │
│  Two operations:  find(x) → root   |   union(x,y) → merge sets  │
│                                                                   │
│  Naive:      O(n) per op                                         │
│  +Rank:      O(log n) per op                                     │
│  +Compress:  O(α(n)) ≈ O(1) per op  ← use this always          │
│                                                                   │
│  α(n) = inverse Ackermann — practically constant for all n       │
│                                                                   │
│  Best for:  dynamic connectivity, cycle detection, Kruskal's MST │
│  Not for:   edge deletion, finding actual paths                  │
│                                                                   │
│  Used in: Kruskal MST, percolation, image segmentation, LCA      │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 21 of daily algorithm commits — next up: Sliding Window Pattern*
