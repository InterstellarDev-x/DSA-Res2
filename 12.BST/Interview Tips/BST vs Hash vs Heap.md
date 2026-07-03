> **Topic:** [Binary Search Trees](../README.md) · **Tips 3 of 4**

# BST vs Hash vs Heap — Choosing the Right Structure

Interviewers often probe *why* you picked a structure. The deciding question is **what kind of
ordering or query** you need.

---

## Decision Table

| You need… | Use | Why | C++ |
|---|---|---|---|
| **Ordered** iteration, range queries, floor/ceil, successor/predecessor | **BST** (balanced) | Keeps keys sorted; all order queries in O(log n) | `std::map`, `std::set` |
| **O(1)** average lookup/insert/delete by exact key; no ordering | **Hash** | Buckets, no ordering overhead | `std::unordered_map`, `std::unordered_set` |
| Repeated **min** (or **max**) extraction from a stream; top-k | **Heap** | O(1) peek, O(log n) push/pop of the extreme only | `std::priority_queue` |
| Insertion-order or LRU iteration | Linked Hash | Hash + linked list | `std::list + std::unordered_map` |

Key contrasts:
- A **heap** gives you the single extreme cheaply but **cannot** answer "is key x present?",
  "what's the key just below x?", or "iterate in sorted order" efficiently. A BST can.
- A **hash** map gives O(1) point lookups but **loses all ordering** — no range, floor, ceil, or
  sorted scan. A BST trades a log factor for those.
- A **BST** (self-balancing) is the only one that does *both* ordered queries *and* logarithmic
  point operations.

---

## C++ `std::map` / `std::set` — the production BST

`std::map` and `std::set` are typically Red-Black trees, so every operation below is **O(log n)** guaranteed.

| Method | Returns |
|---|---|
| `prev(map.upper_bound(k))->first` | greatest key **≤ k** (else `end()`) |
| `map.lower_bound(k)->first` | smallest key **≥ k** (else `end()`) |
| `prev(map.lower_bound(k))->first` | greatest key **< k** (strict) |
| `map.upper_bound(k)->first` | smallest key **> k** (strict) |
| `map.begin()->first` / `prev(map.end())->first` | min / max key |
| `lower_bound(from)` to `lower_bound(to)` | a sorted view over a key range |
| `begin()` to `lower_bound(k)` / `lower_bound(k)` to `end()` | keys `< k` / `≥ k` |

```cpp
#include <bits/stdc++.h>
using namespace std;

map<int, string> m;
m[10] = "a"; m[20] = "b"; m[30] = "c";

prev(m.upper_bound(25))->first;    // 20  (largest key <= 25)   <- maps to BST "floor"
m.lower_bound(25)->first;          // 30  (smallest key >= 25)  <- maps to BST "ceil"
prev(m.lower_bound(20))->first;    // 10  (strictly < 20)
m.upper_bound(20)->first;          // 30  (strictly > 20)
m.begin()->first;                  // 10
prev(m.end())->first;              // 30

// range query [15, 30) — like Range Sum of BST, in O(log n + count)
for (auto it = m.lower_bound(15); it != m.lower_bound(30); ++it) { /* it->first == 20 */ }

// set<int> for set semantics
set<int> s = {1, 3, 5, 7};
*prev(s.upper_bound(4));     // 3   (floor)
*s.lower_bound(4);           // 5   (ceiling)
// headSet(5): elements < 5 → {1, 3}
for (auto it = s.begin(); it != s.lower_bound(5); ++it) { /* 1, 3 */ }
```

These methods are *exactly* the hand-rolled `floor` / `ceil` / `successor` from
[BST LCA & Ancestors](../Patterns/BST%20LCA%20and%20Ancestors.md) — reach for `std::map` in
production unless the interviewer wants the manual implementation.

---

## When the interviewer says "make lookups faster"

If you used a BST only for point lookups and the prompt has **no ordering requirement**, switch to
an `std::unordered_map` for O(1). But if any of "range", "next greater", "k-th", "sorted output", "floor/ceil"
appear, the BST / `std::map` is correct and a hash map would force an O(n log n) re-sort.

> **Last Updated:** 2026-06-26
