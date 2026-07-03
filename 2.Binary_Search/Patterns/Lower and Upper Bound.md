# Lower Bound & Upper Bound Pattern

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `lower-bound` `upper-bound` `floor` `ceil` `first-last-occurrence`

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

**Lower Bound** of `x` in a sorted array: the index of the **first element ≥ x**.
**Upper Bound** of `x` in a sorted array: the index of the **first element > x**.

```
arr = [1, 3, 3, 5, 7]
       0  1  2  3  4

lowerBound(3) = 1   (first index where arr[i] >= 3)
upperBound(3) = 3   (first index where arr[i] >  3)

Count of 3s = upperBound(3) - lowerBound(3) = 3 - 1 = 2
```

These two primitives unlock: floor, ceil, first/last occurrence, count occurrences, insertion point — all in O(log n).

Rust's `slice::binary_search` is NOT lower/upper bound — it returns only a `Result` indicating presence/index, not guaranteed first matching index.

---

## When to Use

- Find first/last occurrence of a value
- Count occurrences in sorted array
- Find floor (largest ≤ x) or ceil (smallest ≥ x)
- Find insertion position maintaining sorted order
- Range queries on sorted data

---

## Recognition Cues

| Cue | What to Use |
|-----|-------------|
| "first position of x" | Lower Bound |
| "last position of x" | Upper Bound - 1 |
| "count of x in sorted array" | Upper Bound - Lower Bound |
| "floor of x" | Lower Bound - 1 (if `arr[lb] != x`) |
| "ceiling of x" | Lower Bound index value |
| "where to insert to keep sorted" | Lower Bound index |
| "find range [first, last] of target" | Both LB and UB |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Lower Bound | O(log n) | O(1) |
| Upper Bound | O(log n) | O(1) |
| First occurrence | O(log n) | O(1) |
| Last occurrence | O(log n) | O(1) |
| Count occurrences | O(log n) | O(1) |

---

## Rust Templates

### 1. Lower Bound — First index where `arr[i] >= target`

```rust
fn lower_bound(arr: &[i32], target: i32) -> usize {
    let mut lo = 0usize;
    let mut hi = arr.len(); // hi = n (one past last valid)
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] < target {
            lo = mid + 1;
        } else {
            hi = mid; // arr[mid] >= target: mid could be answer
        }
    }
    lo // lo == hi == first position where arr[i] >= target
    // Returns arr.len() if all elements < target
}
```

### 2. Upper Bound — First index where `arr[i] > target`

```rust
fn upper_bound(arr: &[i32], target: i32) -> usize {
    let mut lo = 0usize;
    let mut hi = arr.len();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if arr[mid] <= target {
            lo = mid + 1;
        } else {
            hi = mid;
        }
    }
    lo // first position where arr[i] > target
}
```

### 3. First and Last Occurrence of Target

```rust
fn search_range(nums: &[i32], target: i32) -> [i32; 2] {
    let first = lower_bound(nums, target);
    // Verify target actually exists
    if first == nums.len() || nums[first] != target {
        return [-1, -1];
    }
    let last = upper_bound(nums, target) - 1;
    [first as i32, last as i32]
}
// Time: O(log n) | Space: O(1)
```

### 4. Count Occurrences

```rust
fn count_occurrences(arr: &[i32], target: i32) -> usize {
    let first = lower_bound(arr, target);
    if first == arr.len() || arr[first] != target {
        return 0;
    }
    upper_bound(arr, target) - first
}
```

### 5. Floor — Largest element ≤ target

```rust
fn floor_val(arr: &[i32], target: i32) -> Option<i32> {
    let lb = lower_bound(arr, target);
    // If arr[lb] == target, that IS the floor
    if lb < arr.len() && arr[lb] == target {
        return Some(arr[lb]);
    }
    // Otherwise, floor is the element just before lb
    if lb == 0 {
        return None; // all elements > target
    }
    Some(arr[lb - 1])
}
```

### 6. Ceil — Smallest element ≥ target

```rust
fn ceil_val(arr: &[i32], target: i32) -> Option<i32> {
    let lb = lower_bound(arr, target);
    if lb == arr.len() {
        return None; // all elements < target
    }
    Some(arr[lb])
}
```

### 7. Search Insert Position

```rust
fn search_insert(nums: &[i32], target: i32) -> usize {
    lower_bound(nums, target)
    // If target exists, returns its first index
    // If not, returns where it would be inserted
}
```

### 8. Using Rust Standard Library

```rust
// Rust standard library provides partition_point for lower_bound behaviour
// partition_point returns the first index where the predicate is false

let lb = arr.partition_point(|&x| x < target);  // first index where arr[i] >= target
let ub = arr.partition_point(|&x| x <= target); // first index where arr[i] > target

// But roll your own for interviews — don't rely on library
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `hi = arr.len() - 1` instead of `arr.len()` | Lower/Upper bound range is `[0, n]` — include `n` as valid answer |
| Returning `lo - 1` for upper bound | Upper bound returns `lo` directly (first index `> target`) |
| Not verifying `arr[first] == target` after lower bound | Lower bound returns insertion point even when target absent |
| Using `<` vs `<=` in condition | Lower bound: `arr[mid] < target` → `lo=mid+1`. Upper bound: `arr[mid] <= target` → `lo=mid+1` |
| Off-by-one for last occurrence | Last = `upper_bound(target) - 1`, not `upper_bound(target)` |

---

## Variations

| Variation | Code Tweak |
|-----------|-----------|
| Leftmost position ≥ x | Lower Bound (standard) |
| Rightmost position ≤ x | Upper Bound - 1 |
| First position where `f(i)` is true | Binary Search on Answer (predicate-based) |
| Kth missing positive | Lower bound on virtual sorted array |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy | LC 35 |
| [Find First and Last Position of Element](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) | Medium | LC 34 |
| [Count of Smaller Numbers After Self](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | Hard | LC 315 |
| [Kth Smallest in Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | LC 378 |
| [H-Index II](https://leetcode.com/problems/h-index-ii/) | Medium | LC 275 |
| Floor and Ceil in Sorted Array | Easy | GFG |
| Count occurrences of element in sorted array | Easy | GFG |

---

## Related Patterns

- [Classic Binary Search](./Classic%20Binary%20Search.md) — exact target search
- [Binary Search on Answer](./Binary%20Search%20on%20Answer.md) — when predicate replaces comparison

---

> **Interview Tip:** Build `lower_bound` and `upper_bound` as reusable private methods in your solution struct. Then `first_occurrence`, `last_occurrence`, and `count_occurrences` all become one-liners. Interviewers love seeing clean decomposition.

> **Last Updated:** 2026-06-26
