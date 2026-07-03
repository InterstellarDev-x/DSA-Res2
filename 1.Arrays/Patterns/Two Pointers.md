# Two Pointers Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `two-pointers` `sorted-array` `in-place` `linear`

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

Two Pointers uses two index variables that move through an array (or two arrays) to reduce a nested-loop O(n²) problem to O(n).

**Three sub-styles:**

| Style | Pointer Start | Move Condition |
|-------|--------------|----------------|
| Opposite ends | `l=0, r=n-1` | Move one inward per step |
| Same direction | `slow=0, fast=0` | Fast races ahead |
| Two arrays | `i=0, j=0` | Advance the smaller one |

The key invariant: at every step, pointers maintain a constraint (e.g. `sum ≤ target`) that guides which pointer moves.

---

## When to Use

- Sorted array, need pairs/triplets summing to target
- Remove duplicates / elements in-place
- Merge two sorted arrays/lists
- Palindrome check
- Container with most water (optimization)
- Partition problems (negative/positive)

---

## Recognition Cues

| Cue in Problem | Style |
|----------------|-------|
| "two numbers in sorted array that sum to target" | Opposite ends |
| "remove duplicates in-place" | Same direction (fast/slow) |
| "merge two sorted arrays" | Two arrays |
| "is palindrome" | Opposite ends |
| "minimum window / subarray" | See [Sliding Window](./Sliding%20Window.md) |
| "container with most water" | Opposite ends |
| "3Sum / 4Sum" | Sort + opposite ends inside loop |

---

## Complexity

| Algorithm | Time | Space |
|-----------|------|-------|
| Two Sum (sorted) | O(n) | O(1) |
| Remove Duplicates | O(n) | O(1) |
| 3Sum | O(n²) | O(1) |
| 4Sum | O(n³) | O(1) |
| Merge Two Arrays | O(m+n) | O(m+n) |

---

## Rust Templates

### 1. Opposite Ends — Two Sum (Sorted)

```rust
fn two_sum(arr: &[i32], target: i32) -> (i32, i32) {
    let mut l = 0;
    let mut r = arr.len() - 1;
    while l < r {
        let sum = arr[l] + arr[r];
        if sum == target {
            return (l as i32, r as i32);
        } else if sum < target {
            l += 1;
        } else {
            r -= 1;
        }
    }
    (-1, -1)
}
// Time: O(n) | Space: O(1)
```

### 2. Same Direction — Remove Duplicates (Sorted)

```rust
fn remove_duplicates(nums: &mut Vec<i32>) -> usize {
    if nums.is_empty() {
        return 0;
    }
    let mut slow = 0;
    for fast in 1..nums.len() {
        if nums[fast] != nums[slow] {
            slow += 1;
            nums[slow] = nums[fast];
        }
    }
    slow + 1 // new length
}
// Time: O(n) | Space: O(1)
```

### 3. Move Zeros to End

```rust
fn move_zeroes(nums: &mut Vec<i32>) {
    let mut slow = 0;
    for fast in 0..nums.len() {
        if nums[fast] != 0 {
            nums[slow] = nums[fast];
            slow += 1;
        }
    }
    while slow < nums.len() {
        nums[slow] = 0;
        slow += 1;
    }
}
// Time: O(n) | Space: O(1)
```

### 4. 3Sum

```rust
fn three_sum(nums: &mut Vec<i32>) -> Vec<Vec<i32>> {
    nums.sort();
    let mut result: Vec<Vec<i32>> = Vec::new();
    let n = nums.len();

    for i in 0..n.saturating_sub(2) {
        if i > 0 && nums[i] == nums[i - 1] {
            continue; // skip duplicate i
        }

        let mut l = i + 1;
        let mut r = n - 1;
        while l < r {
            let sum = nums[i] + nums[l] + nums[r];
            if sum == 0 {
                result.push(vec![nums[i], nums[l], nums[r]]);
                while l < r && nums[l] == nums[l + 1] { l += 1; } // skip dup l
                while l < r && nums[r] == nums[r - 1] { r -= 1; } // skip dup r
                l += 1;
                r -= 1;
            } else if sum < 0 {
                l += 1;
            } else {
                r -= 1;
            }
        }
    }
    result
}
// Time: O(n²) | Space: O(1) excluding output
```

### 5. Container With Most Water

```rust
fn max_area(height: &[i32]) -> i32 {
    let mut l = 0;
    let mut r = height.len() - 1;
    let mut max_water = 0;
    while l < r {
        let water = height[l].min(height[r]) * (r - l) as i32;
        max_water = max_water.max(water);
        if height[l] < height[r] {
            l += 1;
        } else {
            r -= 1;
        }
    }
    max_water
}
// Time: O(n) | Space: O(1)
```

### 6. Merge Two Sorted Arrays into One

```rust
fn merge_sorted(a: &[i32], b: &[i32]) -> Vec<i32> {
    let mut result = Vec::with_capacity(a.len() + b.len());
    let mut i = 0;
    let mut j = 0;
    while i < a.len() && j < b.len() {
        if a[i] <= b[j] {
            result.push(a[i]);
            i += 1;
        } else {
            result.push(b[j]);
            j += 1;
        }
    }
    while i < a.len() {
        result.push(a[i]);
        i += 1;
    }
    while j < b.len() {
        result.push(b[j]);
        j += 1;
    }
    result
}
// Time: O(m+n) | Space: O(m+n)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Applying Two Pointers to unsorted array | Sort first (adds O(n log n)) or use HashMap |
| Loop condition `l < r` vs `l <= r` | `l < r` for pairs; `l <= r` if element can be used alone |
| 3Sum: not skipping duplicates | Skip when `nums[i] == nums[i-1]` AND `nums[l] == nums[l+1]` |
| Moving both pointers when sum equals target | Move both; `l++` and `r--` together |
| Integer overflow in sum of large values | Cast to `i64` before adding |

---

## Variations

| Variation | Description |
|-----------|-------------|
| 4Sum | Sort + three-pointer (one fixed, two-pointer inside) |
| Trapping Rain Water | Two pointers with max-left / max-right tracking |
| Minimum in Rotated Sorted Array | Modified Two Pointers + Binary Search |
| Palindrome (Two Pointers on String) | Opposite ends, compare chars |
| Dutch National Flag | Three pointers (lo, mid, hi) |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Two Sum II - Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Easy | LC 167 |
| [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy | LC 26 |
| [Move Zeroes](https://leetcode.com/problems/move-zeroes/) | Easy | LC 283 |
| [Squares of Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) | Easy | LC 977 |
| [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium | LC 11 |
| [3Sum](https://leetcode.com/problems/3sum/) | Medium | LC 15 |
| [4Sum](https://leetcode.com/problems/4sum/) | Medium | LC 18 |
| [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | LC 42 |

---

## Related Patterns

- [Sliding Window](./Sliding%20Window.md) — Two Pointers where the window size changes
- [Dutch National Flag](./Dutch%20National%20Flag.md) — Three-pointer variant
- [Prefix Sum](./Prefix%20Sum.md) — When counting subarrays instead of finding pairs

---

> **Interview Tip:** For any "find pair/triplet with sum" on unsorted arrays, ask: "Can I sort it?" Sorting costs O(n log n) but enables O(n) two-pointer scan — almost always the optimal approach.

> **Last Updated:** 2026-06-26
