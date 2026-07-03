# Kadane's Algorithm Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `kadane` `subarray` `dynamic-programming` `greedy`

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

Kadane's Algorithm finds the **maximum sum contiguous subarray** in O(n) time.

**Core Insight:** At each index `i`, decide whether to extend the existing subarray or start fresh. Start fresh when the current running sum becomes negative — it can only hurt future sums.

```
maxEndingHere = max(arr[i], maxEndingHere + arr[i])
maxSoFar      = max(maxSoFar, maxEndingHere)
```

This is a **greedy-DP hybrid**: greedy in the extension decision, DP in state transition.

---

## When to Use

- Maximum (or minimum) sum subarray
- Maximum product subarray
- Maximum sum circular subarray
- Maximum sum rectangle in a 2D matrix
- Variants where you need indices of the subarray

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "maximum sum of contiguous subarray" | Classic Kadane |
| "maximum product subarray" | Kadane with two states (max, min) |
| "maximum sum subarray in circular array" | Kadane + (totalSum - minKadane) |
| "find subarray with largest sum" | Kadane + index tracking |
| "at most one deletion" | Kadane forward + backward |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Classic Kadane | O(n) | O(1) |
| With indices | O(n) | O(1) |
| Max product | O(n) | O(1) |
| Circular | O(n) | O(1) |
| 2D max rectangle | O(m × n²) | O(m) |

---

## Rust Templates

### 1. Classic Kadane — Maximum Sum

```rust
fn max_sub_array(nums: &[i32]) -> i32 {
    let mut max_so_far = nums[0];
    let mut max_ending_here = nums[0];

    for i in 1..nums.len() {
        max_ending_here = nums[i].max(max_ending_here + nums[i]);
        max_so_far = max_so_far.max(max_ending_here);
    }
    max_so_far
}
// Time: O(n) | Space: O(1)
```

### 2. Kadane with Subarray Indices

```rust
fn max_sub_array_with_indices(nums: &[i32]) -> (i32, usize, usize) {
    let mut max_sum = nums[0];
    let mut cur_sum = nums[0];
    let mut start = 0usize;
    let mut end = 0usize;
    let mut temp_start = 0usize;

    for i in 1..nums.len() {
        if cur_sum + nums[i] < nums[i] {
            cur_sum = nums[i];
            temp_start = i;
        } else {
            cur_sum += nums[i];
        }
        if cur_sum > max_sum {
            max_sum = cur_sum;
            start = temp_start;
            end = i;
        }
    }
    (max_sum, start, end)
}
// Returns (maxSum, startIndex, endIndex)
```

### 3. Maximum Product Subarray

```rust
fn max_product(nums: &[i32]) -> i32 {
    let mut max_prod = nums[0];
    let mut cur_max = nums[0];
    let mut cur_min = nums[0]; // track both: negatives flip sign

    for i in 1..nums.len() {
        let temp = cur_max;
        cur_max = nums[i].max((cur_max * nums[i]).max(cur_min * nums[i]));
        cur_min = nums[i].min((temp * nums[i]).min(cur_min * nums[i]));
        max_prod = max_prod.max(cur_max);
    }
    max_prod
}
// Time: O(n) | Space: O(1)
```

### 4. Maximum Sum Circular Subarray

```rust
fn max_subarray_sum_circular(nums: &[i32]) -> i32 {
    let mut total_sum = 0;
    let mut max_sum = nums[0];
    let mut cur_max = 0;
    let mut min_sum = nums[0];
    let mut cur_min = 0;

    for &num in nums {
        cur_max = (cur_max + num).max(num);
        max_sum = max_sum.max(cur_max);

        cur_min = (cur_min + num).min(num);
        min_sum = min_sum.min(cur_min);

        total_sum += num;
    }
    // If all negative, total_sum - min_sum wraps entire array → invalid
    if max_sum > 0 { max_sum.max(total_sum - min_sum) } else { max_sum }
}
// Time: O(n) | Space: O(1)
```

### 5. Minimum Sum Subarray (Kadane Inverted)

```rust
fn min_sub_array(nums: &[i32]) -> i32 {
    let mut min_so_far = nums[0];
    let mut min_ending_here = nums[0];

    for i in 1..nums.len() {
        min_ending_here = nums[i].min(min_ending_here + nums[i]);
        min_so_far = min_so_far.min(min_ending_here);
    }
    min_so_far
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Initializing `maxSoFar = 0` | Init to `nums[0]` — all-negative arrays have negative max |
| Starting loop at index 0 | Start `maxEndingHere = nums[0]`, loop from index 1 |
| Product variant: forgetting `curMin` | Negative × negative = large positive; track both max and min |
| Circular: returning `totalSum - minSum` when all elements negative | Guard: `if maxSum < 0, return maxSum` |
| Index tracking: not resetting `tempStart` correctly | Reset `tempStart = i` only when starting fresh |

---

## Variations

| Variation | Description | Key Change |
|-----------|-------------|------------|
| Count of max-sum subarrays | Count how many subarrays achieve max sum | Track count alongside |
| At most k deletions | DP with state `(index, deletions_used)` | O(n×k) DP |
| Max sum with no two adjacent | House Robber pattern | DP |
| Max sum submatrix | Fix row bounds, Kadane on column sums | O(m²×n) |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Maximum Subarray](https://leetcode.com/problems/maximum-subarray/) | Medium | LC 53 |
| [Maximum Product Subarray](https://leetcode.com/problems/maximum-product-subarray/) | Medium | LC 152 |
| [Maximum Sum Circular Subarray](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Medium | LC 918 |
| [Maximum Subarray Sum with One Deletion](https://leetcode.com/problems/maximum-subarray-sum-with-one-deletion/) | Medium | LC 1186 |
| [Max Sum of Rectangle No Larger Than K](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/) | Hard | LC 363 |
| [K-Concatenation Maximum Sum](https://leetcode.com/problems/k-concatenation-maximum-sum/) | Medium | LC 1191 |

---

## Related Patterns

- [Prefix Sum](./Prefix%20Sum.md) — when you need exact sum = K, not maximum
- [Sliding Window](./Sliding%20Window.md) — when all values are non-negative (can shrink window)
- [Dynamic Programming](../../14.Dynamic_Programming/README.md) — Kadane is DP with O(1) state compression

---

> **Interview Tip:** Interviewers love asking "what if all numbers are negative?" — always initialize from `nums[0]` and start the loop at index 1. Also, follow-up: "print the actual subarray" — track `tempStart` and `start`/`end` indices.

> **Last Updated:** 2026-06-26
