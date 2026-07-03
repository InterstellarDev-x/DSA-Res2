# Heap Basics

> **Topic:** [Heaps](../README.md) · **Pattern 1 of 5**
> **Problems:** Last Stone Weight · heap fundamentals

---

## Core Concept

A **heap** is a complete binary tree satisfying the heap property:
- **Min-heap:** parent ≤ children → root = minimum element
- **Max-heap:** parent ≥ children → root = maximum element

C++'s `std::priority_queue` is a **max-heap** by default. To get a min-heap, use `greater<T>` as the comparator.

---

## C++ priority_queue API

```cpp
#include <bits/stdc++.h>
using namespace std;

// Min-heap
priority_queue<int, vector<int>, greater<int>> minHeap;

// Max-heap (default)
priority_queue<int> maxHeap;

// Custom comparator — sort by absolute value (min-heap by abs value)
auto cmp = [](int a, int b) { return abs(a) > abs(b); };
priority_queue<int, vector<int>, decltype(cmp)> pq(cmp);

// Operations
pq.push(x);       // insert — O(log n)
pq.pop();         // remove top (min or max) — O(log n); undefined behavior if empty
pq.top();         // view top without removing — O(1); undefined behavior if empty
pq.size();        // O(1)
pq.empty();       // O(1)
// contains: no direct equivalent — O(n) manual search
// remove arbitrary element: no direct equivalent — use custom structures
```

**Critical:** In C++, `std::priority_queue` is a max-heap by default (unlike Java). Use `greater<T>` for a min-heap. Never use subtraction in comparators — it overflows for extreme values like `INT_MIN`. Always use explicit comparisons.

---

## Heap Internals

| Operation | Min-Heap | Max-Heap |
|-----------|---------|---------|
| Insert | O(log n) — bubble up | O(log n) — bubble up |
| Delete top | O(log n) — bubble down | O(log n) — bubble down |
| Peek top | O(1) | O(1) |
| Build heap | O(n) — heapify all | O(n) — heapify all |
| Find kth | O(k log n) — poll k times | O(k log n) |

**Build heap is O(n), not O(n log n):** Bottom-up heapification; nodes at lower levels have minimal work; by amortized analysis total is O(n).

---

## Problem 1: Last Stone Weight — LC 1046

Repeatedly smash the two heaviest stones. If equal, both destroyed; otherwise `|y - x|` remains.

```cpp
#include <bits/stdc++.h>
using namespace std;

int lastStoneWeight(vector<int>& stones) {
    priority_queue<int> maxHeap; // max-heap by default
    for (int s : stones) maxHeap.push(s);

    while (maxHeap.size() > 1) {
        int y = maxHeap.top(); maxHeap.pop();  // largest
        int x = maxHeap.top(); maxHeap.pop();  // second largest
        if (y != x) maxHeap.push(y - x);       // remainder
    }

    return maxHeap.empty() ? 0 : maxHeap.top();
}
```

**Complexity:** O(n log n) — n stones, each operation O(log n)

**Edge cases:**
- All stones equal → all destroyed → return 0 (heap becomes empty)
- Single stone → while loop never runs → return that stone

---

## Min-Heap vs Max-Heap Selection

| Goal | Heap Type | Why |
|------|----------|-----|
| Always access the minimum | Min-heap | Root = min |
| Always access the maximum | Max-heap | Root = max |
| Find kth largest | Min-heap of size k | Root = kth largest; pop when size > k |
| Find kth smallest | Max-heap of size k | Root = kth smallest; pop when size > k |
| Median (lower half) | Max-heap | Root = middle of lower half |
| Median (upper half) | Min-heap | Root = middle of upper half |

---

## Heap with Custom Objects

```cpp
#include <bits/stdc++.h>
using namespace std;

// Sort by distance, then by index for ties (min-heap)
auto cmp = [](const vector<int>& a, const vector<int>& b) {
    if (a[0] != b[0]) return a[0] > b[0];  // primary: distance
    return a[1] > b[1];                      // tie-break: index
};
priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> pq(cmp);
pq.push({distance, index});
```

**For Top K Frequent Words** (string comparison + frequency):
```cpp
#include <bits/stdc++.h>
using namespace std;

using Entry = pair<string, int>;
auto cmp = [](const Entry& a, const Entry& b) {
    if (a.second != b.second)
        return a.second > b.second;   // fewer freq first (min-heap of freq)
    return a.first < b.first;         // more lexicographic first for same freq
};
priority_queue<Entry, vector<Entry>, decltype(cmp)> pq(cmp);
```

---

## Heapify — Building from Array

```cpp
#include <bits/stdc++.h>
using namespace std;

// C++: priority_queue constructor accepts iterators for O(n) heapify
vector<int> arr = {3, 1, 4, 1, 5, 9, 2, 6};
priority_queue<int> pq(arr.begin(), arr.end());   // O(n) heapify

// Alternatively, build via repeated inserts — O(n log n)
priority_queue<int> pq2;
for (int x : arr) pq2.push(x);
```

---

## Related Files

- [Top K Elements](./Top%20K%20Elements.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Two Heaps](./Two%20Heaps.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
