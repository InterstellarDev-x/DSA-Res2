> **Topic:** [Binary Search Trees](../README.md) · **Tips 3 of 4**

# BST vs Hash vs Heap — Choosing the Right Structure

Interviewers often probe *why* you picked a structure. The deciding question is **what kind of
ordering or query** you need.

---

## Decision Table

| You need… | Use | Why | Rust |
|---|---|---|---|
| **Ordered** iteration, range queries, floor/ceil, successor/predecessor | **BST** (balanced) | Keeps keys sorted; all order queries in O(log n) | `BTreeMap`, `BTreeSet` |
| **O(1)** average lookup/insert/delete by exact key; no ordering | **Hash** | Buckets, no ordering overhead | `HashMap`, `HashSet` |
| Repeated **min** (or **max**) extraction from a stream; top-k | **Heap** | O(1) peek, O(log n) push/pop of the extreme only | `BinaryHeap` |
| Insertion-order or LRU iteration | Linked Hash | Hash + linked list | `LinkedList + HashMap` |

Key contrasts:
- A **heap** gives you the single extreme cheaply but **cannot** answer "is key x present?",
  "what's the key just below x?", or "iterate in sorted order" efficiently. A BST can.
- A **hash** map gives O(1) point lookups but **loses all ordering** — no range, floor, ceil, or
  sorted scan. A BST trades a log factor for those.
- A **BST** (self-balancing) is the only one that does *both* ordered queries *and* logarithmic
  point operations.

---

## Rust `BTreeMap` / `BTreeSet` — the production BST

`BTreeMap` and `BTreeSet` are B-Trees in Rust's standard library, so every operation below is **O(log n)** guaranteed.

| Method | Returns |
|---|---|
| `m.range(..=k).next_back()` | greatest key **≤ k** (else `None`) |
| `m.range(k..).next()` | smallest key **≥ k** (else `None`) |
| `m.range(..k).next_back()` | greatest key **< k** (strict) |
| `m.range((Bound::Excluded(k), Bound::Unbounded)).next()` | smallest key **> k** (strict) |
| `m.keys().next()` / `m.keys().next_back()` | min / max key |
| `m.range(from..to)` | a sorted view over a key range |
| `m.range(..k)` / `m.range(k..)` | keys `< k` / `≥ k` |

```rust
use std::collections::{BTreeMap, BTreeSet};
use std::ops::Bound;

let mut m: BTreeMap<i32, String> = BTreeMap::new();
m.insert(10, "a".to_string()); m.insert(20, "b".to_string()); m.insert(30, "c".to_string());

m.range(..=25).next_back().map(|(k, _)| *k);  // Some(20)  (largest key <= 25)   <- maps to BST "floor"
m.range(25..).next().map(|(k, _)| *k);         // Some(30)  (smallest key >= 25)  <- maps to BST "ceil"
m.range(..20).next_back().map(|(k, _)| *k);    // Some(10)  (strictly < 20)
m.range((Bound::Excluded(20), Bound::Unbounded)).next().map(|(k, _)| *k);  // Some(30)  (strictly > 20)
m.keys().next();       // Some(10)
m.keys().next_back();  // Some(30)

// range query [15, 30) — like Range Sum of BST, in O(log n + count)
for (k, _v) in m.range(15..30) { /* k == 20 */ }

// BTreeSet<i32> for set semantics
let s: BTreeSet<i32> = [1, 3, 5, 7].iter().cloned().collect();
s.range(..=4).next_back();  // Some(3)  (floor)
s.range(4..).next();         // Some(5)  (ceiling)
// headSet(5): elements < 5 → {1, 3}
for x in s.range(..5) { /* 1, 3 */ }
```

These methods are *exactly* the hand-rolled `floor` / `ceil` / `successor` from
[BST LCA & Ancestors](../Patterns/BST%20LCA%20and%20Ancestors.md) — reach for `BTreeMap` in
production unless the interviewer wants the manual implementation.

---

## When the interviewer says "make lookups faster"

If you used a BST only for point lookups and the prompt has **no ordering requirement**, switch to
a `HashMap` for O(1). But if any of "range", "next greater", "k-th", "sorted output", "floor/ceil"
appear, the BST / `BTreeMap` is correct and a hash map would force an O(n log n) re-sort.

> **Last Updated:** 2026-06-26
