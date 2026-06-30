# Amazon — Sliding Window & Two Pointers OA Questions

> **Topic:** [Sliding Window](../README.md) · **Company:** Amazon

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | ⭐⭐⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 3 |
| 2 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | ⭐⭐⭐⭐ | Hard | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 76 |
| 3 | [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | ⭐⭐⭐ | Medium | [At Most K](../Patterns/At%20Most%20K%20Trick.md) | LC 1248 |
| 4 | [Fruits Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | ⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 904 |
| 5 | [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | ⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 424 |
| 6 | [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | ⭐⭐⭐ | Medium | [At Most K](../Patterns/At%20Most%20K%20Trick.md) | LC 713 |
| 7 | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | ⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 209 |
| 8 | [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | ⭐⭐ | Easy | [Fixed Window](../Patterns/Fixed%20Size%20Window.md) | LC 643 |

---

## Top 3 Must-Know for Amazon

### 1. Longest Substring Without Repeating Characters ⭐⭐⭐⭐⭐
Amazon's most-asked sliding window question. Appears in SDE OA and phone screens.
- Frequency array (O(1) space) preferred over HashMap for clean code
- Follow-up: Count of distinct characters, at most 2 distinct types

### 2. Minimum Window Substring ⭐⭐⭐⭐
Amazon system design rounds ("find the smallest log window containing all error types").
- `need[c] > 0` for `have++` is the key — understand why excess chars don't count
- Follow-up: Return all such windows, not just the smallest

### 3. Count Nice Subarrays ⭐⭐⭐
Amazon OA framed as "count contiguous subarrays with exactly k special elements."
- `atMost(k) - atMost(k-1)` is the key technique

---

## Related Files

- [Amazon Interview Problems](../Interview%20Problems/Amazon.md)
- [Google OA](./Google.md)

> **Last Updated:** 2026-06-26
