> **Topic:** [Binary Search Trees](../README.md) · **Tips 3 of 4**

# BST vs Hash vs Heap — Choosing the Right Structure

Interviewers often probe *why* you picked a structure. The deciding question is **what kind of
ordering or query** you need.

---

## Decision Table

| You need… | Use | Why | Java |
|---|---|---|---|
| **Ordered** iteration, range queries, floor/ceil, successor/predecessor | **BST** (balanced) | Keeps keys sorted; all order queries in O(log n) | `TreeMap`, `TreeSet` |
| **O(1)** average lookup/insert/delete by exact key; no ordering | **Hash** | Buckets, no ordering overhead | `HashMap`, `HashSet` |
| Repeated **min** (or **max**) extraction from a stream; top-k | **Heap** | O(1) peek, O(log n) push/pop of the extreme only | `PriorityQueue` |
| Insertion-order or LRU iteration | Linked Hash | Hash + linked list | `LinkedHashMap` |

Key contrasts:
- A **heap** gives you the single extreme cheaply but **cannot** answer "is key x present?",
  "what's the key just below x?", or "iterate in sorted order" efficiently. A BST can.
- A **hash** map gives O(1) point lookups but **loses all ordering** — no range, floor, ceil, or
  sorted scan. A BST trades a log factor for those.
- A **BST** (self-balancing) is the only one that does *both* ordered queries *and* logarithmic
  point operations.

---

## Java `TreeMap` / `TreeSet` — the production BST

`TreeMap` and `TreeSet` are Red-Black trees, so every operation below is **O(log n)** guaranteed.

| Method | Returns |
|---|---|
| `floorKey(k)` | greatest key **≤ k** (else `null`) |
| `ceilingKey(k)` | smallest key **≥ k** (else `null`) |
| `lowerKey(k)` | greatest key **< k** (strict) |
| `higherKey(k)` | smallest key **> k** (strict) |
| `firstKey()` / `lastKey()` | min / max key |
| `subMap(from, to)` | a sorted view over a key range |
| `headMap(k)` / `tailMap(k)` | keys `< k` / `≥ k` |

```java
TreeMap<Integer, String> map = new TreeMap<>();
map.put(10, "a"); map.put(20, "b"); map.put(30, "c");

map.floorKey(25);    // 20  (largest key <= 25)   ← maps to BST "floor"
map.ceilingKey(25);  // 30  (smallest key >= 25)  ← maps to BST "ceil"
map.lowerKey(20);    // 10  (strictly < 20)
map.higherKey(20);   // 30  (strictly > 20)
map.firstKey();      // 10
map.lastKey();       // 30

// range query [15, 30) — like Range Sum of BST, in O(log n + count)
for (Integer k : map.subMap(15, 30).keySet()) { /* 20 */ }

// TreeSet for set semantics
TreeSet<Integer> set = new TreeSet<>(List.of(1, 3, 5, 7));
set.floor(4);    // 3
set.ceiling(4);  // 5
set.headSet(5);  // [1, 3]
```

These methods are *exactly* the hand-rolled `floor` / `ceil` / `successor` from
[BST LCA & Ancestors](../Patterns/BST%20LCA%20and%20Ancestors.md) — reach for `TreeMap` in
production unless the interviewer wants the manual implementation.

---

## When the interviewer says "make lookups faster"

If you used a BST only for point lookups and the prompt has **no ordering requirement**, switch to
a `HashMap` for O(1). But if any of "range", "next greater", "k-th", "sorted output", "floor/ceil"
appear, the BST / `TreeMap` is correct and a hash map would force an O(n log n) re-sort.

> **Last Updated:** 2026-06-26
