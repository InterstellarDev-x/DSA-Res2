# Optimization Tips — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `optimization` `time-complexity` `space-complexity` `interview`

---

## Table of Contents

1. [Optimization Strategy](#optimization-strategy)
2. [Time Complexity Upgrades](#time-complexity-upgrades)
3. [Space Complexity Upgrades](#space-complexity-upgrades)
4. [Common Optimization Techniques](#common-optimization-techniques)
5. [When NOT to Optimize](#when-not-to-optimize)
6. [Related Files](#related-files)

---

## Optimization Strategy

**Step-by-step:**

1. **Start with brute force** — state it, but don't code it unless asked
2. **Identify the bottleneck** — is it the nested loop? The sort? The space?
3. **Ask: what extra information can I precompute?** — prefix sums, frequency maps
4. **Ask: can I avoid recomputation?** — sliding window, DP
5. **Ask: can I trade space for time?** — `HashMap` / `HashSet`
6. **Ask: can I eliminate space?** — in-place manipulation, cyclic sort

---

## Time Complexity Upgrades

| From | To | Technique | Example |
|------|----|-----------|---------|
| O(n²) pair check | O(n) | `HashMap` | Two Sum |
| O(n²) subarray sum | O(n) | [Prefix Sum + `HashMap`](../Patterns/Prefix%20Sum.md) | Subarray Sum = K |
| O(n²) window check | O(n) | [Sliding Window](../Patterns/Sliding%20Window.md) | Longest substring |
| O(n² log n) 3Sum naive | O(n²) | Sort + [Two Pointers](../Patterns/Two%20Pointers.md) | 3Sum |
| O(n log n) merge-find | O(n) | [Cyclic Sort](../Patterns/Cyclic%20Sort.md) | Missing number |
| O(n log n) sort-based majority | O(n) | [Moore's Voting](../Patterns/Moore's%20Voting.md) | Majority element |
| O(n) repeated max scans | O(1) per query | [Sparse Table](../Design%20Data%20Structure%20Problems/Sparse%20Table.md) | RMQ |

---

## Space Complexity Upgrades

| From | To | Technique | Example |
|------|----|-----------|---------|
| O(n) copy for reverse | O(1) | In-place two-pointer | Reverse array |
| O(n) hash set for duplicates | O(1) | Cyclic sort / Floyd's cycle | Find duplicate |
| O(n) prefix array | O(1) | Running variable | Stock buy/sell |
| O(n) result array | O(1) | Encode in input | Set Matrix Zeroes |
| O(n) stack for monotonic | O(1) | Two-pointer variant | Trapping Rain Water |

### Set Matrix Zeroes — O(1) Space Optimization

```rust
// Use first row and first column as markers
let mut first_row_zero = false;
let mut first_col_zero = false;
// Check if first row/col have zeros originally
for j in 0..cols {
    if matrix[0][j] == 0 { first_row_zero = true; }
}
for i in 0..rows {
    if matrix[i][0] == 0 { first_col_zero = true; }
}
// Mark zeros using first row/col
for i in 1..rows {
    for j in 1..cols {
        if matrix[i][j] == 0 {
            matrix[i][0] = 0;
            matrix[0][j] = 0;
        }
    }
}
// Apply markers
for i in 1..rows {
    for j in 1..cols {
        if matrix[i][0] == 0 || matrix[0][j] == 0 {
            matrix[i][j] = 0;
        }
    }
}
// Handle first row and col
if first_row_zero {
    for j in 0..cols { matrix[0][j] = 0; }
}
if first_col_zero {
    for i in 0..rows { matrix[i][0] = 0; }
}
```

---

## Common Optimization Techniques

### 1. Precompute Prefix/Suffix Values

Instead of scanning left or right at each step, precompute:

```rust
// Suffix maximum for Trapping Rain Water
let mut right_max = vec![0i32; n];
right_max[n - 1] = height[n - 1];
for i in (0..n - 1).rev() {
    right_max[i] = height[i].max(right_max[i + 1]);
}
```

### 2. Use Complement in `HashMap`

Instead of "find pair that sums to target", store "what I've seen" and check "target - current":

```rust
use std::collections::HashMap;

// O(n) instead of O(n²)
let mut seen: HashMap<i32, usize> = HashMap::new();
for i in 0..nums.len() {
    let complement = target - nums[i];
    if let Some(&j) = seen.get(&complement) {
        return vec![j as i32, i as i32];
    }
    seen.insert(nums[i], i);
}
```

### 3. Eliminate Sort with `HashSet`

For "Longest Consecutive Sequence":
- Naive: sort then scan → O(n log n)
- Optimal: `HashSet` + only extend from sequence start → O(n)

### 4. Early Termination

```rust
// If max so far is already at theoretical maximum, stop
if max_len == n { return max_len; }
// In sorted array, if current element > target/3, no valid triplet possible
if nums[i] > 0 && nums[i] * 3 > target { break; }
```

---

## When NOT to Optimize

| Scenario | Reason |
|----------|--------|
| O(n log n) when n ≤ 1000 | Premature optimization — code clarity matters more |
| Trading readability for 2× speedup | Maintainability > micro-optimization |
| When the bottleneck is I/O, not algorithm | Optimizing wrong thing |
| When interviewer hasn't asked | Finish a correct solution first |

> **Rule:** Get a correct solution working first. Optimize only after it's verified correct and the interviewer asks.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
