# Classic Binary Search Pattern

> **Topic:** [Binary Search](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `binary-search` `sorted-array` `rotated-array` `O(log n)`

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

Binary Search eliminates half the search space at every step by exploiting a **monotone ordering** property. The core loop maintains one invariant:

> **The answer always lies within `[lo, hi]` at the start of every iteration.**

Three template styles exist — each handles the invariant slightly differently:

| Style | `lo` init | `hi` init | Mid formula | Loop condition | Narrows by |
|-------|-----------|-----------|-------------|----------------|-----------|
| **Inclusive** | `0` | `n-1` | `lo+(hi-lo)/2` | `lo <= hi` | `lo=mid+1` or `hi=mid-1` |
| **Half-open** | `0` | `n` | `lo+(hi-lo)/2` | `lo < hi` | `lo=mid+1` or `hi=mid` |
| **Left-biased** | `0` | `n-1` | `lo+(hi-lo)/2` | `lo < hi` | always `hi=mid` or `lo=mid+1` |

**Use the inclusive template** for "find exact target". **Use half-open** for "find first position where condition is true" (lower bound).

---

## When to Use

- Array is sorted (or partially sorted — rotated)
- Answer must be found in O(log n)
- Problem says "sorted array" + "find" / "search" / "count occurrences"
- Problem involves a rotated sorted array

---

## Recognition Cues

| Cue | Variant |
|-----|---------|
| "sorted array, find target" | Classic inclusive |
| "find first/last occurrence" | [Lower/Upper Bound](./Lower%20and%20Upper%20Bound.md) |
| "rotated sorted array, find target" | Rotated variant |
| "find minimum in rotated sorted array" | Rotated variant |
| "single non-duplicate in sorted pairs" | XOR index parity |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| All classic variants | O(log n) | O(1) |
| Recursive binary search | O(log n) | O(log n) stack |

---

## Rust Templates

### 1. Classic — Find Exact Target (Inclusive)

```rust
fn binary_search(nums: &[i32], target: i32) -> Option<usize> {
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo <= hi {
        let mid = lo + (hi - lo) / 2; // avoids integer overflow vs (lo+hi)/2
        let m = mid as usize;
        if nums[m] == target { return Some(m); }
        else if nums[m] < target { lo = mid + 1; }
        else { hi = mid - 1; }
    }
    None // not found
}
// Time: O(log n) | Space: O(1)
```

### 2. Search in Rotated Sorted Array I (no duplicates)

```rust
fn search_rotated(nums: &[i32], target: i32) -> Option<usize> {
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        let m = mid as usize;
        if nums[m] == target { return Some(m); }

        // Determine which half is sorted
        if nums[lo as usize] <= nums[m] {          // left half is sorted
            if nums[lo as usize] <= target && target < nums[m] {
                hi = mid - 1;                      // target in left half
            } else {
                lo = mid + 1;
            }
        } else {                                   // right half is sorted
            if nums[m] < target && target <= nums[hi as usize] {
                lo = mid + 1;                      // target in right half
            } else {
                hi = mid - 1;
            }
        }
    }
    None
}
// Time: O(log n) | Space: O(1)
```

### 3. Search in Rotated Sorted Array II (with duplicates)

```rust
fn search_rotated_with_dups(nums: &[i32], target: i32) -> bool {
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        let m = mid as usize;
        if nums[m] == target { return true; }

        // Handle duplicates: can't determine sorted half
        if nums[lo as usize] == nums[m] && nums[m] == nums[hi as usize] {
            lo += 1; hi -= 1;                      // shrink both ends
            continue;
        }
        if nums[lo as usize] <= nums[m] {
            if nums[lo as usize] <= target && target < nums[m] { hi = mid - 1; }
            else { lo = mid + 1; }
        } else {
            if nums[m] < target && target <= nums[hi as usize] { lo = mid + 1; }
            else { hi = mid - 1; }
        }
    }
    false
}
// Time: O(log n) average, O(n) worst (all duplicates) | Space: O(1)
```

### 4. Find Minimum in Rotated Sorted Array

```rust
fn find_min(nums: &[i32]) -> i32 {
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if nums[mid as usize] > nums[hi as usize] {
            lo = mid + 1;  // min is in right half
        } else {
            hi = mid;      // mid could be the min; don't exclude it
        }
    }
    nums[lo as usize]
}
// Time: O(log n) | Space: O(1)
```

### 5. Single Element in Sorted Array

```rust
// All elements appear twice except one; find the single element.
fn single_non_duplicate(nums: &[i32]) -> i32 {
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo < hi {
        let mut mid = lo + (hi - lo) / 2;
        if mid % 2 == 1 { mid -= 1; }             // ensure mid is even index
        if nums[mid as usize] == nums[(mid + 1) as usize] {
            lo = mid + 2;                          // single is to the right
        } else {
            hi = mid;                              // single is at mid or left
        }
    }
    nums[lo as usize]
}
// Time: O(log n) | Space: O(1)
```

### 6. Count Rotations (How Many Times Rotated)

```rust
fn count_rotations(nums: &[i32]) -> usize {
    // Index of minimum = number of rotations
    let mut lo = 0i32;
    let mut hi = nums.len() as i32 - 1;
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        if nums[mid as usize] > nums[hi as usize] { lo = mid + 1; }
        else { hi = mid; }
    }
    lo as usize
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `mid = (lo + hi) / 2` overflow | Use `lo + (hi - lo) / 2` |
| `while (lo < hi)` then accessing `nums[hi]` after loop | After loop, `lo == hi` — only one element remains |
| `hi = mid` vs `hi = mid - 1` — which to use? | If `mid` can be the answer, use `hi = mid`; if not, `hi = mid - 1` |
| Rotated search: not checking which half is sorted first | Always check sorted half first, then test if target is in that range |
| Infinite loop when `lo == mid` | Happens when `lo + 1 == hi` and mid-formula rounds down — use `lo + (hi - lo + 1) / 2` (upper mid) for right-biased |

---

## Variations

| Variation | Key Difference |
|-----------|---------------|
| Rotated with duplicates | Add `lo += 1; hi -= 1` when `nums[lo]==nums[mid]==nums[hi]` |
| Find rotation count | Index of minimum = rotation count |
| Single non-duplicate | Pair indexing trick (force even index mid) |
| Find peak element | See [Peak Finding](./Peak%20Finding.md) |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Binary Search](https://leetcode.com/problems/binary-search/) | Easy | LC 704 |
| [Search Insert Position](https://leetcode.com/problems/search-insert-position/) | Easy | LC 35 |
| [Search in Rotated Sorted Array](https://leetcode.com/problems/search-in-rotated-sorted-array/) | Medium | LC 33 |
| [Search in Rotated Sorted Array II](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) | Medium | LC 81 |
| [Find Minimum in Rotated Sorted Array](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) | Medium | LC 153 |
| [Find Minimum in Rotated Sorted Array II](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array-ii/) | Hard | LC 154 |
| [Single Element in a Sorted Array](https://leetcode.com/problems/single-element-in-a-sorted-array/) | Medium | LC 540 |

---

## Related Patterns

- [Lower and Upper Bound](./Lower%20and%20Upper%20Bound.md) — for first/last occurrence queries
- [Binary Search on Answer](./Binary%20Search%20on%20Answer.md) — when search space is not an array
- [Peak Finding](./Peak%20Finding.md) — unimodal arrays and 2D peaks

---

> **Interview Tip:** The single biggest source of bugs is the `hi = mid` vs `hi = mid - 1` decision. Rule of thumb: if after the mid check the element at `mid` *could still be the answer*, use `hi = mid`. If it definitely can't, use `hi = mid - 1`.

> **Last Updated:** 2026-06-26
