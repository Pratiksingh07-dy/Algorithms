# 🗺️ Dijkstra's Algorithm

> *"Find the shortest path from one node to all others — greedily, one closest node at a time."*

---

## What is Dijkstra's Algorithm?

Dijkstra's Algorithm finds the **shortest path from a source vertex to all other vertices** in a weighted graph with **non-negative edge weights**.

It's a **greedy algorithm** — at each step it picks the unvisited node with the smallest known distance, processes its neighbours, and locks in that node's shortest distance forever. Once locked, a node's distance never changes.

It uses a **min-heap (priority queue)** to efficiently find the closest unvisited node at each step.

> **Critical constraint:** Edge weights must be **non-negative**. For graphs with negative weights, use Bellman-Ford instead.

---

## How It Works

<p align="center">
  <img src="./assets/dijkstra-overview.png" alt="Dijkstra's Algorithm — Graph and Step Table" width="900">
</p>

**Core idea:**
```
dist[source] = 0
dist[all others] = ∞

Repeat:
  1. Pick the unvisited node u with smallest dist[u]   ← min-heap
  2. For each neighbour v of u:
       if dist[u] + weight(u,v) < dist[v]:
           dist[v] = dist[u] + weight(u,v)             ← relaxation
           update priority queue
  3. Mark u as visited (never revisit)
Until all nodes visited or destination reached
```

---

## Full Trace — Source: A

```
Graph edges:
  A→B: 4    A→C: 2
  C→D: 3    C→E: 3    C→B: 5
  B→D: 10   B→E: 11
  D→E: 4    D→F: 9
  E→F: 7

Initial: dist = {A:0, B:∞, C:∞, D:∞, E:∞, F:∞}
Heap: [(0,A)]

── Iteration 1: Pop (0, A) ───────────────────────────────────────
  Relax A→B: 0+4=4  < ∞  → dist[B]=4
  Relax A→C: 0+2=2  < ∞  → dist[C]=2
  Heap: [(2,C), (4,B)]
  dist = {A:0✓, B:4, C:2, D:∞, E:∞, F:∞}

── Iteration 2: Pop (2, C) ───────────────────────────────────────
  Relax C→D: 2+3=5  < ∞  → dist[D]=5
  Relax C→E: 2+3=5  < ∞  → dist[E]=5
  Relax C→B: 2+5=7  > 4  → no update (B already better via A)
  Heap: [(4,B), (5,D), (5,E)]
  dist = {A:0✓, B:4, C:2✓, D:5, E:5, F:∞}

── Iteration 3: Pop (4, B) ───────────────────────────────────────
  Relax B→D: 4+10=14 > 5 → no update
  Relax B→E: 4+11=15 > 5 → no update
  Heap: [(5,D), (5,E)]
  dist = {A:0✓, B:4✓, C:2✓, D:5, E:5, F:∞}

── Iteration 4: Pop (5, D) ───────────────────────────────────────
  Relax D→E: 5+4=9   > 5 → no update
  Relax D→F: 5+9=14  < ∞ → dist[F]=14
  Heap: [(5,E), (14,F)]
  dist = {A:0✓, B:4✓, C:2✓, D:5✓, E:5, F:14}

── Iteration 5: Pop (5, E) ───────────────────────────────────────
  Relax E→F: 5+7=12  < 14 → dist[F]=12  ← better path found!
  Heap: [(12,F), (14,F_stale)]
  dist = {A:0✓, B:4✓, C:2✓, D:5✓, E:5✓, F:12}

── Iteration 6: Pop (12, F) ──────────────────────────────────────
  No outgoing edges. Done.
  dist = {A:0✓, B:4✓, C:2✓, D:5✓, E:5✓, F:12✓}

Shortest paths from A:
  A→B: 4    (A→B)
  A→C: 2    (A→C)
  A→D: 5    (A→C→D)
  A→E: 5    (A→C→E)
  A→F: 12   (A→C→E→F)  ✅
```

---

## Python Implementation

```python
import heapq
from collections import defaultdict

def dijkstra(graph, source):
    """
    graph: dict of {node: [(neighbour, weight), ...]}
    Returns: dict of shortest distances from source to all nodes
    """
    dist    = defaultdict(lambda: float('inf'))
    dist[source] = 0
    prev    = {}                     # for path reconstruction
    visited = set()
    heap    = [(0, source)]          # (distance, node)

    while heap:
        d, u = heapq.heappop(heap)

        if u in visited:
            continue                 # stale entry — skip
        visited.add(u)

        for v, weight in graph[u]:
            if v in visited:
                continue
            new_dist = d + weight
            if new_dist < dist[v]:
                dist[v] = new_dist
                prev[v] = u
                heapq.heappush(heap, (new_dist, v))

    return dict(dist), prev


def reconstruct_path(prev, source, target):
    """Trace back through prev pointers to get the path"""
    path = []
    node = target
    while node != source:
        if node not in prev:
            return []               # no path exists
        path.append(node)
        node = prev[node]
    path.append(source)
    return path[::-1]               # reverse to get source→target


# Build the graph
graph = defaultdict(list)
edges = [
    ('A','B',4), ('A','C',2),
    ('C','D',3), ('C','E',3), ('C','B',5),
    ('B','D',10),('B','E',11),
    ('D','E',4), ('D','F',9),
    ('E','F',7)
]
for u, v, w in edges:
    graph[u].append((v, w))
    graph[v].append((u, w))   # undirected: add both directions

dist, prev = dijkstra(graph, 'A')
print("Distances:", dict(dist))
# {'A':0, 'C':2, 'B':4, 'D':5, 'E':5, 'F':12}

path = reconstruct_path(prev, 'A', 'F')
print("Shortest path A→F:", "→".join(path))
# A→C→E→F
```

---

## The Relaxation Step — Core of Dijkstra

```
"Relaxing" an edge (u, v) means:
  if dist[u] + weight(u,v) < dist[v]:
      dist[v] = dist[u] + weight(u,v)   ← found a shorter path!
      update priority queue

Visual:

Before relaxation:          After relaxation:
  A ──4── B                   A ──4── B
  │                           │
  2   dist[B] = 7             2   dist[B] = 6  ← improved!
  │                           │
  C ──2── B                   C ──2── B
  (A→C→B = 2+2 = 4... )

Each edge is relaxed at most once → O(E log V) total
```

---

## Handling Stale Heap Entries

Python's `heapq` doesn't support `decrease-key`, so we push duplicates and skip stale ones:

```python
# When we find a better distance, push a NEW entry — don't update old one
heapq.heappush(heap, (new_dist, v))   # new entry

# When popping, skip if we've already visited this node
d, u = heapq.heappop(heap)
if u in visited:
    continue    # this is a stale entry with an outdated distance
```

This is the standard Python approach. The heap may have O(E) entries instead of O(V), but the total complexity remains O((V + E) log E) ≈ O(E log V) for sparse graphs.

---

## Complexity

### Time

| Implementation        | Complexity           | Notes                               |
|-----------------------|---------------------|-------------------------------------|
| Binary heap (heapq)   | **O((V+E) log V)**  | Standard Python approach            |
| Fibonacci heap        | O(V log V + E)      | Theoretical best; complex to implement |
| No heap (linear scan) | O(V²)               | Better for dense graphs (E ≈ V²)    |

### Space

| Metric     | Complexity | Notes                              |
|------------|-----------|-------------------------------------|
| dist array | O(V)      | One entry per vertex                |
| prev array | O(V)      | For path reconstruction             |
| Heap       | O(E)      | At most one entry per edge (stale)  |

---

## Properties at a Glance

| Property                  | Value   | Notes                                          |
|---------------------------|---------|------------------------------------------------|
| Works with negative weights | ❌ No | Use Bellman-Ford instead                       |
| Finds shortest to ONE node | ✅ Yes | Stop early when destination popped             |
| Finds shortest to ALL nodes| ✅ Yes | Continue until heap is empty                   |
| Works on directed graphs  | ✅ Yes | Just add directed edges                        |
| Works on undirected graphs| ✅ Yes | Add edges in both directions                   |
| Detects negative cycles   | ❌ No  | Bellman-Ford required                          |

---

## Dijkstra vs Other Shortest Path Algorithms

| Algorithm      | Weights      | Time             | Negative cycles | Use when               |
|----------------|-------------|------------------|-----------------|------------------------|
| **Dijkstra**   | Non-negative| O((V+E) log V)   | ❌ No detect    | Fast, non-negative weights |
| Bellman-Ford   | Any         | O(V × E)         | ✅ Detects      | Negative weights exist  |
| Floyd-Warshall | Any         | O(V³)            | ✅ Detects      | All-pairs shortest path |
| BFS            | Unweighted  | O(V + E)         | N/A             | Unweighted graphs       |
| A*             | Non-negative| O(E log V)       | ❌              | Single target + heuristic|

---

## Variants

### Single-destination shortest path
Run Dijkstra from source; stop as soon as destination is popped from heap.

```python
if u == destination:
    break   # we have the shortest path to destination
```

### Bidirectional Dijkstra
Run Dijkstra simultaneously from both source and destination; stop when they meet. ~2× faster in practice.

### Dijkstra on grid (matrix)
```python
def min_path_grid(grid):
    rows, cols = len(grid), len(grid[0])
    dist = [[float('inf')] * cols for _ in range(rows)]
    dist[0][0] = grid[0][0]
    heap = [(grid[0][0], 0, 0)]
    while heap:
        d, r, c = heapq.heappop(heap)
        if d > dist[r][c]: continue
        for dr, dc in [(0,1),(1,0),(0,-1),(-1,0)]:
            nr, nc = r+dr, c+dc
            if 0<=nr<rows and 0<=nc<cols:
                nd = d + grid[nr][nc]
                if nd < dist[nr][nc]:
                    dist[nr][nc] = nd
                    heapq.heappush(heap, (nd, nr, nc))
    return dist[rows-1][cols-1]
```

---

## Real-World Uses

```
1. GPS Navigation          — Google Maps, Apple Maps use Dijkstra variants
2. Network routing         — OSPF protocol uses Dijkstra to find shortest routes
3. Flight path planning    — Shortest flight path between airports
4. Game AI pathfinding     — NPC movement (A* is Dijkstra + heuristic)
5. Social network distance — "Degrees of separation" between users
6. Robotics                — Robot navigation in known environments
7. Telecommunications      — Minimise latency in packet routing
```

---

## Common Interview Questions

**Q: Why can't Dijkstra handle negative weights?**
If a negative edge exists, relaxing a node that was already "settled" could find a shorter path — but Dijkstra never re-examines settled nodes. This leads to incorrect results. Bellman-Ford re-examines all edges V-1 times, handling this correctly.

**Q: What's the difference between Dijkstra and BFS?**
BFS finds shortest paths in unweighted graphs (each edge = 1). Dijkstra handles weighted graphs. BFS uses a regular queue; Dijkstra uses a min-heap priority queue to always process the minimum-distance node first.

**Q: How do you reconstruct the actual shortest path?**
Maintain a `prev` dictionary. When you find `dist[v] = dist[u] + w`, set `prev[v] = u`. After the algorithm completes, trace back from destination to source using `prev`, then reverse.

**Q: What is the time complexity and why?**
O((V+E) log V). Each vertex is popped from the heap once — O(V log V). Each edge causes at most one push — O(E log V). Total: O((V+E) log V).

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  DIJKSTRA'S ALGORITHM — AT A GLANCE                               │
│                                                                   │
│  Strategy: Greedy — always process the closest unvisited node    │
│  Tool:     Min-heap (priority queue)                             │
│  Core op:  Relaxation — update dist[v] if shorter path found    │
│                                                                   │
│  Time  →  O((V+E) log V)   Space → O(V + E)                     │
│                                                                   │
│  Constraint: NO negative edge weights                            │
│  Alternative: Bellman-Ford for negative weights                  │
│               A* for single-target with heuristic               │
│                                                                   │
│  Used in: GPS navigation, OSPF routing, A* pathfinding, games   │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 20 of daily algorithm commits — next up: Union-Find (Disjoint Set Union)*
