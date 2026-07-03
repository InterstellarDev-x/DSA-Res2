# Amazon — Binary Search Interview Questions

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Problems
> **Tags:** `amazon` `interview` `binary-search`

---

## Table of Contents

1. [What Amazon Checks](#what-amazon-checks)
2. [Frequently Asked Questions](#frequently-asked-questions)
3. [Related Files](#related-files)

---

## What Interviewers Usually Check

| Dimension | What They Want |
|-----------|---------------|
| **Pattern identification** | Recognise "minimize maximum" → binary search on answer within 2 min |
| **Feasibility check** | Write a clean, correct `canAchieve(mid)` function |
| **Boundary conditions** | Correct `lo`/`hi` initialisation — no missed answers |
| **Overflow** | `mid = lo + (hi-lo)/2`, use `long` for sums |
| **Edge cases** | k > array length, empty array, single element |
| **Communication** | Explain why binary search works: "the answer space is monotone" |

---

## Frequently Asked Questions

### 1. Koko Eating Bananas

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | OA, Round 1 |
| **Skill tested** | Binary search on answer, ceil division |
| **Pattern** | [BS on Answer](../Patterns/Binary%20Search%20on%20Answer.md) |
| **LeetCode** | [LC 875](https://leetcode.com/problems/koko-eating-bananas/) |

**Follow-ups:**
- "What's the time complexity?" → O(n log maxPile)
- "What if piles can be 0?" → Skip zeros; ceil(0/speed) = 0
- "Can you generalise this to any resource allocation?" → Yes — same feasibility template

---

### 2. Capacity to Ship Packages in D Days

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | OA, Round 1 |
| **Skill tested** | BS on Answer, greedy feasibility scan |
| **Pattern** | [BS on Answer](../Patterns/Binary%20Search%20on%20Answer.md) |
| **LeetCode** | [LC 1011](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) |

**Follow-ups:**
- "Why is `lo = max(weights)`?" → Single heaviest package must fit
- "What if order doesn't matter?" → Sort by weight, bin packing (NP-hard generally)

---

### 3. Split Array Largest Sum

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Round 2, Bar Raiser |
| **Skill tested** | BS on Answer — identical feasibility to LC 1011 |
| **Pattern** | [BS on Answer](../Patterns/Binary%20Search%20on%20Answer.md) |
| **LeetCode** | [LC 410](https://leetcode.com/problems/split-array-largest-sum/) |
| **Variants** | Book Allocation (GFG), Painter's Partition (GFG) — same solution |

**Rust solution:**
```rust
fn split_array(nums: &[i32], k: i32) -> i32 {
    let mut lo = *nums.iter().max().unwrap();
    let mut hi: i32 = nums.iter().sum();
    while lo < hi {
        let mid = lo + (hi - lo) / 2;
        let mut parts = 1;
        let mut cur = 0;
        for &n in nums {
            if cur + n > mid {
                parts += 1;
                cur = 0;
            }
            cur += n;
        }
        if parts <= k {
            hi = mid;
        } else {
            lo = mid + 1;
        }
    }
    lo
}
// Time: O(n log(sum)) | Space: O(1)
```

---

### 4. Search in Rotated Sorted Array

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone screen, Round 1 |
| **Skill tested** | Determine sorted half, then range check |
| **Pattern** | [Classic Binary Search](../Patterns/Classic%20Binary%20Search.md) |
| **LeetCode** | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |

**Follow-ups:**
- "What if duplicates exist?" → LC 81 — add `lo++; hi--` when boundaries equal
- "Find minimum instead?" → LC 153

---

## Related Files

- [Amazon OA Questions](../OA-Qns/Amazon.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)
- [BS on Answer Pattern](../Patterns/Binary%20Search%20on%20Answer.md)
- [Interview Tips — Identifying Search Space](../Interview%20Tips/Identifying%20Search%20Space.md)

> **Last Updated:** 2026-06-26
