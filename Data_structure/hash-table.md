# 🗂️ Hash Table

> *"The data structure that turns 'find in O(n)' into 'find in O(1)' — through the magic of hashing."*

![Hash Table Overview](./assets/hash-table-overview.png)

---

## What is a Hash Table?

A Hash Table (also called a **Hash Map** or **Dictionary**) is a data structure that maps **keys to values** using a **hash function**. It provides average O(1) time for insertion, deletion, and lookup — faster than any comparison-based structure.

The core idea:
1. Take a key (e.g. `"name"`)
2. Run it through a **hash function** → produces an integer (the **hash**)
3. Use `hash % table_size` as the **index** into an array
4. Store the value at that index

```
key = "name"
hash("name") = 2718364823
index = 2718364823 % 8 = 7
arr[7] = "Alice"
```

---

## How a Hash Function Works

![Hash Function Diagram](./assets/hash-function-diagram.png)

A good hash function:
- Is **deterministic** — same key always gives same hash
- Is **fast** to compute — O(1) ideally
- **Distributes keys uniformly** — avoids clustering in one bucket
- **Minimises collisions** — two keys rarely produce the same index

```
Simple hash for strings (djb2 algorithm):
  hash = 5381
  for each character c in key:
      hash = hash * 33 + ord(c)
  return hash % table_size

"cat" → hash = 193491849 → index = 193491849 % 10 = 9
"dog" → hash = 193502722 → index = 193502722 % 10 = 2
"cow" → hash = 193491699 → index = 193491699 % 10 = 9  ← COLLISION with "cat"
```

---

## Collisions

A **collision** occurs when two different keys hash to the same index. Collisions are inevitable (by the **Pigeonhole Principle** — more possible keys than table slots). The strategy used to resolve them defines the hash table's behaviour.

![Collision Types Diagram](./assets/hash-collision-types.png)

---

## Collision Resolution Strategy 1 — Chaining

Each bucket holds a **linked list** of all key-value pairs that hash to that index.

![Separate Chaining Diagram](./assets/hash-chaining.png)

```
Table (size 8):

index 0: →  NULL
index 1: →  [("apple", 5)] → NULL
index 2: →  [("dog", 12)] → NULL
index 3: →  NULL
index 4: →  [("cat", 7)] → [("cow", 3)] → NULL   ← collision handled by chain
index 5: →  NULL
index 6: →  [("fish", 9)] → NULL
index 7: →  [("name", "Alice")] → NULL
```

- **Lookup:** hash the key → go to bucket → scan the chain for the key
- **Worst case:** all keys hash to one bucket → O(n) per operation
- **Average case:** O(1) with a good hash function and low load factor

---

## Collision Resolution Strategy 2 — Open Addressing

All entries live **inside the array** itself. On collision, probe the array for the next available slot.

![Open Addressing Diagram](./assets/hash-open-addressing.png)

### Linear Probing
```
On collision at index i: try i+1, i+2, i+3, ...

Insert "cat" → index 4 (occupied by "cow")
Probe index 5 → empty → store "cat" at index 5

Lookup "cat":
  hash("cat") = 4 → check index 4 → "cow" ≠ "cat"
  Probe index 5 → "cat" = "cat" → found ✅
```

### Quadratic Probing
```
On collision at index i: try i+1², i+2², i+3², ...
Reduces clustering compared to linear probing.
```

### Double Hashing
```
On collision at index i: try i + k*hash2(key) for k=1,2,3,...
hash2 is a secondary hash function.
Most uniform distribution — lowest clustering.
```

---

## Load Factor

The **load factor** α = n / m where n = number of entries, m = table size.

```
α = 0.1  → sparse table, many empty slots, very few collisions
α = 0.7  → typical target for open addressing (Python dicts use ~0.67)
α = 1.0  → table full; performance degrades significantly
α > 1.0  → only possible with chaining; chain lengths grow

Rule of thumb:
  Chaining:        resize when α > 0.75
  Open Addressing: resize when α > 0.70
```

When the load factor is exceeded, the table **resizes** (typically doubles) and **rehashes** all entries — O(n), but amortised O(1) per insertion.

---

## Python Implementation (Chaining)

```python
class HashTable:
    def __init__(self, capacity=8):
        self._capacity = capacity
        self._size     = 0
        self._buckets  = [[] for _ in range(capacity)]

    def _hash(self, key):
        return hash(key) % self._capacity

    def put(self, key, value):
        """O(1) average — insert or update"""
        idx = self._hash(key)
        bucket = self._buckets[idx]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket[i] = (key, value)   # update existing
                return
        bucket.append((key, value))        # insert new
        self._size += 1
        if self._size / self._capacity > 0.75:
            self._resize()

    def get(self, key, default=None):
        """O(1) average — retrieve value by key"""
        idx = self._hash(key)
        for k, v in self._buckets[idx]:
            if k == key:
                return v
        return default

    def delete(self, key):
        """O(1) average — remove key-value pair"""
        idx = self._hash(key)
        bucket = self._buckets[idx]
        for i, (k, v) in enumerate(bucket):
            if k == key:
                bucket.pop(i)
                self._size -= 1
                return True
        return False

    def _resize(self):
        """O(n) — double capacity and rehash all entries"""
        old_buckets    = self._buckets
        self._capacity *= 2
        self._buckets  = [[] for _ in range(self._capacity)]
        self._size     = 0
        for bucket in old_buckets:
            for key, value in bucket:
                self.put(key, value)

    def __contains__(self, key):
        return self.get(key) is not None

    def __repr__(self):
        items = [(k, v) for b in self._buckets for k, v in b]
        return f"HashTable({items})"


# Usage
ht = HashTable()
ht.put("name", "Alice")
ht.put("age", 30)
ht.put("city", "London")
print(ht.get("name"))     # Alice
print(ht.get("age"))      # 30
ht.delete("age")
print("age" in ht)        # False
```

---

## Complexity

### Time

| Operation | Average | Worst Case (all collide) |
|-----------|---------|--------------------------|
| Insert    | **O(1)**| O(n)                     |
| Lookup    | **O(1)**| O(n)                     |
| Delete    | **O(1)**| O(n)                     |
| Resize    | O(n)    | O(n)                     |

### Space

| Metric    | Complexity | Notes                              |
|-----------|-----------|-------------------------------------|
| Storage   | **O(n)**  | One slot per key-value pair         |
| Chaining  | O(n + m)  | m = number of buckets               |

---

## Properties at a Glance

| Property          | Value   | Notes                                          |
|-------------------|---------|------------------------------------------------|
| Ordered           | ❌ No   | Hash maps don't preserve insertion order*      |
| Allows duplicates | ❌ No   | Keys are unique; values can repeat             |
| Null keys         | ⚠️     | Python dicts allow `None` as a key             |
| Thread-safe       | ❌ No   | Use `threading.Lock` or `concurrent.futures`   |

> *Python's `dict` (3.7+) **does** preserve insertion order as an implementation detail, but Hash Tables in general do not guarantee it.

---

## Python's Built-in `dict`

Python's `dict` **is** a hash table — highly optimised with open addressing and compact memory layout.

```python
# dict is already a hash table — use it directly
d = {}
d["name"] = "Alice"        # O(1) insert
d["age"]  = 30
print(d["name"])           # O(1) lookup → "Alice"
del d["age"]               # O(1) delete
print("age" in d)          # O(1) membership test → False

# Common patterns
d.get("missing", "default")    # safe lookup with default
d.setdefault("count", 0)       # insert if key doesn't exist
d.items()                      # iterate key-value pairs
d.keys()                       # iterate keys
d.values()                     # iterate values

# Counter — hash table for frequency counting
from collections import Counter
freq = Counter("mississippi")
print(freq)   # Counter({'s': 4, 'i': 4, 'p': 2, 'm': 1})

# defaultdict — hash table with default value factory
from collections import defaultdict
graph = defaultdict(list)
graph["A"].append("B")   # no KeyError if "A" doesn't exist
```

---

## Classic Interview Problems

### 1. Two Sum — O(n)
```python
def two_sum(nums, target):
    seen = {}
    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i

print(two_sum([2, 7, 11, 15], 9))   # [0, 1]
```

### 2. First non-repeating character
```python
def first_unique(s):
    freq = Counter(s)
    for i, ch in enumerate(s):
        if freq[ch] == 1:
            return i
    return -1

print(first_unique("leetcode"))   # 0 ('l')
```

### 3. Group anagrams
```python
def group_anagrams(words):
    groups = defaultdict(list)
    for word in words:
        key = tuple(sorted(word))   # anagrams share the same sorted key
        groups[key].append(word)
    return list(groups.values())

print(group_anagrams(["eat","tea","tan","ate","nat","bat"]))
# [['eat','tea','ate'], ['tan','nat'], ['bat']]
```

### 4. Subarray sum equals k — O(n)
```python
def subarray_sum(nums, k):
    count, prefix = 0, 0
    seen = {0: 1}
    for num in nums:
        prefix += num
        count  += seen.get(prefix - k, 0)
        seen[prefix] = seen.get(prefix, 0) + 1
    return count
```

---

## Hash Table vs Other Structures

| Feature           | Hash Table | Array  | BST        | Linked List |
|-------------------|-----------|--------|------------|-------------|
| Lookup by key     | **O(1)**  | O(n)   | O(log n)   | O(n)        |
| Insert            | **O(1)**  | O(1)** | O(log n)   | O(1)        |
| Delete            | **O(1)**  | O(n)   | O(log n)   | O(1)*       |
| Ordered iteration | ❌        | ✅     | ✅ (sorted)| ✅          |
| Range queries     | ❌        | ✅     | ✅         | ❌          |
| Memory            | O(n+m)    | O(n)   | O(n)       | O(n)        |

---

## Real-World Uses

```
1. Programming language runtimes  — variable/symbol lookup tables
2. Database indexing              — hash indexes for exact-match queries
3. Caches (LRU, CDN)             — key = URL/resource, value = cached response
4. Compilers                     — symbol tables map identifiers to types/addresses
5. DNS resolution                 — domain name → IP address mapping
6. Counting / frequency analysis  — word count, histogram, anagram grouping
7. Sets                           — Python `set` is a hash table without values
8. Cryptographic hash functions   — SHA-256, MD5 (not for hash tables but same idea)
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  HASH TABLE — AT A GLANCE                                         │
│                                                                   │
│  Core idea:  hash(key) % size → array index → O(1) access        │
│                                                                   │
│  Time  →  O(1) avg for insert, lookup, delete                    │
│  Space →  O(n)                                                   │
│                                                                   │
│  Collision handling:                                              │
│    • Chaining       — linked list per bucket                     │
│    • Open Addressing— linear/quadratic probe or double hash      │
│                                                                   │
│  Load factor α = n/m → resize at ~0.75 to maintain O(1)         │
│  Python dict IS a hash table — use it directly                   │
│                                                                   │
│  Best for: key-value store, frequency count, deduplication       │
│  Avoid for: ordered data, range queries → use BST instead        │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 14 of daily algorithm commits — next up: Binary Search Tree*
