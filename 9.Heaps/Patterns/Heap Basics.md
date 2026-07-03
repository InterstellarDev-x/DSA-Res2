# Heap Basics

> **Topic:** [Heaps](../README.md) · **Pattern 1 of 5**
> **Problems:** Last Stone Weight · heap fundamentals

---

## Core Concept

A **heap** is a complete binary tree satisfying the heap property:
- **Min-heap:** parent ≤ children → root = minimum element
- **Max-heap:** parent ≥ children → root = maximum element

Rust's `BinaryHeap` is a **max-heap** by default. To get a min-heap, use `Reverse<T>` as a wrapper.

---

## Rust BinaryHeap API

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// Min-heap (wrap values in Reverse)
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();

// Max-heap (default)
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new();

// Custom comparator — sort by absolute value (min-heap by abs value)
// BinaryHeap doesn't support custom comparators directly; use a newtype wrapper.
#[derive(Eq, PartialEq)]
struct ByAbsMin(i32);
impl Ord for ByAbsMin {
    fn cmp(&self, other: &Self) -> std::cmp::Ordering {
        // Reverse so smaller abs value = higher priority (min-heap by abs)
        other.0.abs().cmp(&self.0.abs())
    }
}
impl PartialOrd for ByAbsMin {
    fn partial_cmp(&self, other: &Self) -> Option<std::cmp::Ordering> {
        Some(self.cmp(other))
    }
}
let mut pq: BinaryHeap<ByAbsMin> = BinaryHeap::new();

// Operations
pq.push(ByAbsMin(x));   // insert — O(log n)
pq.pop();               // remove top (min or max) — O(log n); returns None if empty
pq.peek();              // view top without removing — O(1); returns None if empty
pq.len();               // O(1)
pq.is_empty();          // O(1)
// contains: no direct equivalent — O(n) manual search
// remove arbitrary element: no direct equivalent — use custom structures
```

**Critical:** In Rust, `BinaryHeap` is a max-heap by default (unlike Java's `PriorityQueue`). Use `Reverse<T>` for a min-heap. Never use subtraction in comparators — it overflows for extreme values like `i32::MIN`. Always use explicit comparisons.

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

```rust
use std::collections::BinaryHeap;

fn last_stone_weight(stones: Vec<i32>) -> i32 {
    let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // max-heap by default
    for s in stones { max_heap.push(s); }

    while max_heap.len() > 1 {
        let y = max_heap.pop().unwrap(); // largest
        let x = max_heap.pop().unwrap(); // second largest
        if y != x { max_heap.push(y - x); } // remainder
    }

    max_heap.peek().copied().unwrap_or(0)
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

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// Sort by distance, then by index for ties (min-heap)
// Use Reverse<(distance, index)>: tuples compare lexicographically, Reverse inverts order
let mut pq: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new();
pq.push(Reverse((distance, index)));
// Access: if let Some(Reverse((dist, idx))) = pq.pop() { ... }
```

**For Top K Frequent Words** (string comparison + frequency):
```rust
use std::collections::BinaryHeap;
use std::cmp::Ordering;

// Min-heap by frequency, then max by lexicographic order for same freq
#[derive(Eq, PartialEq)]
struct Entry {
    word: String,
    freq: i32,
}

impl Ord for Entry {
    fn cmp(&self, other: &Self) -> Ordering {
        if self.freq != other.freq {
            self.freq.cmp(&other.freq) // fewer freq first (min-heap of freq)
        } else {
            other.word.cmp(&self.word) // more lexicographic first for same freq
        }
    }
}

impl PartialOrd for Entry {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> {
        Some(self.cmp(other))
    }
}

let mut pq: BinaryHeap<Entry> = BinaryHeap::new();
```

---

## Heapify — Building from Array

```rust
use std::collections::BinaryHeap;

// Rust: BinaryHeap::from() accepts a Vec for O(n) heapify
let arr = vec![3, 1, 4, 1, 5, 9, 2, 6];
let mut pq: BinaryHeap<i32> = BinaryHeap::from(arr.clone()); // O(n) heapify

// Alternatively, build via repeated inserts — O(n log n)
let mut pq2: BinaryHeap<i32> = BinaryHeap::new();
for x in &arr { pq2.push(*x); }
```

---

## Related Files

- [Top K Elements](./Top%20K%20Elements.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Two Heaps](./Two%20Heaps.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
