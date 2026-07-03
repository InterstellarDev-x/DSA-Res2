# Google — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Google

---

## Problem 1: Find Median from Data Stream — Deep Dive

**LC 295** · Hard

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

struct MedianFinder {
    lo: BinaryHeap<i32>,          // max-heap
    hi: BinaryHeap<Reverse<i32>>, // min-heap
}

impl MedianFinder {
    fn new() -> Self {
        MedianFinder {
            lo: BinaryHeap::new(),
            hi: BinaryHeap::new(),
        }
    }

    fn add_num(&mut self, num: i32) {
        if self.lo.is_empty() || num <= *self.lo.peek().unwrap() {
            self.lo.push(num);
        } else {
            self.hi.push(Reverse(num));
        }
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

**Q: What if the stream only contains integers in [0, 100]?**
A: Use a frequency array of size 101. Track total count. For `find_median`, walk the array finding the position(s) of the median. O(1) add_num, O(100) = O(1) find_median, but truly O(1) space (not O(n)).

**Q: What if 99% of numbers are in [0, 100] with occasional outliers?**
A: Hybrid approach: bucket array for [0..100], two small heaps for outliers. Median calculation walks the appropriate structure based on total count.

**Q: What is the invariant that guarantees correctness?**
A: Two invariants: (1) Size: `|lo.len() - hi.len()| ≤ 1`. (2) Order: `lo.peek() ≤ hi.peek()`. If both hold, the median is either `lo.peek()` (odd total) or `(lo.peek() + hi.peek()) / 2` (even total).

---

## Problem 2: IPO — Deep Dive

**LC 502** · Hard

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn find_maximized_capital(k: i32, mut w: i32, profits: Vec<i32>, capital: Vec<i32>) -> i32 {
    let n = profits.len();
    // min-heap by capital: (capital, profit)
    let mut locked: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new();
    // max-heap by profit: (profit, capital)
    let mut available: BinaryHeap<(i32, i32)> = BinaryHeap::new();

    for i in 0..n {
        locked.push(Reverse((capital[i], profits[i])));
    }

    for _ in 0..k {
        while let Some(&Reverse((cap, prof))) = locked.peek() {
            if cap <= w {
                locked.pop();
                available.push((prof, cap));
            } else {
                break;
            }
        }
        if available.is_empty() {
            break;
        }
        w += available.pop().unwrap().0;
    }
    w
}
```

**Q: Prove the greedy choice is optimal.**
A: At each step, we can afford any project with `capital ≤ w`. Among these, picking the one with the highest profit maximizes our capital gain — any other choice leads to equal or lower capital for all future steps. This is a classic exchange argument: swapping any non-maximum profit project with the maximum always improves or maintains the result.

**Q: Google follow-up — what if we have a budget (total capital we can spend)?**
A: Modify: track total spent capital separately, and `available` heap now also checks if we can afford the cost (if projects have both profit AND cost).

**Q: Google follow-up — what if k is very large?**
A: Once `available` is empty (can't afford anything more), break early. With `k = n`, we might complete all projects.

---

## Problem 3: Smallest Range from K Lists — Deep Dive

**LC 632** · Hard

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn smallest_range(nums: Vec<Vec<i32>>) -> Vec<i32> {
    // min-heap: (value, list_idx, elem_idx)
    let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
    let mut max_val = i32::MIN;

    for i in 0..nums.len() {
        heap.push(Reverse((nums[i][0], i, 0)));
        max_val = max_val.max(nums[i][0]);
    }

    let Reverse((init_val, _, _)) = *heap.peek().unwrap();
    let mut result = vec![init_val, max_val];

    loop {
        let Reverse((val, list_idx, elem_idx)) = heap.pop().unwrap();
        let _ = val; // used via heap ordering
        if elem_idx + 1 == nums[list_idx].len() {
            break;
        }

        let next = nums[list_idx][elem_idx + 1];
        heap.push(Reverse((next, list_idx, elem_idx + 1)));
        max_val = max_val.max(next);
        let Reverse((heap_min, _, _)) = *heap.peek().unwrap();
        if max_val - heap_min < result[1] - result[0] {
            result[0] = heap_min;
            result[1] = max_val;
        }
    }
    result
}
```

**Q: Why does `max_val` only increase?**
A: We only add elements from lists, and each list is sorted ascending. When we advance to the next element of a list, the new value is ≥ the old value. So `max_val` is non-decreasing. The minimum of the heap (= `heap.peek()`) is the range's lower bound; we minimize the range by advancing the minimum.

**Q: Why stop when a list is exhausted?**
A: We need exactly one element from each list. If list `i` is exhausted, the range's minimum is now determined by the second-smallest across lists. We can't include list `i` anymore — there's no valid range.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | Kth Largest, Top K Frequent, K Closest |
| L4 | Find Median, IPO, K Pairs |
| L5+ | Sliding Window Median, Smallest Range, Design Twitter |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [Two Heaps Pattern](../Patterns/Two%20Heaps.md)
- [K-Way Merge Pattern](../Patterns/K%20Way%20Merge.md)

> **Last Updated:** 2026-06-26
