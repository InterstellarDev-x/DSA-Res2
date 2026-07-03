> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Mistakes**

# Greedy Algorithms — Common Mistakes (Top 10)

## Mistake 1: Wrong Sort Key — Start vs End

**Bug:** Sorting by start time when you should sort by end time for interval scheduling.

```rust
// WRONG — sort by start time for non-overlapping intervals
intervals.sort_by(|a, b| a[0].cmp(&b[0]));

// CORRECT — sort by end time
intervals.sort_by(|a, b| a[1].cmp(&b[1]));
```

**Consequence:** Counter-example: `[[1,10],[2,3],[4,5]]`. Sorted by start, greedy keeps `[1,10]` first, then removes everything. Answer: 2 (wrong). Sorted by end, keeps `[2,3]` and `[4,5]`. Answer: 1 (correct).

**Rule of thumb:**
- "Maximize number of non-overlapping intervals" → sort by **end**
- "Merge overlapping intervals" → sort by **start**
- "Minimum arrows / removals" → sort by **end**

---

## Mistake 2: Missing Gas Station Circular Check

**Bug:** Forgetting to check if total gain is negative before returning `startStation`.

```rust
// WRONG — forgets infeasibility check
fn can_complete_circuit_wrong(gas: &[i32], cost: &[i32]) -> i32 {
    let mut current_gain = 0i32;
    let mut start_station = 0usize;
    for i in 0..gas.len() {
        current_gain += gas[i] - cost[i];
        if current_gain < 0 { start_station = i + 1; current_gain = 0; }
    }
    start_station as i32 // BUG: may return invalid answer when no solution exists
}

// CORRECT
fn can_complete_circuit(gas: &[i32], cost: &[i32]) -> i32 {
    let mut total_gain = 0i32;
    let mut current_gain = 0i32;
    let mut start_station = 0usize;
    for i in 0..gas.len() {
        let gain = gas[i] - cost[i];
        total_gain += gain;
        current_gain += gain;
        if current_gain < 0 { start_station = i + 1; current_gain = 0; }
    }
    if total_gain >= 0 { start_station as i32 } else { -1 } // CORRECT: check feasibility
}
```

**Test case that reveals bug:** `gas=[2,3,4]`, `cost=[3,4,3]`. Total gain = -1, no solution. Wrong code returns 2; correct code returns -1.

---

## Mistake 3: Candy Single-Pass Failure

**Bug:** Trying to solve Candy in a single left-to-right pass.

```rust
// WRONG — single pass misses decreasing sequences
fn candy_wrong(ratings: &[i32]) -> i32 {
    let mut candies = vec![1i32; ratings.len()];
    for i in 1..ratings.len() {
        if ratings[i] > ratings[i - 1] { candies[i] = candies[i - 1] + 1; }
        else if ratings[i] < ratings[i - 1] { candies[i - 1] += 1; } // BUG: too late!
    }
    // ...
    0
}
```

**Why it fails:** `[5, 4, 3, 2, 1]` — when we reach index 1 and increment index 0, we have already committed to index 0's value being used for future comparisons. The retroactive increment does not propagate backward correctly.

**Correct: Two passes.** See [Greedy on Strings](../Patterns/Greedy%20on%20Strings.md) for full solution.

---

## Mistake 4: Valid Parenthesis String — Forgetting `*` Can Be Empty

**Bug:** Only treating `*` as `(` or `)`, not as empty string.

```rust
// The real bug: not clamping lo = lo.max(0)

// WRONG — lo goes negative, which is invalid
for c in s.chars() {
    if c == '(' { lo += 1; hi += 1; }
    else if c == ')' { lo -= 1; hi -= 1; }
    else { lo -= 1; hi += 1; }
    if hi < 0 { return false; }
    // Missing: lo = lo.max(0);
}
lo == 0 // BUG: lo might be -2, and this returns false when should be true
```

**Test case:** `s = "*"`. lo=-1, hi=1. Without clamp, lo=-1 ≠ 0, returns false. But `"*"` is valid (treat as empty). Correct answer: true.

---

## Mistake 5: Jump Game II — Off-by-One in Loop Bound

**Bug:** Using `i <= nums.length - 1` instead of `i < nums.length - 1`.

```rust
// WRONG — processes the last index unnecessarily, overcounts jumps
for i in 0..nums.len() {
    farthest = farthest.max(i + nums[i] as usize);
    if i == cur_end { jumps += 1; cur_end = farthest; }
}

// CORRECT — stop at second-to-last index
for i in 0..nums.len() - 1 {
    farthest = farthest.max(i + nums[i] as usize);
    if i == cur_end { jumps += 1; cur_end = farthest; }
}
```

**Why:** When `i == nums.len() - 1`, we are already AT the destination. We do not need to "jump from" it. Processing it causes a spurious increment of `jumps`.

**Test case:** `[0]`. Single element, already at destination. Correct: 0 jumps. Wrong: 1 jump.

---

## Mistake 6: Meeting Rooms II — Wrong Heap Type

**Bug:** Using max-heap instead of min-heap.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — max-heap gives farthest ending room, not earliest ending room
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // default is max-heap

// CORRECT — min-heap gives earliest ending room
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
```

**Why it matters:** We want to check if ANY room is free (has end ≤ new start). The room most likely to be free is the one that ends earliest. Max-heap gives the room that ends latest — useless for this check.

---

## Mistake 7: Insert Interval — Wrong Overlap Condition

**Bug:** Using `<` instead of `<=` in the overlap merge condition.

```rust
// WRONG — misses touching intervals [1,3] and [3,5] should merge to [1,5]
while i < n && intervals[i][0] < new_interval[1] { /* ... */ }

// CORRECT — intervals touching at a single point ARE overlapping
while i < n && intervals[i][0] <= new_interval[1] { /* ... */ }
```

**Test case:** `intervals=[[1,3],[3,5]]`, `newInterval=[2,3]`. Wrong code stops before merging `[3,5]` because `3 < 3` is false. Correct code merges both.

---

## Mistake 8: Hand of Straights — Not Cleaning Up Zero-Count Keys

**Bug:** Not removing keys with count 0 from `BTreeMap`, causing the first key to return completed cards.

```rust
use std::collections::BTreeMap;

// WRONG — zero-count keys remain in map
*count.get_mut(&i).unwrap() -= 1;
// If count[i] == 0, the key stays in the BTreeMap

// CORRECT — remove exhausted keys
*count.get_mut(&i).unwrap() -= 1;
if count[&i] == 0 { count.remove(&i); }
```

**Consequence:** The first key of the map may return a card with count 0 on the next iteration, causing an infinite loop or incorrect group formation.

---

## Mistake 9: Non-overlapping Intervals — Wrong Comparison Operator

**Bug:** Using `>` instead of `>=` for the non-overlap check.

```rust
// WRONG — treats touching intervals [1,2] and [2,3] as overlapping
if interval[0] > prev_end { kept += 1; prev_end = interval[1]; }

// CORRECT — touching intervals do NOT overlap
if interval[0] >= prev_end { kept += 1; prev_end = interval[1]; }
```

**Test case:** `[[1,2],[2,3]]`. These intervals touch at point 2 but do not overlap. Wrong code removes one; correct code keeps both (result = 0 removals).

---

## Mistake 10: Partition Labels — Using Frequency Count Instead of Last Occurrence

**Bug:** Building a frequency count instead of a last-occurrence map.

```rust
// WRONG — frequency count does not tell us WHERE the last occurrence is
let mut freq = vec![0i32; 26];
for c in s.chars() { freq[(c as u8 - b'a') as usize] += 1; }

// CORRECT — last occurrence index is what we need for window stretching
let mut last = vec![0usize; 26];
for (i, c) in s.chars().enumerate() { last[(c as u8 - b'a') as usize] = i; }
```

**Why:** The window must extend to the POSITION of the last occurrence of each character (so the character is fully contained), not just count how many times it occurs.

---

## Quick Reference: Bug → Fix Table

| Mistake | Symptom | Fix |
|---------|---------|-----|
| Wrong sort key | Wrong answer on interval problems | Sort by end (maximize) or start (merge) |
| Gas Station no feasibility check | Returns wrong station when -1 expected | Track `totalGain`, return -1 if negative |
| Candy single pass | Wrong candy count on peaks/valleys | Two passes: L→R then R→L |
| Valid Paren missing lo clamp | Returns false on valid `"*"` strings | Add `lo = lo.max(0)` |
| Jump Game II loop bound | Overcounts jumps | Use `i < nums.len() - 1` |
| Meeting Rooms max-heap | Allocates too many rooms | Use `BinaryHeap<Reverse<i32>>` (min) |
| Insert Interval `<` vs `<=` | Misses touching interval merges | Use `<=` in merge condition |
| Hand of Straights no cleanup | Infinite loop or wrong groups | Remove 0-count keys from BTreeMap |
| Non-overlapping `>` vs `>=` | Removes touching intervals | Use `>=` for non-overlap |
| Partition Labels frequency vs last | Wrong partition boundaries | Build last-occurrence array, not freq |

> **Last Updated:** 2026-06-26
