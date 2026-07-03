# MedianFinder

> **Topic:** [Heaps](../README.md) · **Design 1 of 2**
> **Problems:** Find Median from Data Stream (LC 295)

---

## Problem Statement

Design a data structure that supports:
- `addNum(num)` — add integer to the data structure
- `findMedian()` — return the median of all added numbers

All operations must be efficient.

---

## Solution: Two Heaps

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

struct MedianFinder {
    // lo: max-heap, holds lower half
    lo: BinaryHeap<i32>,
    // hi: min-heap, holds upper half
    hi: BinaryHeap<Reverse<i32>>,
}

impl MedianFinder {
    fn new() -> Self {
        MedianFinder {
            lo: BinaryHeap::new(),
            hi: BinaryHeap::new(),
        }
    }

    fn add_num(&mut self, num: i32) {
        // 1. Route to correct half
        if self.lo.is_empty() || num <= *self.lo.peek().unwrap() {
            self.lo.push(num);
        } else {
            self.hi.push(Reverse(num));
        }

        // 2. Rebalance: lo can have at most 1 more than hi
        if self.lo.len() > self.hi.len() + 1 {
            let top = self.lo.pop().unwrap();
            self.hi.push(Reverse(top));
        } else if self.hi.len() > self.lo.len() {
            let Reverse(top) = self.hi.pop().unwrap();
            self.lo.push(top);
        }
    }

    fn find_median(&self) -> f64 {
        if self.lo.len() > self.hi.len() {
            *self.lo.peek().unwrap() as f64
        } else {
            let lo_top = *self.lo.peek().unwrap() as f64;
            let Reverse(hi_top) = *self.hi.peek().unwrap();
            (lo_top + hi_top as f64) / 2.0
        }
    }
}
```

**Complexity:** `add_num` O(log n), `find_median` O(1), space O(n)

---

## Dry Run: [1, 2, 3, 4, 5]

```
add_num(1): lo=[1], hi=[]              → median = 1.0
add_num(2): 2>1 → hi.push(2). hi.len>lo.len → lo.push(hi.pop()=2) → lo=[2,1], hi=[]
           Wait: 2 > lo.peek().unwrap()=1 → hi.push(2) → lo=[1], hi=[2]. lo.len=hi.len=1 → OK
           find_median: lo.len=hi.len=1 → (1+2)/2=1.5
add_num(3): 3 > lo.peek().unwrap()=1 → hi.push(3) → lo=[1], hi=[2,3]. hi.len>lo.len → lo.push(hi.pop()=2) → lo=[2,1], hi=[3]
           find_median: lo.len=2>hi.len=1 → lo.peek().unwrap()=2
add_num(4): 4>2 → hi.push(4) → lo=[2,1], hi=[3,4]. lo.len=hi.len=2 → OK
           find_median: (2+3)/2=2.5
add_num(5): 5>2 → hi.push(5) → lo=[2,1], hi=[3,4,5]. hi.len=3>lo.len=2 → lo.push(hi.pop()=3) → lo=[3,2,1], hi=[4,5]
           find_median: lo.len=3>hi.len=2 → lo.peek().unwrap()=3
```

---

## Follow-up 1: Handle Large Even Counts — Integer Overflow in Median

```rust
fn find_median(&self) -> f64 {
    if self.lo.len() > self.hi.len() {
        return *self.lo.peek().unwrap() as f64;
    }
    let Reverse(hi_top) = *self.hi.peek().unwrap();
    *self.lo.peek().unwrap() as f64 / 2.0 + hi_top as f64 / 2.0  // avoids overflow vs (a+b)/2
}
```

---

## Follow-up 2: Find Median of Integer Stream with Constraint

**"99% of all numbers are in range [0, 100]"** — Use a bucket array for common values, heap for outliers.

```rust
// Optimization: O(1) amortized via bucket counts for [0..100]
// Heap only for values outside that range
// find_median: walk bucket array from 0, counting until median position
```

---

## Follow-up 3: Thread Safety

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;
use std::sync::RwLock;

struct MedianFinderInner {
    lo: BinaryHeap<i32>,
    hi: BinaryHeap<Reverse<i32>>,
}

struct ThreadSafeMedianFinder {
    inner: RwLock<MedianFinderInner>,
}

impl ThreadSafeMedianFinder {
    fn new() -> Self {
        ThreadSafeMedianFinder {
            inner: RwLock::new(MedianFinderInner {
                lo: BinaryHeap::new(),
                hi: BinaryHeap::new(),
            }),
        }
    }

    fn add_num(&self, num: i32) {
        let mut data = self.inner.write().unwrap();
        if data.lo.is_empty() || num <= *data.lo.peek().unwrap() {
            data.lo.push(num);
        } else {
            data.hi.push(Reverse(num));
        }
        if data.lo.len() > data.hi.len() + 1 {
            let top = data.lo.pop().unwrap();
            data.hi.push(Reverse(top));
        } else if data.hi.len() > data.lo.len() {
            let Reverse(top) = data.hi.pop().unwrap();
            data.lo.push(top);
        }
    }

    fn find_median(&self) -> f64 {
        let data = self.inner.read().unwrap();
        if data.lo.len() > data.hi.len() {
            *data.lo.peek().unwrap() as f64
        } else {
            let lo_top = *data.lo.peek().unwrap() as f64;
            let Reverse(hi_top) = *data.hi.peek().unwrap();
            (lo_top + hi_top as f64) / 2.0
        }
    }
}
```

**`RwLock`:** Multiple concurrent `find_median` readers allowed (`read()`); only one `add_num` writer at a time (`write()`).

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Which heap holds extra element? | `lo` (max-heap) | `lo.peek().unwrap()` is the median for odd count |
| How to route new element? | Route to `lo` first if ≤ lo.peek().unwrap(), else to `hi` | Maintains order property |
| Rebalance direction? | Balance so `lo.len() ≤ hi.len() + 1` | `lo` is allowed 1 extra |
| Integer arithmetic in find_median? | Use `as f64` and `/2.0` division | Avoids integer truncation |

---

## Related Files

- [Two Heaps Pattern](../Patterns/Two%20Heaps.md)
- [Twitter Feed Design](./Twitter%20Feed.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
