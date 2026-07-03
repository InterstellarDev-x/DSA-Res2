# Merge Intervals Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `intervals` `sorting` `greedy` `sweep-line`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Rust Templates](#rust-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

The Merge Intervals pattern processes a list of `[start, end]` intervals, most commonly by **sorting on start time** and then doing a greedy linear scan to merge, insert, or count overlaps.

**Core merge rule:**
```
If current.start <= last.end  → overlapping → extend last.end
Else                          → non-overlapping → add new interval
```

The sort step is the key enabler: once sorted, all overlapping intervals are consecutive.

---

## When to Use

- Merging overlapping intervals
- Inserting a new interval into a sorted list
- Finding gaps between intervals
- Counting intervals that overlap at a given point
- Meeting rooms / scheduling problems
- Activity selection

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "merge all overlapping intervals" | Classic Merge Intervals |
| "insert interval and merge" | Insert + merge |
| "minimum meeting rooms" | Sweep line / min-heap |
| "maximum overlap at any point" | Sweep line events |
| "non-overlapping intervals to remove" | Greedy activity selection |
| "employee free time" | Multi-list merge |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Sort + merge | O(n log n) | O(n) |
| Insert interval (pre-sorted) | O(n) | O(n) |
| Meeting rooms I (can hold?) | O(n log n) | O(1) |
| Meeting rooms II (min rooms) | O(n log n) | O(n) |

---

## Rust Templates

### 1. Merge Overlapping Intervals

```rust
fn merge(intervals: &mut Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    intervals.sort_by(|a, b| a[0].cmp(&b[0])); // sort by start
    let mut merged: Vec<Vec<i32>> = Vec::new();

    for interval in intervals.iter() {
        if merged.is_empty() || merged.last().unwrap()[1] < interval[0] {
            merged.push(interval.clone()); // no overlap
        } else {
            // overlap: extend the end
            let last = merged.last_mut().unwrap();
            last[1] = last[1].max(interval[1]);
        }
    }
    merged
}
// Time: O(n log n) | Space: O(n)
```

### 2. Insert Interval

```rust
fn insert(intervals: &Vec<Vec<i32>>, mut new_interval: Vec<i32>) -> Vec<Vec<i32>> {
    let mut result: Vec<Vec<i32>> = Vec::new();
    let n = intervals.len();
    let mut i = 0;

    // Add all intervals ending before new_interval starts
    while i < n && intervals[i][1] < new_interval[0] {
        result.push(intervals[i].clone());
        i += 1;
    }
    // Merge all overlapping intervals with new_interval
    while i < n && intervals[i][0] <= new_interval[1] {
        new_interval[0] = new_interval[0].min(intervals[i][0]);
        new_interval[1] = new_interval[1].max(intervals[i][1]);
        i += 1;
    }
    result.push(new_interval);
    // Add remaining intervals
    while i < n {
        result.push(intervals[i].clone());
        i += 1;
    }

    result
}
// Time: O(n) | Space: O(n)
```

### 3. Meeting Rooms I — Can One Person Attend All?

```rust
fn can_attend_meetings(intervals: &mut Vec<Vec<i32>>) -> bool {
    intervals.sort_by(|a, b| a[0].cmp(&b[0]));
    for i in 1..intervals.len() {
        if intervals[i][0] < intervals[i - 1][1] {
            return false; // overlap
        }
    }
    true
}
// Time: O(n log n) | Space: O(1)
```

### 4. Meeting Rooms II — Minimum Rooms Required

```rust
fn min_meeting_rooms(intervals: &Vec<Vec<i32>>) -> i32 {
    let mut starts: Vec<i32> = intervals.iter().map(|iv| iv[0]).collect();
    let mut ends: Vec<i32> = intervals.iter().map(|iv| iv[1]).collect();
    starts.sort();
    ends.sort();

    let mut rooms = 0;
    let mut end_ptr = 0;
    for i in 0..starts.len() {
        if starts[i] < ends[end_ptr] {
            rooms += 1; // new room needed
        } else {
            end_ptr += 1; // room freed
        }
    }
    rooms
}
// Time: O(n log n) | Space: O(n)
```

### 5. Non-overlapping Intervals — Minimum Removals (Activity Selection)

```rust
fn erase_overlap_intervals(intervals: &mut Vec<Vec<i32>>) -> i32 {
    intervals.sort_by(|a, b| a[1].cmp(&b[1])); // sort by END
    let mut count = 0;
    let mut last_end = i32::MIN;
    for iv in intervals.iter() {
        if iv[0] >= last_end {
            last_end = iv[1]; // keep this interval
        } else {
            count += 1; // remove this interval
        }
    }
    count
}
// Greedy: sort by end, greedily keep non-overlapping intervals
// Time: O(n log n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Sorting by end instead of start (for merge) | Sort by `start` for merge; sort by `end` for activity selection |
| Overlap condition: `<` vs `<=` | `intervals[i][0] <= last_end` means touching intervals merge; adjust per problem |
| Mutating the original input array | Clone interval arrays when needed |
| Not handling empty input | Guard `if intervals.is_empty() { return ... }` |
| Meeting rooms: using a min-heap when two sorted arrays work | Two-pointer approach on sorted starts/ends is cleaner |

---

## Variations

| Variation | Key Idea |
|-----------|----------|
| Employee Free Time | Merge all intervals from all employees, find gaps |
| Interval List Intersections | Two pointer on two sorted interval lists |
| My Calendar I/II/III | BTreeMap / Sweep line for dynamic insertions |
| Maximum CPU Load | Sweep line, track sum of loads |

### Interval Intersection

```rust
fn interval_intersection(a: &Vec<Vec<i32>>, b: &Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let mut result: Vec<Vec<i32>> = Vec::new();
    let (mut i, mut j) = (0, 0);
    while i < a.len() && j < b.len() {
        let lo = a[i][0].max(b[j][0]);
        let hi = a[i][1].min(b[j][1]);
        if lo <= hi {
            result.push(vec![lo, hi]);
        }
        if a[i][1] < b[j][1] {
            i += 1;
        } else {
            j += 1;
        }
    }
    result
}
```

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Merge Intervals](https://leetcode.com/problems/merge-intervals/) | Medium | LC 56 |
| [Insert Interval](https://leetcode.com/problems/insert-interval/) | Medium | LC 57 |
| [Non-overlapping Intervals](https://leetcode.com/problems/non-overlapping-intervals/) | Medium | LC 435 |
| [Meeting Rooms](https://leetcode.com/problems/meeting-rooms/) | Easy | LC 252 |
| [Meeting Rooms II](https://leetcode.com/problems/meeting-rooms-ii/) | Medium | LC 253 |
| [Interval List Intersections](https://leetcode.com/problems/interval-list-intersections/) | Medium | LC 986 |
| [Employee Free Time](https://leetcode.com/problems/employee-free-time/) | Hard | LC 759 |
| [My Calendar III](https://leetcode.com/problems/my-calendar-iii/) | Hard | LC 732 |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — used in interval intersection
- [Greedy](../../10.Greedy/README.md) — activity selection is a greedy problem
- [Sweep Line](../../13.Graphs/Patterns/Sweep%20Line.md) — for counting overlaps at any point

---

> **Interview Tip:** Always clarify: do touching intervals `[1,2],[2,3]` merge or not? The problem usually specifies. Default assumption: `[1,2],[2,3]` → merge to `[1,3]` (non-strict overlap).

> **Last Updated:** 2026-06-26
