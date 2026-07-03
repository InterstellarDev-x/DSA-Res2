# Cyclic Sort Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `cyclic-sort` `in-place` `missing-number` `duplicate`

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

Cyclic Sort places each element at its **correct index** in a single O(n) pass. It works when the array contains numbers in range `[1, n]` or `[0, n]`.

**Core Insight:** If `nums[i]` belongs at index `nums[i] - 1` and isn't already there, swap it into place. Don't advance `i` until the element at `i` is correct.

```
i = 0
while i < n:
    correct = nums[i] - 1   // index where nums[i] belongs
    if nums[i] != nums[correct]:
        swap(nums[i], nums[correct])
    else:
        i++
```

After sorting, scan for anomalies (missing index, duplicate value).

---

## When to Use

- Array contains numbers in range `[1, n]` or `[0, n-1]`
- Finding missing/duplicate numbers in O(n) time, O(1) space
- Placing elements at their "natural" index

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "numbers from 1 to n, find missing" | Cyclic Sort |
| "numbers from 1 to n, find duplicate" | Cyclic Sort |
| "find all missing numbers in [1, n]" | Cyclic Sort + scan |
| "first missing positive" | Cyclic Sort variant |
| "set of n+1 numbers in [1, n]" | Cyclic Sort |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Cyclic sort pass | O(n) | O(1) |
| Find one missing | O(n) | O(1) |
| Find all missing | O(n) | O(1) |

> **Why O(n)?** Each element is swapped at most once — into its correct position. Total swaps ≤ n.

---

## Rust Templates

### 1. Cyclic Sort — Sort Numbers [1, n]

```rust
fn cyclic_sort(nums: &mut Vec<i32>) {
    let mut i = 0;
    while i < nums.len() {
        let correct = (nums[i] - 1) as usize; // nums[i] belongs at index correct
        if nums[i] != nums[correct] {
            nums.swap(i, correct);
        } else {
            i += 1;
        }
    }
}
// Time: O(n) | Space: O(1)
```

### 2. Find the Missing Number [0, n]

```rust
fn missing_number(nums: &mut Vec<i32>) -> i32 {
    let n = nums.len();
    let mut i = 0;
    while i < n {
        let correct = nums[i] as usize;
        // nums[i] should be at index nums[i], range [0, n]
        if nums[i] < n as i32 && nums[i] != nums[correct] {
            nums.swap(i, correct);
        } else {
            i += 1;
        }
    }
    // Scan for mismatch
    for j in 0..n {
        if nums[j] != j as i32 { return j as i32; }
    }
    n as i32 // n is missing
}
// Time: O(n) | Space: O(1)
```

### 3. Find the Duplicate Number [1, n] (One Duplicate)

```rust
fn find_duplicate(nums: &mut Vec<i32>) -> i32 {
    let mut i = 0;
    while i < nums.len() {
        let correct = (nums[i] - 1) as usize;
        if nums[i] != i as i32 + 1 {
            if nums[i] != nums[correct] {
                nums.swap(i, correct);
            } else {
                return nums[i]; // duplicate found
            }
        } else {
            i += 1;
        }
    }
    -1
}
// Time: O(n) | Space: O(1)
```

### 4. Find All Missing Numbers [1, n]

```rust
fn find_disappeared_numbers(nums: &mut Vec<i32>) -> Vec<i32> {
    let mut i = 0;
    while i < nums.len() {
        let correct = (nums[i] - 1) as usize;
        if nums[i] != nums[correct] {
            nums.swap(i, correct);
        } else {
            i += 1;
        }
    }
    let mut missing = Vec::new();
    for j in 0..nums.len() {
        if nums[j] != j as i32 + 1 { missing.push(j as i32 + 1); }
    }
    missing
}
// Time: O(n) | Space: O(1) (output not counted)
```

### 5. Find All Duplicates [1, n]

```rust
fn find_all_duplicates(nums: &mut Vec<i32>) -> Vec<i32> {
    let mut i = 0;
    while i < nums.len() {
        let correct = (nums[i] - 1) as usize;
        if nums[i] != nums[correct] {
            nums.swap(i, correct);
        } else {
            i += 1;
        }
    }
    let mut duplicates = Vec::new();
    for j in 0..nums.len() {
        if nums[j] != j as i32 + 1 { duplicates.push(nums[j]); }
    }
    duplicates
}
```

### 6. First Missing Positive (Cyclic Sort Variant)

```rust
fn first_missing_positive(nums: &mut Vec<i32>) -> i32 {
    let n = nums.len();
    let mut i = 0;
    while i < n {
        let correct = (nums[i] - 1) as usize;
        if nums[i] > 0 && nums[i] <= n as i32 && nums[i] != nums[correct] {
            nums.swap(i, correct);
        } else {
            i += 1;
        }
    }
    for j in 0..n {
        if nums[j] != j as i32 + 1 { return j as i32 + 1; }
    }
    n as i32 + 1
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Advancing `i` after every swap | Only advance when `nums[i] == i+1` (already correct) |
| Not handling out-of-range values | Guard: `nums[i] > 0 && nums[i] <= n` |
| Infinite loop when duplicate exists | Check `nums[i] != nums[correct]` before swapping |
| Missing the `n` case in `[0, n]` | After scan, return `n` if nothing found |
| Modifying input when you shouldn't | Clarify with interviewer if mutation is allowed |

---

## Variations

| Variation | Description |
|-----------|-------------|
| [0, n-1] range | `correct = nums[i]` instead of `nums[i] - 1` |
| Multiple missing numbers | Scan all mismatches after sort |
| Corrupt pair (one missing, one duplicate) | Run cyclic sort, scan for both |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Missing Number](https://leetcode.com/problems/missing-number/) | Easy | LC 268 |
| [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | LC 287 |
| [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Easy | LC 448 |
| [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Medium | LC 442 |
| [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | LC 41 |
| [Set Mismatch](https://leetcode.com/problems/set-mismatch/) | Easy | LC 645 |

---

## Related Patterns

- [Dutch National Flag](./Dutch%20National%20Flag.md) — in-place element placement
- [Two Pointers](./Two%20Pointers.md) — for in-place swapping
- [Bit Manipulation](../../6.Bit_Manipulation/README.md) — XOR trick for single missing number

---

> **Interview Tip:** LC 41 (First Missing Positive) is a classic Hard that uses this exact pattern. The "don't advance i" rule is the key insight interviewers check — many candidates write infinite loops.

> **Last Updated:** 2026-06-26
