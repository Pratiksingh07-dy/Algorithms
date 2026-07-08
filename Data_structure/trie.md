# 🌿 Trie (Prefix Tree)

> *"A tree where every path from root to a node spells out a string. The most natural data structure for words."*

![Trie Overview](./assets/trie-overview.png)

---

## What is a Trie?

A **Trie** (pronounced "try", from re**trie**val) is a tree-based data structure for storing strings where:
- Each **node** represents a single character
- Each **path from root to a marked node** represents a complete word
- All words sharing a common **prefix** share the same path from the root

Unlike a Hash Table, a Trie stores strings in a way that naturally groups them by prefix — making it the ideal structure for **autocomplete, spell-check, IP routing, and prefix search**.

---

## Visualising a Trie

![Trie Structure Diagram](./assets/trie-structure.png)

Words: `["cat", "car", "card", "care", "bat", "ball"]`

```
                 root
               /      \
              c          b
              │          │
              a          a
             / \        / \
            t   r      t   l
            *   │      *   │
               / \         l
              d   e        *
              *   *

* = end of word marker

Paths:
  root→c→a→t*          = "cat"
  root→c→a→r→d*        = "card"
  root→c→a→r→e*        = "care"
  root→c→a→r           = "car" (marked with *)
  root→b→a→t*          = "bat"
  root→b→a→l→l*        = "ball"

Shared prefix "ca": "cat", "car", "card", "care"
Shared prefix "ba": "bat", "ball"
```

---

## Node Structure

```python
class TrieNode:
    def __init__(self):
        self.children   = {}       # char → TrieNode
        self.is_end     = False    # marks end of a complete word
        # Optional: store the word itself for easy retrieval
        # self.word = None
```

---

## Full Trie Implementation

```python
class Trie:
    def __init__(self):
        self.root = TrieNode()

    # ── Insert ─────────────────────────────────────────────────────

    def insert(self, word):
        """O(m) — m = length of word"""
        node = self.root
        for char in word:
            if char not in node.children:
                node.children[char] = TrieNode()   # create node if missing
            node = node.children[char]
        node.is_end = True                         # mark end of word

    # ── Search ─────────────────────────────────────────────────────

    def search(self, word):
        """O(m) — returns True only if exact word exists"""
        node = self._find_node(word)
        return node is not None and node.is_end

    def starts_with(self, prefix):
        """O(m) — returns True if any word starts with prefix"""
        return self._find_node(prefix) is not None

    def _find_node(self, prefix):
        """O(m) — traverse to the end of prefix, return node or None"""
        node = self.root
        for char in prefix:
            if char not in node.children:
                return None
            node = node.children[char]
        return node

    # ── Delete ─────────────────────────────────────────────────────

    def delete(self, word):
        """O(m) — remove word; clean up orphan nodes"""
        def _delete(node, word, depth):
            if depth == len(word):
                if not node.is_end:
                    return False            # word doesn't exist
                node.is_end = False
                return len(node.children) == 0   # True if node can be deleted
            char = word[depth]
            if char not in node.children:
                return False
            should_delete = _delete(node.children[char], word, depth + 1)
            if should_delete:
                del node.children[char]
                return len(node.children) == 0 and not node.is_end
            return False

        _delete(self.root, word, 0)

    # ── Autocomplete ───────────────────────────────────────────────

    def autocomplete(self, prefix):
        """O(m + k) — return all words with given prefix"""
        node = self._find_node(prefix)
        if not node:
            return []
        results = []
        self._collect(node, prefix, results)
        return results

    def _collect(self, node, current, results):
        if node.is_end:
            results.append(current)
        for char, child in node.children.items():
            self._collect(child, current + char, results)

    # ── Count words with prefix ────────────────────────────────────

    def count_prefix(self, prefix):
        """O(m + k) — count words starting with prefix"""
        return len(self.autocomplete(prefix))

    # ── Get all words ──────────────────────────────────────────────

    def all_words(self):
        return self.autocomplete("")


# Usage
trie = Trie()
for word in ["cat", "car", "card", "care", "bat", "ball"]:
    trie.insert(word)

print(trie.search("car"))         # True
print(trie.search("ca"))          # False  (not a complete word)
print(trie.starts_with("ca"))     # True
print(trie.autocomplete("car"))   # ['car', 'card', 'care']
print(trie.autocomplete("ba"))    # ['bat', 'ball']
trie.delete("car")
print(trie.search("car"))         # False
print(trie.search("card"))        # True  (card still exists)
```

---

## Step-by-Step Insert Trace

![Trie Insert Diagram](./assets/trie-insert.png)

```
Insert "car" into empty Trie:

Step 1: char='c'  → root has no 'c' → create node
  root → [c]

Step 2: char='a'  → [c] has no 'a' → create node
  root → [c] → [a]

Step 3: char='r'  → [a] has no 'r' → create node, mark end
  root → [c] → [a] → [r]*

Insert "card":

Step 1: char='c'  → root has 'c' → follow existing node
Step 2: char='a'  → [c] has 'a'  → follow existing node
Step 3: char='r'  → [a] has 'r'  → follow existing node
Step 4: char='d'  → [r] has no 'd' → create node, mark end
  root → [c] → [a] → [r]* → [d]*

"car" and "card" now share the path root→c→a→r
```

---

## Complexity

### Time

| Operation        | Complexity | Notes                              |
|------------------|-----------|-------------------------------------|
| Insert           | **O(m)**  | m = word length                     |
| Search (exact)   | **O(m)**  | Traverse to end of word             |
| Starts with      | **O(m)**  | Traverse to end of prefix           |
| Autocomplete     | O(m + k)  | k = number of matching words        |
| Delete           | **O(m)**  | Traverse + optional node cleanup    |

### Space

| Metric        | Complexity | Notes                                       |
|---------------|-----------|----------------------------------------------|
| Storage       | **O(n·m)**| n = number of words, m = average word length |
| Worst case    | O(26·n·m) | If no prefix sharing (all unique prefixes)   |
| Best case     | O(m)      | All words share the same prefix              |

> Tries use more memory than a Hash Table for storing words, but offer prefix operations that a Hash Table cannot do efficiently.

---

## Properties at a Glance

| Property             | Value   | Notes                                          |
|----------------------|---------|------------------------------------------------|
| Ordered              | ✅ Yes  | Words stored in lexicographic order            |
| Prefix search        | ✅ Yes  | Core strength — O(m) per prefix query          |
| Exact search         | ✅ Yes  | O(m) — same as hash table's O(1) avg for long words |
| Duplicate handling   | ✅ Yes  | Duplicates just re-traverse existing nodes     |
| Memory efficient     | ⚠️      | Good when many shared prefixes, poor otherwise |

---

## Trie vs Hash Table vs BST for String Storage

![Trie vs Hash Table vs BST](./assets/trie-vs-others.png)

| Feature              | Trie          | Hash Table   | BST (balanced) |
|----------------------|--------------|-------------|----------------|
| Exact search         | O(m)         | O(m) avg    | O(m log n)     |
| Prefix search        | **O(m)**     | O(n)        | O(m log n)     |
| Autocomplete         | **O(m + k)** | O(n)        | O(m log n + k) |
| Sorted iteration     | **O(n·m)**   | O(n·m log n)| O(n·m)         |
| Space                | O(n·m)       | O(n·m)      | O(n·m)         |
| Hash collisions      | N/A          | Possible    | N/A            |

---

## Classic Interview Problems

### 1. Word Search II (find all words in a grid)

```python
def find_words(board, words):
    trie = Trie()
    for word in words:
        trie.insert(word)

    rows, cols = len(board), len(board[0])
    found = set()

    def dfs(node, r, c, path):
        ch = board[r][c]
        if ch not in node.children:
            return
        next_node = node.children[ch]
        path += ch
        if next_node.is_end:
            found.add(path)
        board[r][c] = '#'   # mark visited
        for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
            nr, nc = r+dr, c+dc
            if 0<=nr<rows and 0<=nc<cols and board[nr][nc] != '#':
                dfs(next_node, nr, nc, path)
        board[r][c] = ch    # restore

    for r in range(rows):
        for c in range(cols):
            dfs(trie.root, r, c, "")
    return list(found)
```

### 2. Longest Common Prefix

```python
def longest_common_prefix(words):
    trie = Trie()
    for word in words:
        trie.insert(word)
    prefix = ""
    node = trie.root
    while len(node.children) == 1 and not node.is_end:
        char = next(iter(node.children))
        prefix += char
        node = node.children[char]
    return prefix

print(longest_common_prefix(["flower","flow","flight"]))   # "fl"
print(longest_common_prefix(["dog","racecar","car"]))      # ""
```

### 3. Count Unique Substrings

```python
def count_unique_substrings(s):
    """Insert all suffixes of s into a Trie; count total nodes"""
    trie = Trie()
    for i in range(len(s)):
        trie.insert(s[i:])

    def count_nodes(node):
        total = 1
        for child in node.children.values():
            total += count_nodes(child)
        return total

    return count_nodes(trie.root) - 1   # subtract root
```

### 4. Wildcard Pattern Matching

```python
def search_with_wildcard(trie_root, pattern):
    """'.' matches any single character"""
    results = []

    def dfs(node, i, current):
        if i == len(pattern):
            if node.is_end:
                results.append(current)
            return
        ch = pattern[i]
        if ch == '.':
            for c, child in node.children.items():
                dfs(child, i+1, current+c)
        elif ch in node.children:
            dfs(node.children[ch], i+1, current+ch)

    dfs(trie_root, 0, "")
    return results
```

---

## Real-World Uses

```
1. Autocomplete / search suggestions  — Google, IDE code completion
2. Spell checker                      — find closest word in dictionary
3. IP routing (Radix Trie)            — longest prefix match in routers
4. T9 predictive text                 — map digit sequences to word suggestions
5. Boggle / word games                — fast prefix pruning while searching the board
6. DNA sequence search                — genome substring queries
7. Contact/phone book search          — find all names starting with typed letters
8. Browser URL bar                    — prefix-match against history/bookmarks
```

---

## Compressed Trie (Radix Tree)

When a Trie has many nodes with only one child, it wastes space. A **Radix Tree** (compressed Trie) merges single-child chains into one node labelled with the full substring.

![Radix Tree Diagram](./assets/radix-tree.png)

```
Normal Trie for ["card", "care"]:          Compressed Radix Tree:
  root→c→a→r→d*                             root → "car" → d*
             →e*                                          → e*

Saves nodes, same O(m) operations.
Used in: Linux kernel routing, Apache HTTP server.
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  TRIE — AT A GLANCE                                               │
│                                                                   │
│  Structure:  Tree of characters; paths spell words               │
│  Strength:   Prefix operations — autocomplete, starts_with       │
│                                                                   │
│  Insert      → O(m)       m = word length                        │
│  Search      → O(m)                                              │
│  Autocomplete→ O(m + k)   k = number of results                 │
│  Space       → O(n·m)     n = number of words                    │
│                                                                   │
│  Beats Hash Table for: prefix search, autocomplete, sorted output│
│  Beats BST for:        exact prefix queries on strings           │
│                                                                   │
│  Variants: Compressed Trie (Radix Tree), Suffix Trie, Aho-Corasick│
│                                                                   │
│  Used in: search engines, spell checkers, IP routing, T9 text    │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 18 of daily algorithm commits — next up: Dynamic Programming*
