> **Topic:** [Greedy Algorithms](../README.md) · **Design Problem 1 of 1**

# Greedy Iterator — Sorted Interval Stream

## Problem Statement
Design a data structure that supports a stream of intervals. It must:
- `void addInterval(int start, int end)` — Insert interval in O(log n), merging with any overlapping existing intervals
- `Vec<(i32, i32)> getIntervals()` — Return all intervals in sorted order by start time in O(n)

**Example:**
```
addInterval(1, 3)  → intervals: [[1,3]]
addInterval(6, 9)  → intervals: [[1,3],[6,9]]
addInterval(2, 5)  → intervals: [[1,5],[6,9]]  (merged with [1,3])
addInterval(8,10)  → intervals: [[1,5],[6,10]] (merged with [6,9])
```

## Core Idea: BTreeMap with Merge-on-Insert
Use `BTreeMap<i32, i32>` mapping `start → end`. On insert:
1. Find left neighbor (largest key ≤ new start) — merge if it overlaps
2. Find right neighbors (all keys between new start and new end) — merge all of them
3. Put merged interval

## Complete Rust Implementation

```rust
use std::collections::BTreeMap;
use std::ops::Bound;

struct GreedyIntervalIterator {
    // Key: interval start, Value: interval end
    mp: BTreeMap<i32, i32>,
}

impl GreedyIntervalIterator {
    fn new() -> Self {
        GreedyIntervalIterator {
            mp: BTreeMap::new(),
        }
    }

    /// Inserts [start, end] into the sorted structure, merging overlapping intervals.
    /// Time: O(k log n) where k = number of intervals merged (amortized O(log n) per insert
    ///       since each interval is inserted once and removed at most once)
    fn add_interval(&mut self, mut start: i32, mut end: i32) {
        // Step 1: Check left neighbor — does it extend into our interval?
        let mut left_used = false;
        let left = self.mp.range(..=start).next_back().map(|(&k, &v)| (k, v));
        if let Some((lk, lv)) = left {
            if lv >= start {
                // Left interval overlaps: extend our start to left's start, end to max of both ends
                start = lk;
                end = end.max(lv);
                left_used = true;
            }
        }

        // Step 2: Absorb all right neighbors that overlap with [start, end]
        // Collect all entries that start within (start, end]
        // We need to check up to end because a right interval starting at 'end' overlaps
        let keys_to_remove: Vec<(i32, i32)> = self.mp
            .range((Bound::Excluded(start), Bound::Included(end)))
            .map(|(&k, &v)| (k, v))
            .collect();
        for (k, v) in keys_to_remove {
            end = end.max(v);
            self.mp.remove(&k);
        }

        // Step 3: Remove the left neighbor if it was merged
        if left_used {
            // Only remove left if we actually used it
            let check = self.mp.range(..=start).next_back().map(|(&k, &v)| (k, v));
            if let Some((ck, cv)) = check {
                if cv >= start {
                    self.mp.remove(&ck);
                }
            }
        }

        // Step 4: Insert merged interval
        self.mp.insert(start, end);
    }

    // ── Clean implementation (production-ready) ────────────────────────────────
    /// Cleaner add_interval using a rebuilt approach for clarity.
    /// Same asymptotic complexity.
    fn add_interval_clean(&mut self, mut start: i32, mut end: i32) {
        // Extend left if left neighbor overlaps
        let left = self.mp.range(..=start).next_back().map(|(&k, &v)| (k, v));
        if let Some((lk, lv)) = left {
            if lv >= start {
                start = lk;
                end = end.max(lv);
                self.mp.remove(&lk);
            }
        }

        // Absorb all intervals that start within [start, end]
        loop {
            let next = self.mp.range(start..).next().map(|(&k, &v)| (k, v));
            if let Some((nk, nv)) = next {
                if nk > end { break; }
                end = end.max(nv);
                self.mp.remove(&nk);
            } else {
                break;
            }
        }

        self.mp.insert(start, end);
    }

    /// Returns all intervals sorted by start time.
    /// Time: O(n)
    fn get_intervals(&self) -> Vec<(i32, i32)> {
        self.mp.iter().map(|(&k, &v)| (k, v)).collect()
    }
}

fn format_intervals(intervals: &[(i32, i32)]) -> String {
    let mut sb = String::from("[");
    for (i, &(start, end)) in intervals.iter().enumerate() {
        sb.push('[');
        sb.push_str(&start.to_string());
        sb.push(',');
        sb.push_str(&end.to_string());
        sb.push(']');
        if i < intervals.len() - 1 { sb.push(','); }
    }
    sb.push(']');
    sb
}

fn main() {
    let mut gii = GreedyIntervalIterator::new();
    gii.add_interval_clean(1, 3);
    gii.add_interval_clean(6, 9);
    println!("After [1,3],[6,9]: {}", format_intervals(&gii.get_intervals()));
    // Expected: [[1,3],[6,9]]

    gii.add_interval_clean(2, 5);
    println!("After adding [2,5]: {}", format_intervals(&gii.get_intervals()));
    // Expected: [[1,5],[6,9]]

    gii.add_interval_clean(8, 10);
    println!("After adding [8,10]: {}", format_intervals(&gii.get_intervals()));
    // Expected: [[1,5],[6,10]]

    gii.add_interval_clean(0, 20);
    println!("After adding [0,20]: {}", format_intervals(&gii.get_intervals()));
    // Expected: [[0,20]] (all merged)
}
```

## Detailed Trace

| Step | Operation | Map State (start→end) | Note |
|------|-----------|----------------------|------|
| 1 | addInterval(1,3) | {1→3} | No neighbors |
| 2 | addInterval(6,9) | {1→3, 6→9} | No overlap with [1,3] |
| 3 | addInterval(2,5) | {1→5, 6→9} | Floor of 2 is 1→3; 3≥2 so merge: start=1, end=max(5,3)=5 |
| 4 | addInterval(8,10)| {1→5, 6→10} | Floor of 8 is 6→9; 9≥8 so merge: start=6, end=max(10,9)=10 |
| 5 | addInterval(0,20)| {0→20} | Floor of 0 is null; absorbs 1→5, 6→10 |

## Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| `addInterval` | O(k log n) amortized O(log n) | O(1) extra |
| `getIntervals` | O(n) | O(n) |
| Space (structure) | — | O(n) |

Where k = intervals merged (each interval inserted once, removed at most once → amortized O(log n)).

## Why BTreeMap?

`BTreeMap` provides:
- `mp.range(..=key).next_back()` — find largest key ≤ query in O(log n)
- `mp.range(key..).next()` — find smallest key ≥ query in O(log n)
- `.range()` with `Bound` variants — range view in O(log n) + O(k) to iterate
- Sorted iteration via `.iter()` in O(n)

A `HashMap` would require O(n) scan to find overlapping intervals.

## Related Problems
- [Insert Interval (LC #57)](../Patterns/Interval%20Scheduling.md) — one-shot insertion into sorted array
- [Merge Intervals (LC #56)](../Patterns/Interval%20Scheduling.md) — batch merge of unsorted intervals

> **Last Updated:** 2026-06-26
