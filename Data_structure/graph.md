# 🕸️ Graph

> *"Trees are graphs that grew up in a structured family. Graphs are the wild, connected world everything else lives in."*

![Graph Overview](./assets/graph-overview.png)

---

## What is a Graph?

A **Graph** is a non-linear data structure consisting of:
- **Vertices (V)** — the nodes or entities
- **Edges (E)** — the connections or relationships between vertices

Formally: G = (V, E)

Almost every real-world relationship can be modelled as a graph: road networks, social connections, web pages, dependencies, circuits, and more. Trees, linked lists, and even arrays are just special cases of graphs.

---

## Types of Graphs

![Graph Types Diagram](./assets/graph-types.png)

### By edge direction
```
Undirected Graph:            Directed Graph (Digraph):
  A ── B ── C                  A ──► B ──► C
  │         │                  │           │
  D ── E ── F                  ▼           ▼
                               D ◄── E ◄── F

Edge has no direction.       Edge has a direction (one-way).
"A is connected to B"        "A points to B" but not vice versa.
Examples: road networks,     Examples: web links, Twitter follows,
  social friendships           dependency graphs, task scheduling
```

### By edge weight
```
Unweighted:       Weighted:
  A ── B            A ──(4)── B
  │   │             │         │
  C ── D           (2)       (7)
                    │         │
                    C ──(1)── D

Edges just exist.  Edges have a cost (distance, time, capacity).
Examples: social   Examples: maps, network routing, flight paths
  networks
```

### Other types
```
Cyclic Graph      — contains at least one cycle (path that loops back)
Acyclic Graph     — no cycles
DAG               — Directed Acyclic Graph (dependency trees, Git commits)
Connected Graph   — every vertex is reachable from every other
Sparse Graph      — few edges: E ≈ V
Dense Graph       — many edges: E ≈ V²
```

---

## Graph Representations

![Graph Representations Diagram](./assets/graph-representations.png)

### Representation 1 — Adjacency List

Each vertex stores a list of its neighbours. Most common for **sparse graphs**.

```
Graph:   1 ── 2 ── 3
         │         │
         4 ──────── 5

Adjacency List:
  1: [2, 4]
  2: [1, 3]
  3: [2, 5]
  4: [1, 5]
  5: [3, 4]
```

```python
# As a dictionary of lists
graph = {
    1: [2, 4],
    2: [1, 3],
    3: [2, 5],
    4: [1, 5],
    5: [3, 4]
}

# As a defaultdict (handles new vertices gracefully)
from collections import defaultdict
graph = defaultdict(list)
graph[1].extend([2, 4])
graph[2].extend([1, 3])
```

**Space:** O(V + E) — ideal for sparse graphs

### Representation 2 — Adjacency Matrix

A V×V 2D array where `matrix[i][j] = 1` if edge exists. Best for **dense graphs** or when edge lookup must be O(1).

```
Graph (same as above):

     1  2  3  4  5
  1 [0, 1, 0, 1, 0]
  2 [1, 0, 1, 0, 0]
  3 [0, 1, 0, 0, 1]
  4 [1, 0, 0, 0, 1]
  5 [0, 0, 1, 1, 0]

matrix[1][2] = 1  →  edge between 1 and 2 exists
matrix[1][3] = 0  →  no direct edge between 1 and 3
```

```python
V = 5
matrix = [[0] * (V+1) for _ in range(V+1)]
matrix[1][2] = matrix[2][1] = 1   # undirected: set both
matrix[1][4] = matrix[4][1] = 1
# ... etc.

# Edge lookup: O(1)
if matrix[u][v]:
    print(f"Edge exists between {u} and {v}")
```

**Space:** O(V²) — wasteful for sparse graphs

### Representation 3 — Edge List

A simple list of all edges. Used in some algorithms (Kruskal's MST).

```python
edges = [(1,2), (1,4), (2,3), (3,5), (4,5)]

# Weighted edge list:
weighted_edges = [(1,2,4), (1,4,2), (2,3,7), (3,5,1), (4,5,3)]
#                 (u, v, weight)
```

**Space:** O(E)

---

## Representations Comparison

| Feature            | Adjacency List | Adjacency Matrix | Edge List |
|--------------------|---------------|-----------------|-----------|
| Space              | O(V + E)      | O(V²)           | O(E)      |
| Add edge           | O(1)          | O(1)            | O(1)      |
| Remove edge        | O(E)          | O(1)            | O(E)      |
| Check edge (u,v)   | O(degree(u))  | **O(1)**        | O(E)      |
| Get all neighbours | O(degree(u))  | O(V)            | O(E)      |
| Best for           | Sparse graphs | Dense graphs    | MST algos |

---

## Graph Traversals

![BFS vs DFS Diagram](./assets/graph-bfs-vs-dfs.png)

### Breadth-First Search (BFS)

Explore **level by level** — visit all neighbours before going deeper. Uses a **queue**. Finds the **shortest path** in unweighted graphs.

```python
from collections import deque

def bfs(graph, start):
    visited = set([start])
    queue   = deque([start])
    order   = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbour in graph[node]:
            if neighbour not in visited:
                visited.add(neighbour)
                queue.append(neighbour)
    return order

# BFS from node 1: [1, 2, 4, 3, 5]
#   Level 0: 1
#   Level 1: 2, 4
#   Level 2: 3, 5
```

### Depth-First Search (DFS)

Explore **as deep as possible** before backtracking. Uses a **stack** (or recursion). Used for cycle detection, topological sort, connected components.

```python
def dfs(graph, start, visited=None):
    if visited is None:
        visited = set()
    visited.add(start)
    order = [start]
    for neighbour in graph[start]:
        if neighbour not in visited:
            order.extend(dfs(graph, neighbour, visited))
    return order

# DFS from node 1: [1, 2, 3, 5, 4]  (depends on adjacency list order)
```

```
BFS traversal (level-order):       DFS traversal (depth-first):

        1                                  1
       / \                                / \
      2   4                              2   4
     /     \                            /     \
    3        5                         3        5

Visit: 1→2→4→3→5                  Visit: 1→2→3→5→4
       (width first)                      (depth first)
```

---

## Classic Graph Algorithms

### Cycle Detection (Undirected Graph)

```python
def has_cycle(graph, vertices):
    visited = set()

    def dfs(node, parent):
        visited.add(node)
        for neighbour in graph[node]:
            if neighbour not in visited:
                if dfs(neighbour, node):
                    return True
            elif neighbour != parent:
                return True            # back edge found → cycle
        return False

    for v in vertices:
        if v not in visited:
            if dfs(v, -1):
                return True
    return False
```

### Topological Sort (DAG only)

Order tasks so every dependency comes before the task that needs it.

```python
def topological_sort(graph, vertices):
    visited = set()
    stack   = []

    def dfs(node):
        visited.add(node)
        for neighbour in graph[node]:
            if neighbour not in visited:
                dfs(neighbour)
        stack.append(node)   # push AFTER all descendants are processed

    for v in vertices:
        if v not in visited:
            dfs(v)

    return stack[::-1]   # reverse gives topological order

# Example: build order for tasks with dependencies
graph = {'A': ['C'], 'B': ['C', 'D'], 'C': ['E'], 'D': ['F'], 'E': [], 'F': []}
print(topological_sort(graph, graph.keys()))
# ['A', 'B', 'D', 'F', 'C', 'E'] or similar valid order
```

### Shortest Path — BFS (Unweighted)

```python
def shortest_path(graph, start, end):
    if start == end:
        return [start]
    visited = {start}
    queue   = deque([[start]])

    while queue:
        path = queue.popleft()
        node = path[-1]
        for neighbour in graph[node]:
            if neighbour == end:
                return path + [neighbour]   # found!
            if neighbour not in visited:
                visited.add(neighbour)
                queue.append(path + [neighbour])
    return None   # no path exists
```

### Count Connected Components

```python
def count_components(graph, vertices):
    visited = set()
    count   = 0

    def dfs(node):
        visited.add(node)
        for neighbour in graph[node]:
            if neighbour not in visited:
                dfs(neighbour)

    for v in vertices:
        if v not in visited:
            dfs(v)
            count += 1   # each unvisited vertex starts a new component
    return count
```

---

## BFS vs DFS at a Glance

![BFS vs DFS Comparison](./assets/bfs-dfs-comparison.png)

| Feature                 | BFS                        | DFS                         |
|-------------------------|----------------------------|-----------------------------|
| Data structure          | Queue                      | Stack (or recursion)        |
| Traversal order         | Level by level             | Deep branch first           |
| Shortest path (unweighted)| ✅ Guaranteed            | ❌ Not guaranteed           |
| Memory usage            | O(V) — holds entire level  | O(h) — h = max depth       |
| Cycle detection         | ✅ Yes                     | ✅ Yes                      |
| Topological sort        | ❌ (use Kahn's algorithm)  | ✅ Yes                      |
| Connected components    | ✅ Yes                     | ✅ Yes                      |
| Best for                | Shortest paths, level-order| Paths, cycles, topo sort    |

---

## Complexity

| Operation / Algorithm  | Adjacency List | Adjacency Matrix |
|------------------------|---------------|-----------------|
| BFS / DFS              | **O(V + E)**  | O(V²)           |
| Dijkstra (with heap)   | O((V+E) log V)| O(V² log V)     |
| Detect cycle           | O(V + E)      | O(V²)           |
| Topological sort       | O(V + E)      | O(V²)           |
| Find all paths         | O(V + E)      | O(V²)           |

---

## Real-World Uses

```
1. Social networks        — users are vertices, friendships are edges
2. GPS / Maps             — intersections are vertices, roads are weighted edges
3. Web crawling           — pages are vertices, hyperlinks are directed edges
4. Dependency resolution  — package managers (npm, pip) use DAG + topological sort
5. Network routing        — routers use shortest-path algorithms (OSPF uses Dijkstra)
6. Recommendation engines — bipartite graphs connect users to items
7. Compilers              — control flow graphs model program execution paths
8. Git history            — commits form a DAG (Directed Acyclic Graph)
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  GRAPH — AT A GLANCE                                              │
│                                                                   │
│  G = (V, E): vertices + edges                                    │
│  Types: directed/undirected, weighted/unweighted, cyclic/acyclic  │
│                                                                   │
│  Representations:                                                 │
│    Adjacency List  → O(V+E) space, best for sparse               │
│    Adjacency Matrix→ O(V²) space, O(1) edge lookup               │
│                                                                   │
│  Traversals:                                                      │
│    BFS → queue, level-order, shortest path (unweighted)          │
│    DFS → stack/recursion, deep-first, cycle detect, topo sort    │
│                                                                   │
│  Both BFS and DFS: O(V + E) time with adjacency list            │
│                                                                   │
│  Real world: maps, social networks, dependency trees, compilers  │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 16 of daily algorithm commits — next up: Heap (Data Structure)*
