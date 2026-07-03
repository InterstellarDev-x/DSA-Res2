# Moore's Voting Algorithm Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `majority-element` `voting` `constant-space` `linear`

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

Moore's Voting Algorithm finds a **majority element** (appearing more than ⌊n/2⌋ times) in O(n) time and O(1) space.

**Core Insight:** If we cancel out pairs of distinct elements, the majority element survives — because it outnumbers all others combined.

**Phase 1 — Find Candidate:**
```
candidate = nums[0], count = 1
For each num:
    if count == 0 → candidate = num, count = 1
    else if num == candidate → count++
    else → count--
```

**Phase 2 — Verify Candidate:** (required when majority is not guaranteed)
Count actual occurrences of `candidate` and confirm `> n/2`.

---

## When to Use

- Find element appearing more than n/2 times
- Find all elements appearing more than n/3 times
- Majority vote in streams (constant memory)

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "majority element (> n/2)" | Classic Boyer-Moore |
| "elements appearing more than n/3 times" | Extended Moore (2 candidates) |
| "dominant element in O(1) space" | Moore's Voting |
| "find element with frequency > k" | Extended to k−1 candidates |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Majority > n/2 | O(n) | O(1) |
| Majority > n/3 (at most 2 elements) | O(n) | O(1) |
| Majority > n/k (at most k-1 elements) | O(n×k) | O(k) |

---

## Rust Templates

### 1. Majority Element (> n/2) — Guaranteed to Exist

```rust
fn majority_element(nums: &[i32]) -> i32 {
    let mut candidate = nums[0];
    let mut count = 1;

    for i in 1..nums.len() {
        if count == 0 {
            candidate = nums[i];
            count = 1;
        } else if nums[i] == candidate {
            count += 1;
        } else {
            count -= 1;
        }
    }
    candidate // guaranteed to be majority
}
// Time: O(n) | Space: O(1)
```

### 2. Majority Element (> n/2) — Not Guaranteed (with Verification)

```rust
fn majority_element(nums: &[i32]) -> i32 {
    let mut candidate = nums[0];
    let mut count = 1;
    for i in 1..nums.len() {
        if count == 0 { candidate = nums[i]; count = 1; }
        else if nums[i] == candidate { count += 1; }
        else { count -= 1; }
    }
    // Verify
    let freq = nums.iter().filter(|&&num| num == candidate).count();
    if freq > nums.len() / 2 { candidate } else { -1 }
}
```

### 3. Majority Element II (> n/3) — At Most 2 Candidates

```rust
fn majority_element(nums: &[i32]) -> Vec<i32> {
    let mut cand1 = 0;
    let mut cand2 = 0;
    let mut count1 = 0i32;
    let mut count2 = 0i32;

    for &num in nums {
        if num == cand1 {
            count1 += 1;
        } else if num == cand2 {
            count2 += 1;
        } else if count1 == 0 {
            cand1 = num; count1 = 1;
        } else if count2 == 0 {
            cand2 = num; count2 = 1;
        } else {
            count1 -= 1; count2 -= 1;
        }
    }

    // Verify both candidates
    count1 = 0; count2 = 0;
    for &num in nums {
        if num == cand1 { count1 += 1; }
        else if num == cand2 { count2 += 1; }
    }

    let mut result = Vec::new();
    let threshold = (nums.len() / 3) as i32;
    if count1 > threshold { result.push(cand1); }
    if count2 > threshold { result.push(cand2); }
    result
}
// Time: O(n) | Space: O(1)
```

### 4. General: Majority > n/k

```rust
use std::collections::HashMap;

fn majority_element_k(nums: &[i32], k: usize) -> Vec<i32> {
    // At most (k-1) candidates can have frequency > n/k
    let mut candidates: HashMap<i32, i32> = HashMap::new();

    for &num in nums {
        *candidates.entry(num).or_insert(0) += 1;
        if candidates.len() == k {
            candidates.retain(|_, val| {
                *val -= 1;
                *val > 0
            });
        }
    }
    // Verify
    for val in candidates.values_mut() {
        *val = 0;
    }
    for &num in nums {
        if let Some(val) = candidates.get_mut(&num) {
            *val += 1;
        }
    }
    let threshold = (nums.len() / k) as i32;
    candidates
        .into_iter()
        .filter(|&(_, val)| val > threshold)
        .map(|(key, _)| key)
        .collect()
}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Skipping verification phase | Always verify unless problem guarantees majority exists |
| n/3 variant: checking `num == cand1` BEFORE `count1 == 0` | Order matters: check existing candidates first |
| Modifying count before assigning new candidate | Set `count = 1` immediately after `candidate = num` |
| Off-by-one: `> n/2` vs `>= n/2` | Majority = **strictly** more than half |
| n/3 variant: both candidates could be the same | Not possible if you check `num == cand1` first |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Check if majority exists | Run Phase 1, then count — return -1 if not found |
| Find all elements with freq > n/k | Use `HashMap` of k-1 candidates |
| Majority in stream | Classic Moore's — O(1) memory even for infinite streams |
| Weighted majority | Adjust count by weight |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Majority Element](https://leetcode.com/problems/majority-element/) | Easy | LC 169 |
| [Majority Element II](https://leetcode.com/problems/majority-element-ii/) | Medium | LC 229 |
| [Check if a Number is Majority Element in Sorted Array](https://leetcode.com/problems/check-if-a-number-is-majority-element-in-a-sorted-array/) | Easy | LC 1150 |
| Find Majority Element in Row-wise Sorted Matrix | Medium | GFG |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — for sorted array majority check using binary search
- [Prefix Sum](./Prefix%20Sum.md) — counting occurrences
- [Bit Manipulation](../../6.Bit_Manipulation/README.md) — majority in bit-by-bit fashion

---

> **Interview Tip:** Phase 2 (verification) is almost always asked as a follow-up: "What if no majority element exists?" Always code the verification unless the problem explicitly guarantees one exists. For n/3, there can be **0, 1, or 2** results — return a list.

> **Last Updated:** 2026-06-26
