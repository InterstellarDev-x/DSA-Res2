# Design: Order-Statistics Structure (Kth Element)

> **Topic:** [Binary Search](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Hard · **Tags:** `order-statistics` `kth-element` `fenwick-tree` `segment-tree`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Approaches](#approaches)
4. [Rust Implementation](#rust-implementation)
5. [Complexity Analysis](#complexity-analysis)
6. [Edge Cases](#edge-cases)
7. [Similar Problems](#similar-problems)
8. [Follow-up Questions](#follow-up-questions)
9. [Company Tags](#company-tags)

---

## Problem Statement

Design a data structure that supports:

- `void add(int val)` — add a value to the collection
- `int kthSmallest(int k)` — return the kth smallest value (1-indexed)
- `int rank(int val)` — return how many elements are ≤ val

All operations should be efficient on a dynamic, online stream of integers.

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Multiple approaches | Sorted list O(n), BST O(log n), BIT O(log M) |
| Know trade-offs | Space vs time, static vs dynamic |
| BIT / Fenwick Tree for rank queries | Prefix sum on frequency array |
| Coordinate compression | When values can be very large |

---

## Approaches

| Approach | Add | kth Smallest | Space | Notes |
|----------|-----|-------------|-------|-------|
| Sorted Vec + binary search | O(n) | O(log n) | O(n) | Simple, slow add |
| Augmented BST | O(log n) | O(log n) | O(n) | See [BST Design](./BST%20Design.md) |
| Binary Indexed Tree (BIT) | O(log M) | O(log² M) | O(M) | M = value range; fastest for dense ranges |
| Segment Tree | O(log M) | O(log M) | O(M) | Better kth than BIT |
| Two Heaps (median only) | O(log n) | O(1) for median | O(n) | Only works for median |

---

## Rust Implementation

### Approach 1: BIT (Fenwick Tree) — Best for value range [1, MAX]

```rust
struct OrderStatisticsBIT {
    max: usize,
    bit: Vec<i32>,
}

impl OrderStatisticsBIT {
    fn new() -> Self {
        let max = 100001usize;
        OrderStatisticsBIT {
            max,
            bit: vec![0; 100003],
        }
    }

    // BIT point update: add 1 at position val
    fn add(&mut self, val: usize) {
        let mut i = val;
        while i <= self.max {
            self.bit[i] += 1;
            i += i & i.wrapping_neg();
        }
    }

    // BIT prefix sum: count of elements in [1, val]
    fn rank(&self, val: usize) -> i32 {
        let mut count = 0;
        let mut i = val;
        while i > 0 {
            count += self.bit[i];
            i -= i & i.wrapping_neg();
        }
        count
    }

    // kth smallest via binary search on BIT
    fn kth_smallest(&self, k: i32) -> usize {
        let mut lo = 1usize;
        let mut hi = self.max;
        while lo < hi {
            let mid = lo + (hi - lo) / 2;
            if self.rank(mid) < k {
                lo = mid + 1;
            } else {
                hi = mid;
            }
        }
        lo
    }
    // Time: add O(log M), rank O(log M), kth O(log² M)
}
```

### Approach 2: Optimised BIT kth (O(log M) descent)

```rust
fn kth_smallest_fast(bit: &[i32], max: usize, mut k: i32) -> usize {
    let mut pos = 0usize;
    // find highest power of 2 <= max (equivalent to 1 << (31 - leading_zeros(MAX)))
    let mut pw = 1usize << (31 - (max as u32).leading_zeros() as usize);
    while pw > 0 {
        if pos + pw <= max && bit[pos + pw] < k {
            pos += pw;
            k -= bit[pos];
        }
        pw >>= 1;
    }
    pos + 1
}
// Time: O(log M) | descends the implicit BIT tree directly
```

### Approach 3: Augmented BST (dynamic range, any values)

```rust
// See BST Design.md — add size augmentation
// add: O(log n) amortized (balanced); kth_smallest: O(log n)
```

### Approach 4: Coordinate Compression + BIT (for large/negative values)

```rust
// When values can be from -10^9 to 10^9:
// 1. Collect all possible values offline
// 2. Compress to [1, n]
// 3. Use BIT on compressed indices
// Not possible for online (streaming) queries
```

---

## Complexity Analysis

| Operation | BIT | Augmented BST | Segment Tree |
|-----------|-----|---------------|-------------|
| `add` | O(log M) | O(log n) | O(log M) |
| `rank` | O(log M) | O(log n) | O(log M) |
| `kthSmallest` | O(log² M) basic / O(log M) optimised | O(log n) | O(log M) |
| Space | O(M) | O(n) | O(M) |

---

## Edge Cases

- k > total elements added → throw exception or return -1
- Duplicate values → BIT handles naturally (counts multiple)
- Negative values → shift by offset before using BIT
- Very large values (up to 10^9) → coordinate compression or segment tree with dynamic nodes

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | LC 703 |
| [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | LC 295 |
| [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | LC 315 |
| [Kth Smallest Element in BST](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | LC 230 |

---

## Follow-up Questions

1. **What if values can be negative?** → Shift all values by offset (e.g., +10^9) before using BIT
2. **What if the value range is huge (10^9)?** → Use augmented BST (space proportional to n) or dynamic segment tree
3. **How does Rust's `BTreeMap` handle this?** → `.range(..=val).count()` for rank, but O(n) for kth unless augmented
4. **Two-heap approach for median?** → Maintains `BinaryHeap<i32>` (lower half, max-heap) and `BinaryHeap<Reverse<i32>>` (upper half, min-heap); median in O(1); kth only in O(k log n)

---

## Company Tags

`Google` `Amazon` `Microsoft` `Palantir` `Two Sigma`

---

## Navigation

| Related |
|---------|
| [Binary Search README](../README.md) |
| [BST Design](./BST%20Design.md) |
| [Binary Search on Answer Pattern](../Patterns/Binary%20Search%20on%20Answer.md) |

> **Last Updated:** 2026-06-26
