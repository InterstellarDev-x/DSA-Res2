# Design: Sparse Table (Range Minimum/Maximum Query)

> **Topic:** [Arrays](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `sparse-table` `RMQ` `range-query` `O(1)-query`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Approaches](#approaches)
4. [C++ Implementation](#cpp-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design a data structure for a **static array** that answers Range Minimum (or Maximum) Queries in O(1) after O(n log n) preprocessing.

- `SparseTable(int[] arr)` — preprocess the array
- `int queryMin(int l, int r)` — return minimum in `arr[l..r]` in O(1)

The array is **immutable** after construction (no updates).

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core concept | Precompute min over all power-of-2 length intervals |
| Key insight | Any range [l, r] can be covered by two overlapping power-of-2 windows |
| When to use vs Segment Tree | Sparse Table = O(1) query, no updates; Segment Tree = O(log n) query + updates |

---

## Approaches

### Approach 1: Naive (No preprocessing)

Scan `[l, r]` for every query.
- Build: O(1), Query: O(n), Space: O(1)

### Approach 2: Prefix Min Array

Only works for prefix queries, not arbitrary ranges.

### Approach 3: Sparse Table (Optimal for static arrays)

Precompute `table[i][j]` = minimum of `arr[i .. i + 2^j - 1]`.

For query `[l, r]`:
- Let `k = floor(log2(r - l + 1))`
- Answer = `min(table[l][k], table[r - 2^k + 1][k])`

The two windows overlap, but since we're taking minimum (idempotent), overlap is fine.

---

## C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

class SparseTable {
    vector<vector<int>> table;
    vector<int> log2;
    int n;

public:
    SparseTable(vector<int>& arr) {
        n = arr.size();
        int maxLog = (int)(log2f(n)) + 1;
        table.assign(n, vector<int>(maxLog));
        log2.resize(n + 1);

        // Precompute log2 floor values
        log2[1] = 0;
        for (int i = 2; i <= n; i++) {
            log2[i] = log2[i / 2] + 1;
        }

        // Base case: intervals of length 1
        for (int i = 0; i < n; i++) table[i][0] = arr[i];

        // Fill for lengths 2, 4, 8, ...
        for (int j = 1; (1 << j) <= n; j++) {
            for (int i = 0; i + (1 << j) - 1 < n; i++) {
                table[i][j] = min(table[i][j - 1],
                                  table[i + (1 << (j - 1))][j - 1]);
            }
        }
    }

    // O(1) query: minimum of arr[l..r] (inclusive, 0-indexed)
    int queryMin(int l, int r) {
        int k = log2[r - l + 1];
        return min(table[l][k], table[r - (1 << k) + 1][k]);
    }
};
```

### Maximum Query Variant

```cpp
// Replace min with max — works because max is also idempotent
int queryMax(int l, int r) {
    int k = log2[r - l + 1];
    return max(table[l][k], table[r - (1 << k) + 1][k]);
}
```

> **Note:** Sparse Table only works for **idempotent** operations (min, max, GCD, AND, OR). It does **NOT** work for sum queries (overlap would double-count) — use Prefix Sum or Segment Tree for sum.

---

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Preprocessing | O(n log n) | O(n log n) |
| Query | O(1) | — |
| Update (not supported) | — | — |

---

## Edge Cases

- Single element: `queryMin(i, i)` = `arr[i]`
- Entire array: `queryMin(0, n-1)` — ensure `k = log2[n]` is computed
- `l > r` → throw `IllegalArgumentException`
- Array of size 1 → `maxLog = 1`, `table[0][0] = arr[0]`
- Integer overflow in index arithmetic for large n → use `long` if n > 10^6

---

## Similar Problems

| Problem | Data Structure |
|---------|----------------|
| [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | Prefix Sum |
| [Range Minimum Query](https://leetcode.com/problems/range-minimum-query/) | Sparse Table |
| [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Monotonic Deque |
| Range updates + queries | Segment Tree / BIT |

---

## Follow-up Questions

1. **Why can't Sparse Table do sum queries?** — Overlapping windows double-count elements. Sum is not idempotent.
2. **Compare with Segment Tree** — Segment Tree: O(log n) build, O(log n) query, supports updates. Sparse Table: O(n log n) build, O(1) query, no updates.
3. **What if we need range GCD?** — GCD is idempotent (gcd(x, x) = x), so Sparse Table works.
4. **Memory optimization?** — Use 1D rolling array if only answering one query level at a time.

---

## Company Tags

`Google` `Amazon` `Codeforces Competitive` `ICPC`

---

## Navigation

| Related |
|---------|
| [Arrays README](../README.md) |
| [Prefix Sum Pattern](../Patterns/Prefix%20Sum.md) |

> **Last Updated:** 2026-06-26
