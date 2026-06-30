# Google — Sliding Window & Two Pointers OA Questions

> **Topic:** [Sliding Window](../README.md) · **Company:** Google

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | ⭐⭐⭐⭐⭐ | Hard | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 76 |
| 2 | [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | ⭐⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 1004 |
| 3 | [Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) | ⭐⭐⭐ | Medium | [At Most K](../Patterns/At%20Most%20K%20Trick.md) | LC 930 |
| 4 | [Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/) | ⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 1358 |
| 5 | [Fruits Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | ⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 904 |
| 6 | [Longest Subarray of 1s After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) | ⭐⭐⭐ | Medium | [Variable Window](../Patterns/Variable%20Size%20Window.md) | LC 1493 |

---

## Top 3 Must-Know for Google

### 1. Minimum Window Substring ⭐⭐⭐⭐⭐
Google's canonical hard sliding window problem. The `have/need` counter pattern is essential.

### 2. Max Consecutive Ones III ⭐⭐⭐⭐
Google frames this as "given a binary string, flip at most k bits — longest run of 1s."
- Reframe: window with at most k zeros

### 3. Number of Substrings Containing All Three Characters ⭐⭐⭐
The `result += n - right` insight (counting all right extensions at once) separates good from great solutions.

---

## Related Files

- [Google Interview Problems](../Interview%20Problems/Google.md)
- [Amazon OA](./Amazon.md)
- [Microsoft OA](./Microsoft.md)

> **Last Updated:** 2026-06-26
