# Two Heaps

> **Topic:** [Heaps](../README.md) · **Pattern 4 of 5**
> **Problems:** Find Median from Data Stream · Sliding Window Median · IPO

---

## Core Concept

**Two-heap pattern** maintains two heaps to efficiently track the median or balance elements:
- **Max-heap (`lo`):** lower half of all elements; root = largest in lower half
- **Min-heap (`hi`):** upper half of all elements; root = smallest in upper half

**Invariants to maintain after every operation:**
1. **Size balance:** `lo.len() == hi.len()` OR `lo.len() == hi.len() + 1` (lo can have one extra)
2. **Order property:** `lo.peek() ≤ hi.peek()` (every lower-half element ≤ every upper-half element)

**Median:**
- If `lo.len() == hi.len()`: median = `(lo.peek() + hi.peek()) / 2.0`
- If `lo.len() > hi.len()`: median = `lo.peek()`

---

## Problem 1: Find Median from Data Stream — LC 295

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

struct MedianFinder {
    lo: BinaryHeap<i32>,          // max-heap (lower half)
    hi: BinaryHeap<Reverse<i32>>, // min-heap (upper half)
}

impl MedianFinder {
    fn new() -> Self {
        MedianFinder {
            lo: BinaryHeap::new(),
            hi: BinaryHeap::new(),
        }
    }

    fn add_num(&mut self, num: i32) {
        // Step 1: Route to correct heap
        if self.lo.is_empty() || num <= *self.lo.peek().unwrap() {
            self.lo.push(num);
        } else {
            self.hi.push(Reverse(num));
        }

        // Step 2: Rebalance so |lo.len() - hi.len()| <= 1
        if self.lo.len() > self.hi.len() + 1 {
            self.hi.push(Reverse(self.lo.pop().unwrap()));
        } else if self.hi.len() > self.lo.len() {
            self.lo.push(self.hi.pop().unwrap().0);
        }
    }

    fn find_median(&self) -> f64 {
        if self.lo.len() == self.hi.len() {
            (*self.lo.peek().unwrap() as f64 + self.hi.peek().unwrap().0 as f64) / 2.0
        } else {
            *self.lo.peek().unwrap() as f64  // lo always has the extra element
        }
    }
}
```

**Trace for add_num sequence [5, 3, 8, 1]:**
```
add_num(5): lo=[5], hi=[]
add_num(3): 3<=5 → lo=[5,3]. lo.len=2 > hi.len+1=1 → move to hi. lo=[5], hi=[3]
           Wait — rebalance: lo.len(2) > hi.len(0)+1=1 → hi.push(Reverse(lo.pop()=5)) → lo=[3], hi=[5]
           Actually: lo.len=2 > hi.len+1=1, rebalance → hi.push(Reverse(lo.pop())) → lo=[3], hi=[5]
           Hmm, 3 should be in lo (lower half). Let me retrace:
           add_num(3): 3 <= lo.peek()=5 → lo.push(3) → lo=[5,3]. Rebalance: lo.len=2 > hi.len(0)+1=1 → hi.push(Reverse(lo.pop()=5)); → lo=[3], hi=[5]
           find_median: lo.len=hi.len=1 → (3+5)/2=4
add_num(8): 8 > lo.peek()=3 → hi.push(Reverse(8)) → hi=[5,8]. Rebalance: hi.len=2 > lo.len=1 → lo.push(hi.pop().0=5); → lo=[5,3], hi=[8]
           find_median: lo.len=2 > hi.len=1 → lo.peek()=5
add_num(1): 1 <= lo.peek()=5 → lo.push(1) → lo=[5,3,1]. Rebalance: lo.len=3 > hi.len+1=2 → hi.push(Reverse(lo.pop()=5)); → lo=[3,1], hi=[5,8]
           find_median: lo.len=hi.len=2 → (3+5)/2=4
```

**Complexity:** O(log n) add_num, O(1) find_median, O(n) space

---

## Problem 2: Sliding Window Median — LC 480

Find the median of each window of size k as it slides.

**Challenge:** Standard two-heap doesn't support removal of arbitrary elements. Solution: lazy deletion using a `toRemove` counter map.

```rust
use std::collections::BTreeMap;

fn median_sliding_window(nums: &[i32], k: usize) -> Vec<f64> {
    // BTreeMap acts as ordered multiset with counts
    let mut lo: BTreeMap<i32, i32> = BTreeMap::new(); // lower half — use last_key_value() for max
    let mut hi: BTreeMap<i32, i32> = BTreeMap::new(); // upper half — use first_key_value() for min
    let mut lo_size: usize = 0;
    let mut hi_size: usize = 0;

    let mut result = vec![0.0f64; nums.len() - k + 1];

    for i in 0..nums.len() {
        // Add incoming element
        let lo_max = lo.keys().next_back().copied();
        if lo.is_empty() || nums[i] <= lo_max.unwrap() {
            *lo.entry(nums[i]).or_insert(0) += 1;
            lo_size += 1;
        } else {
            *hi.entry(nums[i]).or_insert(0) += 1;
            hi_size += 1;
        }

        // Rebalance
        if lo_size > hi_size + 1 {
            let top = *lo.keys().next_back().unwrap();
            *hi.entry(top).or_insert(0) += 1;
            hi_size += 1;
            let cnt = lo.get_mut(&top).unwrap();
            *cnt -= 1;
            if *cnt == 0 { lo.remove(&top); }
            lo_size -= 1;
        } else if hi_size > lo_size {
            let top = *hi.keys().next().unwrap();
            *lo.entry(top).or_insert(0) += 1;
            lo_size += 1;
            let cnt = hi.get_mut(&top).unwrap();
            *cnt -= 1;
            if *cnt == 0 { hi.remove(&top); }
            hi_size -= 1;
        }

        // Window is full starting at i == k-1
        if i >= k - 1 {
            let lo_max_val = *lo.keys().next_back().unwrap();
            let hi_min_val = *hi.keys().next().unwrap();
            result[i - k + 1] = if lo_size == hi_size {
                (lo_max_val as f64 + hi_min_val as f64) / 2.0
            } else {
                lo_max_val as f64
            };

            // Remove outgoing element (leftmost of window)
            let out = nums[i - k + 1];
            if lo.contains_key(&out) {
                let cnt = lo.get_mut(&out).unwrap();
                *cnt -= 1;
                if *cnt == 0 { lo.remove(&out); }
                lo_size -= 1;
            } else {
                let cnt = hi.get_mut(&out).unwrap();
                *cnt -= 1;
                if *cnt == 0 { hi.remove(&out); }
                hi_size -= 1;
            }

            // Rebalance after removal
            if lo_size > hi_size + 1 {
                let top = *lo.keys().next_back().unwrap();
                *hi.entry(top).or_insert(0) += 1;
                hi_size += 1;
                let cnt = lo.get_mut(&top).unwrap();
                *cnt -= 1;
                if *cnt == 0 { lo.remove(&top); }
                lo_size -= 1;
            } else if hi_size > lo_size {
                let top = *hi.keys().next().unwrap();
                *lo.entry(top).or_insert(0) += 1;
                lo_size += 1;
                let cnt = hi.get_mut(&top).unwrap();
                *cnt -= 1;
                if *cnt == 0 { hi.remove(&top); }
                hi_size -= 1;
            }
        }
    }
    result
}
```

**Why `BTreeMap` instead of `BinaryHeap`?** `BinaryHeap` does not support O(log n) arbitrary removal. `BTreeMap` with remove is O(log n). For a sliding window, O(log n) removal is essential.

---

## Problem 3: IPO — LC 502

Maximize capital after k projects. Each project has profit and capital requirement.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

fn find_maximized_capital(k: i32, mut w: i32, profits: &[i32], capital: &[i32]) -> i32 {
    let n = profits.len();
    // Min-heap by capital requirement — projects we can't afford yet
    let mut locked: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new();
    // Max-heap by profit — projects we can afford; pick most profitable
    let mut available: BinaryHeap<i32> = BinaryHeap::new();

    for i in 0..n {
        locked.push(Reverse((capital[i], profits[i])));
    }

    for _ in 0..k {
        // Unlock all projects we can now afford
        while let Some(&Reverse((cap, _))) = locked.peek() {
            if cap <= w {
                let Reverse((_, profit)) = locked.pop().unwrap();
                available.push(profit);
            } else {
                break;
            }
        }
        if available.is_empty() { break; }  // no affordable projects
        w += available.pop().unwrap();       // pick most profitable
    }
    w
}
```

**Two-heap insight:** One min-heap sorted by capital (locked projects) and one max-heap sorted by profit (available projects). This models a "greedy unlock + greedy pick" strategy.

**Complexity:** O(n log n + k log n) time, O(n) space

---

## Two Heaps — Invariant Summary

| Problem | lo (max-heap) | hi (min-heap) | Invariant |
|---------|--------------|--------------|-----------|
| Median Stream | lower half | upper half | \|lo\|-\|hi\| ≤ 1; lo.peek ≤ hi.peek |
| Sliding Window Median | lower half of window | upper half of window | Same + need removal |
| IPO | available projects (by profit) | locked projects (by capital) | Different semantic — unlock to available on each step |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [MedianFinder Design](../Design%20Data%20Structure%20Problems/Median%20Finder.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
