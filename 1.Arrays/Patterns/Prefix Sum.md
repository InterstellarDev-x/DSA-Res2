# Prefix Sum Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `prefix-sum` `subarray` `range-query` `hashmap`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Rust Template](#rust-template)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Prefix Sum precomputes cumulative sums so that any subarray sum `[l, r]` can be answered in **O(1)** after an **O(n)** build phase.

```
prefix[0] = 0
prefix[i] = arr[0] + arr[1] + ... + arr[i-1]

sum(l, r) = prefix[r+1] - prefix[l]
```

The key insight: instead of summing `arr[l..r]` every query, store prefix sums and subtract.

For **subarray sum = K** problems, pair it with a `HashMap` tracking `count of prefix sums seen so far`. If `prefix[j] - K` was seen, the subarray ending at `j` has sum `K`.

---

## When to Use

- Range sum queries on a static array
- Count subarrays with sum / XOR / product equal to a target
- 2D matrix range sum queries
- Difference arrays for range updates

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "subarray sum equals K" | Prefix Sum + `HashMap` |
| "number of subarrays with sum ≤ K" | Prefix Sum + Binary Search or sliding window |
| "range sum query, immutable" | Classic Prefix Sum |
| "how many subarrays have even/odd sum" | Prefix Sum parity |
| "minimum operations to make subarray sum 0" | Prefix Sum difference |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix array | O(n) | O(n) |
| Single range query | O(1) | — |
| Count subarrays (with `HashMap`) | O(n) | O(n) |
| 2D prefix sum build | O(m×n) | O(m×n) |

---

## Rust Template

### 1. Classic Prefix Sum (Range Query)

```rust
fn build_prefix(arr: &[i32]) -> Vec<i32> {
    let n = arr.len();
    let mut prefix = vec![0; n + 1]; // prefix[0] = 0
    for i in 0..n {
        prefix[i + 1] = prefix[i] + arr[i];
    }
    prefix
}

// Sum of arr[l..r] (0-indexed, inclusive)
fn range_sum(prefix: &[i32], l: usize, r: usize) -> i32 {
    prefix[r + 1] - prefix[l]
}
```

### 2. Count Subarrays with Sum = K

```rust
use std::collections::HashMap;

fn subarray_sum(nums: &[i32], k: i32) -> i32 {
    let mut prefix_count: HashMap<i32, i32> = HashMap::new();
    prefix_count.insert(0, 1); // empty prefix
    let mut sum = 0i32;
    let mut count = 0i32;

    for &num in nums {
        sum += num;
        // If (sum - k) was seen, those subarrays sum to k
        count += prefix_count.get(&(sum - k)).copied().unwrap_or(0);
        *prefix_count.entry(sum).or_insert(0) += 1;
    }
    count
}
// Time: O(n) | Space: O(n)
```

### 3. Largest Subarray with Sum 0

```rust
use std::collections::HashMap;

fn largest_subarray_with_zero_sum(arr: &[i32]) -> usize {
    let mut first_seen: HashMap<i32, i32> = HashMap::new();
    first_seen.insert(0, -1);
    let mut sum = 0i32;
    let mut max_len = 0usize;

    for (i, &val) in arr.iter().enumerate() {
        sum += val;
        if let Some(&j) = first_seen.get(&sum) {
            max_len = max_len.max((i as i32 - j) as usize);
        } else {
            first_seen.insert(sum, i as i32);
        }
    }
    max_len
}
// Time: O(n) | Space: O(n)
```

### 4. 2D Prefix Sum

```rust
fn build_2d_prefix(matrix: &[Vec<i32>]) -> Vec<Vec<i32>> {
    let m = matrix.len();
    let n = matrix[0].len();
    let mut p = vec![vec![0i32; n + 1]; m + 1];
    for i in 1..=m {
        for j in 1..=n {
            p[i][j] = matrix[i-1][j-1] + p[i-1][j] + p[i][j-1] - p[i-1][j-1];
        }
    }
    p
}

// Sum of submatrix (r1,c1) to (r2,c2) — 0-indexed
fn query_2d(p: &[Vec<i32>], r1: usize, c1: usize, r2: usize, c2: usize) -> i32 {
    p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]
}
// Time: O(1) per query after O(m*n) build
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Off-by-one: `prefix[i] = arr[0..i]` vs `arr[0..i-1]` | Use 1-indexed prefix: `prefix[0]=0`, `prefix[i]=prefix[i-1]+arr[i-1]` |
| Forgetting to seed `prefix_count[0] = 1` | Always insert the empty prefix before the loop |
| Integer overflow on large arrays | Use `i64` for sum accumulation |
| Modifying prefix array while answering queries | Prefix is read-only after build |
| 2D prefix: forgetting the inclusion-exclusion formula | `p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]` |

---

## Variations

| Variation | Description | Key Change |
|-----------|-------------|------------|
| XOR Prefix | `prefix[i] = arr[0] XOR ... XOR arr[i-1]` | Replace `+` with `^` |
| Product Prefix | Prefix products for range product queries | Divide instead of subtract |
| Difference Array | Range increment/decrement in O(1) | Inverse of prefix sum |
| Running Maximum | Track max prefix so far | Use `max` instead of `+` |
| Subarray with sum divisible by K | Prefix mod K + `HashMap` | Store `sum % k` |

### Subarray Sum Divisible by K

```rust
use std::collections::HashMap;

fn subarrays_div_by_k(nums: &[i32], k: i32) -> i32 {
    let mut mod_count: HashMap<i32, i32> = HashMap::new();
    mod_count.insert(0, 1);
    let mut sum = 0i32;
    let mut count = 0i32;
    for &num in nums {
        sum = ((sum + num) % k + k) % k; // handle negatives
        count += mod_count.get(&sum).copied().unwrap_or(0);
        *mod_count.entry(sum).or_insert(0) += 1;
    }
    count
}
```

---

## Practice Problems

| Problem | Difficulty | LeetCode | Pattern |
|---------|-----------|----------|---------|
| [Range Sum Query - Immutable](https://leetcode.com/problems/range-sum-query-immutable/) | Easy | LC 303 | Classic Prefix |
| [Subarray Sum Equals K](https://leetcode.com/problems/subarray-sum-equals-k/) | Medium | LC 560 | Prefix + `HashMap` |
| [Contiguous Array](https://leetcode.com/problems/contiguous-array/) | Medium | LC 525 | Prefix (0→-1 trick) |
| [Subarray Sums Divisible by K](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | Medium | LC 974 | Prefix mod |
| [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | Medium | LC 1248 | Prefix (odd→1) |
| [Maximum Size Subarray Sum Equals k](https://leetcode.com/problems/maximum-size-subarray-sum-equals-k/) | Medium | LC 325 | Prefix + first-seen |
| [Range Sum Query 2D - Immutable](https://leetcode.com/problems/range-sum-query-2d-immutable/) | Medium | LC 304 | 2D Prefix |
| Largest Subarray with 0 Sum | Medium | GFG | Prefix + first-seen |
| Count Subarrays with XOR = K | Medium | InterviewBit | XOR Prefix |

---

## Related Patterns

- [Sliding Window](./Sliding%20Window.md) — for variable-size subarray problems with constraints
- [Kadane's Algorithm](./Kadane's%20Algorithm.md) — for maximum subarray sum (no target K)
- [Two Pointers](./Two%20Pointers.md) — when all values are non-negative (monotone prefix)

---

> **Interview Tip:** When a problem says "count subarrays with property X", immediately think Prefix Sum + `HashMap`. The map stores seen prefix values; the check is `prefix[j] - target` already in map.

> **Last Updated:** 2026-06-26
