# Heap Patterns Guide

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Pattern Recognition Flowchart

```
Does the problem involve repeatedly finding min/max?
    YES →
        Fixed k? (k largest, k smallest, k frequent)
            YES → Top K Pattern (opposite heap, size k)
        Merging sorted sequences?
            YES → K-Way Merge (min-heap with (val, listIdx, elemIdx))
        Tracking median or two halves?
            YES → Two Heaps (lo max-heap + hi min-heap)
        Scheduling with frequency/cooldown?
            YES → Heap + Frequency (max-heap by freq + cooldown queue)
    NO → Consider sorting, prefix sum, or other approach
```

---

## Pattern 1: Top K Elements

**Signal:** "Find k largest/smallest/frequent/closest"
**Structure:** Opposite heap of size k; root = the kth element

| Variant | Heap Type | Size | Eviction |
|---------|----------|------|---------|
| k largest | Min-heap | k | Remove smallest when size > k |
| k smallest | Max-heap | k | Remove largest when size > k |
| k most frequent | Min-heap (by freq) | k | Remove least frequent |
| k closest | Max-heap (by dist²) | k | Remove farthest |

**When to use quickselect instead:** k = 1 or k ≈ n/2, array is mutable, O(n) average is needed.

---

## Pattern 2: K-Way Merge

**Signal:** "Given k sorted lists/arrays, find the kth smallest" or "merge k sorted sequences"
**Structure:** Min-heap of size k, one entry per list, advance within list on each poll

**Heap entry:** `[value, listIndex, elementIndex]`
**Init:** Push first element of each list
**Advance:** After polling `[v, i, j]`, push `[list[i][j+1], i, j+1]` if j+1 valid

**Problems:**
- Merge K Sorted Lists — full merge until heap empty
- Kth Smallest in Matrix — k polls
- K Pairs Smallest Sums — k polls with 2D index
- Smallest Range from K Lists — advance minimum until a list exhausted

---

## Pattern 3: Two Heaps

**Signal:** "Median", "balance two groups", "greedy pick from two pools"
**Structure:** lo = max-heap (lower half), hi = min-heap (upper half)

**Invariants:**
- `|lo.size() - hi.size()| ≤ 1` (lo can have 1 extra)
- `lo.top() ≤ hi.top()` (every lo element ≤ every hi element)

**Median extraction:**
- Odd total: `lo.top()`
- Even total: `(lo.top() + hi.top()) / 2.0`

**Variant — IPO (two pools, different semantics):**
- `locked` = min-heap by capital (projects not yet affordable)
- `available` = max-heap by profit (affordable projects)
- Pattern: unlock from `locked` to `available` as capital grows; always pick max profit from `available`

---

## Pattern 4: Heap + Frequency

**Signal:** "Arrange/schedule tasks", "no two adjacent same", "rearrange by frequency"
**Structure:** Frequency map → max-heap by frequency → greedy pick → cooldown

**Key check:** If max frequency > `(n+1)/2`, it's impossible to arrange without adjacent duplicates.

**Template:**
```cpp
#include <bits/stdc++.h>
using namespace std;

unordered_map<int, int> freq = buildFreqMap(input);
priority_queue<vector<int>> maxHeap; // max-heap by default; store {freq, elem}
for (auto& [key, val] : freq) maxHeap.push({val, encode(key)});

// Process in rounds: pick most frequent, place, decrement, put back if still needed
// For cooldown: queue of (freq, available_at_time)
```

---

## Heap vs Other Approaches

| Problem Type | Heap | Sort | Prefix Sum | Hash Map |
|-------------|------|------|-----------|---------|
| k largest (offline) | O(n log k) | O(n log n) | — | — |
| k largest (streaming) | O(log k) per element | Not applicable | — | — |
| k most frequent | O(n log k) | O(n log n) | — | O(n) bucket sort |
| Median stream | O(log n) add | Not applicable | — | O(1) bucket (bounded range) |
| Merge k sorted | O(n log k) | O(n log n) | — | — |
| Kth smallest matrix | O(k log k) | O(n² log n) | Binary search | — |

---

## C++ priority_queue Patterns for Custom Objects

```cpp
#include <bits/stdc++.h>
using namespace std;

// Records/ints: simple comparator (min-heap by first element)
auto cmp = [](const vector<int>& a, const vector<int>& b) { return a[0] > b[0]; };
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> pq(cmp);

// Multi-key sort: primary by val, secondary by index
auto cmp2 = [](const vector<int>& a, const vector<int>& b) {
    return a[0] != b[0] ? a[0] > b[0] : a[1] > b[1];
};
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp2)> pq2(cmp2);

// String with frequency: lower freq → evict first; for equal freq, lex later → evict first
auto cmp3 = [&](const string& a, const string& b) {
    int fa = freq[a], fb = freq[b];
    return fa != fb ? fa > fb : a < b;
};
priority_queue<string, vector<string>, decltype(cmp3)> pq3(cmp3);
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
