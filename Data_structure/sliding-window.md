# 🪟 Sliding Window Pattern

> *"Instead of recomputing from scratch each time, slide — keep what's still useful, drop what's left behind."*

---

## What is the Sliding Window Pattern?

The **Sliding Window** is an algorithmic pattern (not a data structure) that maintains a **contiguous subset of a sequence** — a "window" — and slides it across the data to avoid redundant computation.

Instead of using two nested loops to check every possible subarray in **O(n²)**, a sliding window processes each element at most twice (once when added, once when removed), giving **O(n)**.

Two flavours:
- **Fixed window** — window size k is constant; slide it across
- **Variable window** — window expands/shrinks based on a condition

---

## Visualisation

<p align="center">
  <img src="./assets/sliding-window-overview.png" alt="Sliding Window — Fixed and Variable Window Examples" width="900">
</p>

---

## Fixed Window — Template

Window size k is constant. Slide by removing the leftmost element and adding the next right element.

```
arr = [2, 1, 5, 1, 3, 2],  k = 3

Initial window:  [2, 1, 5]   sum = 8
Slide →:  remove 2, add 1 → [1, 5, 1]   sum = 8-2+1 = 7
Slide →:  remove 1, add 3 → [5, 1, 3]   sum = 7-1+3 = 9  ← max
Slide →:  remove 5, add 2 → [1, 3, 2]   sum = 9-5+2 = 6

Key: no recomputing the whole sum — just add one, subtract one → O(n)
```

```python
def max_sum_fixed_window(arr, k):
    n = len(arr)
    if n < k:
        return -1

    # Build initial window
    window_sum = sum(arr[:k])
    max_sum    = window_sum

    # Slide the window
    for i in range(k, n):
        window_sum += arr[i]       # add new right element
        window_sum -= arr[i - k]   # remove old left element
        max_sum = max(max_sum, window_sum)

    return max_sum


print(max_sum_fixed_window([2,1,5,1,3,2], 3))   # 9
```

---

## Variable Window — Template

Window size changes dynamically. Expand the right pointer to grow; shrink the left pointer when a condition is violated.

```python
def variable_window_template(arr, condition):
    left  = 0
    result = 0
    state  = ...   # window state (sum, set, counter, etc.)

    for right in range(len(arr)):
        # Step 1: Expand — add arr[right] to window
        state = update(state, arr[right])

        # Step 2: Shrink — while condition violated, remove arr[left]
        while condition_violated(state):
            state = remove(state, arr[left])
            left += 1

        # Step 3: Record — window [left..right] is now valid
        result = max(result, right - left + 1)

    return result
```

---

## Classic Problem 1 — Maximum Sum Subarray of Size K

```python
def max_sum_subarray(arr, k):
    window_sum = sum(arr[:k])
    max_sum    = window_sum
    for i in range(k, len(arr)):
        window_sum += arr[i] - arr[i - k]
        max_sum = max(max_sum, window_sum)
    return max_sum

print(max_sum_subarray([2,1,5,1,3,2], 3))   # 9
```

---

## Classic Problem 2 — Longest Substring Without Repeating Characters

```python
def length_of_longest_substring(s):
    char_set = set()
    left     = 0
    result   = 0

    for right in range(len(s)):
        # Shrink: while duplicate found, remove from left
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1

        char_set.add(s[right])
        result = max(result, right - left + 1)

    return result


print(length_of_longest_substring("abcabcbb"))   # 3  ("abc")
print(length_of_longest_substring("pwwkew"))     # 3  ("wke")
print(length_of_longest_substring("bbbbb"))      # 1  ("b")
```

```
Trace: s = "abcabcbb"
  r=0 (a): set={a}           window="a"       len=1
  r=1 (b): set={a,b}         window="ab"      len=2
  r=2 (c): set={a,b,c}       window="abc"     len=3
  r=3 (a): 'a' in set → remove 'a', l=1
            set={b,c,a}       window="bca"     len=3
  r=4 (b): 'b' in set → remove 'b', l=2
            set={c,a,b}       window="cab"     len=3
  r=5 (c): 'c' in set → remove 'c', l=3
            set={a,b,c}       window="abc"     len=3
  ...
  Result: 3
```

---

## Classic Problem 3 — Minimum Window Substring

Find the smallest substring of `s` containing all characters of `t`.

```python
from collections import Counter

def min_window_substring(s, t):
    if not t or not s:
        return ""

    need    = Counter(t)       # chars we still need
    missing = len(t)           # total chars still needed
    left    = 0
    best    = (float('inf'), 0, 0)   # (length, left, right)

    for right, char in enumerate(s):
        if need[char] > 0:
            missing -= 1       # one more needed char found
        need[char] -= 1        # decrement (can go negative)

        if missing == 0:       # valid window found — try to shrink
            while need[s[left]] < 0:
                need[s[left]] += 1
                left += 1

            if right - left + 1 < best[0]:
                best = (right - left + 1, left, right + 1)

            need[s[left]] += 1    # release leftmost char
            missing += 1
            left += 1

    return s[best[1]:best[2]] if best[0] != float('inf') else ""


print(min_window_substring("ADOBECODEBANC", "ABC"))   # "BANC"
print(min_window_substring("a", "a"))                  # "a"
print(min_window_substring("a", "aa"))                 # ""
```

---

## Classic Problem 4 — Longest Subarray with Sum ≤ K

```python
def longest_subarray_sum_k(arr, k):
    left     = 0
    curr_sum = 0
    result   = 0

    for right in range(len(arr)):
        curr_sum += arr[right]                   # expand right

        while curr_sum > k and left <= right:    # shrink left
            curr_sum -= arr[left]
            left += 1

        result = max(result, right - left + 1)

    return result

print(longest_subarray_sum_k([4,2,1,6,3,1,2], 7))   # 3  ([2,1] or [1,6] → max=3 with [2,1,6]? no... [4,2,1]=7 len=3)
```

---

## Classic Problem 5 — Fruits Into Baskets (At Most 2 Distinct)

```python
def total_fruit(fruits):
    basket = Counter()
    left   = 0
    result = 0

    for right in range(len(fruits)):
        basket[fruits[right]] += 1

        while len(basket) > 2:              # more than 2 fruit types
            basket[fruits[left]] -= 1
            if basket[fruits[left]] == 0:
                del basket[fruits[left]]
            left += 1

        result = max(result, right - left + 1)

    return result

print(total_fruit([1,2,1,2,3]))   # 4  ([1,2,1,2])
print(total_fruit([3,3,3,1,2,1,1,2,3,3,4]))   # 5
```

---

## Classic Problem 6 — Maximum Consecutive Ones III (At Most K Zeros)

```python
def longest_ones(nums, k):
    left  = 0
    zeros = 0
    result = 0

    for right in range(len(nums)):
        if nums[right] == 0:
            zeros += 1

        while zeros > k:
            if nums[left] == 0:
                zeros -= 1
            left += 1

        result = max(result, right - left + 1)

    return result

print(longest_ones([1,1,1,0,0,0,1,1,1,1,0], 2))   # 6
print(longest_ones([0,0,1,1,0,0,1,1,1,0,1,1,0,0,0,1,1,1,1], 3))   # 10
```

---

## Fixed vs Variable Window Decision Guide

```
Is k (window size) given?
  YES → Fixed window
        Template: add arr[right], subtract arr[right-k], track best

  NO  → Variable window
        Template: expand right, shrink left while violated, track best

What is the "condition"?
  sum ≤ target       → track running sum
  at most k distinct → track Counter of window
  no duplicates      → track set of window
  k zeros allowed    → track count of zeros
```

---

## Complexity

| Approach          | Time   | Space  | Notes                              |
|-------------------|--------|--------|------------------------------------|
| Brute force (2 loops) | O(n²) | O(1) | Check every subarray               |
| Fixed window      | **O(n)** | O(1) | Each element added/removed once    |
| Variable window   | **O(n)** | O(k) | k = window state size (set/counter)|

> The sliding window turns O(n²) into O(n) because each element is **added at most once** (when right pointer passes it) and **removed at most once** (when left pointer passes it). Total operations = 2n = O(n).

---

## Recognising Sliding Window Problems

```
✅ USE sliding window when you see:
  • "subarray / substring of size k"       → fixed window
  • "longest / shortest subarray where..." → variable window
  • "contiguous elements satisfying..."    → variable window
  • "at most k distinct..."                → variable window + counter
  • "no repeating characters..."           → variable window + set

❌ SLIDING WINDOW DOESN'T WORK when:
  • Elements are not contiguous (subsequences)
  • Need global optimum over non-contiguous subsets
  • Data is non-linear (trees, graphs)
  → Use DP or two pointers instead
```

---

## Sliding Window vs Two Pointers

```
Two Pointers:      left and right, but they can cross,
                   and one often starts from the end
                   Used for: pair sum, palindromes, sorted arrays

Sliding Window:    left and right, always left ≤ right,
                   define a contiguous window between them
                   Used for: subarrays/substrings, contiguous data

Overlap: sliding window IS a form of two pointers.
All sliding window problems use two pointers,
but not all two-pointer problems use a window.
```

---

## Real-World Uses

```
1. Network monitoring      — average bandwidth over last k seconds
2. Stock prices            — max/min price in rolling k-day window
3. Text search             — find shortest substring containing all keywords
4. Video streaming         — buffer management over sliding time window
5. Rate limiting           — count API requests in last 60 seconds (fixed window)
6. Fraud detection         — flag unusual transaction patterns in recent history
7. DNA analysis            — find patterns in genome sequences
```

---

## Key Takeaways

```
┌───────────────────────────────────────────────────────────────────┐
│  SLIDING WINDOW — AT A GLANCE                                     │
│                                                                   │
│  Pattern: maintain a window [left..right] over a sequence        │
│  Gain:    O(n²) brute force → O(n) with sliding window          │
│                                                                   │
│  Fixed window:    size k given → slide by add right, drop left   │
│  Variable window: condition given → expand right, shrink left    │
│                                                                   │
│  Each element enters window once, exits once → O(n) total       │
│                                                                   │
│  Classic problems:                                                │
│    Max sum subarray k  · Longest no-repeat substring             │
│    Min window substring · At most k distinct · K zeros allowed   │
│                                                                   │
│  Recognise by: "subarray", "substring", "contiguous", "window"   │
└───────────────────────────────────────────────────────────────────┘
```

---

*Day 22 of daily algorithm commits — next up: Two Pointers Pattern*
