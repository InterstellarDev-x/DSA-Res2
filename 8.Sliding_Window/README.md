# Sliding Window & Two Pointers

> **Topic 10 of 18** — 12 problems (1E / 9M / 2H)
> Java-centric, pattern-driven reference for interview preparation.
> [← Stacks & Queues](../7.Stacks_and_Queues/README.md) · [→ Heaps](../9.Heaps/README.md)

---

## Quick Navigation

| Section | Files |
|---------|-------|
| [Patterns](#patterns) | Fixed Size Window · Variable Size Window · At Most K → Exactly K · Two Pointers |
| [Design Problems](#design-data-structure-problems) | Sliding Window Rate Limiter |
| [OA Questions](#oa-questions) | Amazon · Google · Microsoft · Goldman Sachs · Adobe |
| [Interview Problems](#interview-problems) | Amazon · Google · Microsoft |
| [Interview Tips](#interview-tips) | Coding Tips · Common Mistakes · Window Template · Complexity Analysis |
| [Recent Questions](#most-recent-questions) | 2024 · 2025 · 2026 |

---

## Problem List

| # | Problem | Difficulty | Pattern | Companies |
|---|---------|-----------|---------|-----------|
| 1 | [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Easy | [Fixed Window](./Patterns/Fixed%20Size%20Window.md) | Amazon |
| 2 | [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Amazon, Google |
| 3 | [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Amazon, Google, Microsoft |
| 4 | [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Google |
| 5 | [Fruits Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Amazon, Google |
| 6 | [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Amazon, Google |
| 7 | [Binary Subarrays With Sum](https://leetcode.com/problems/binary-subarrays-with-sum/) | Medium | [At Most K Trick](./Patterns/At%20Most%20K%20Trick.md) | Google |
| 8 | [Count Number of Nice Subarrays](https://leetcode.com/problems/count-number-of-nice-subarrays/) | Medium | [At Most K Trick](./Patterns/At%20Most%20K%20Trick.md) | Amazon |
| 9 | [Number of Substrings Containing All Three Characters](https://leetcode.com/problems/number-of-substrings-containing-all-three-characters/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Google |
| 10 | [Subarray Product Less Than K](https://leetcode.com/problems/subarray-product-less-than-k/) | Medium | [At Most K Trick](./Patterns/At%20Most%20K%20Trick.md) | Amazon, Microsoft |
| 11 | [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Amazon, Google, Microsoft |
| 12 | [Longest Subarray of 1s After Deleting One Element](https://leetcode.com/problems/longest-subarray-of-1s-after-deleting-one-element/) | Medium | [Variable Window](./Patterns/Variable%20Size%20Window.md) | Google |

---

## Patterns

| Pattern | Key Problems | Core Idea |
|---------|-------------|-----------|
| [Fixed Size Window](./Patterns/Fixed%20Size%20Window.md) | Max Average, Sliding Window Max | Slide window of fixed size k; add right, remove left |
| [Variable Size Window](./Patterns/Variable%20Size%20Window.md) | Min Subarray Sum, LSWORC, Min Window Substr | Expand right; shrink left when constraint violated |
| [At Most K → Exactly K](./Patterns/At%20Most%20K%20Trick.md) | Binary Subarrays, Nice Subarrays, Product < K | `exactly(k) = atMost(k) - atMost(k-1)` |
| [Two Pointers](./Patterns/Two%20Pointers.md) | Container with Most Water, 3Sum, Move Zeroes | Opposite-end or same-direction pointer movement |

---

## Design Data Structure Problems

| Design | Key Operations | Complexity |
|--------|---------------|------------|
| [Sliding Window Rate Limiter](./Design%20Data%20Structure%20Problems/Rate%20Limiter.md) | `allow(timestamp)` — O(1) amortized | Queue-based sliding window |

---

## Company Coverage

| Company | Top OA Problems | Top Interview Problems |
|---------|----------------|------------------------|
| Amazon | LSWORC ⭐⭐⭐⭐⭐, Min Window Substr ⭐⭐⭐⭐, Nice Subarrays | Fruits Into Baskets, Char Replacement |
| Google | Min Window Substr ⭐⭐⭐⭐⭐, Max Cons Ones III ⭐⭐⭐⭐ | Binary Subarrays, All Three Chars |
| Microsoft | LSWORC ⭐⭐⭐⭐, Min Subarray Sum ⭐⭐⭐ | Min Window Substr, Product < K |
| Goldman Sachs | Max Average ⭐⭐⭐, LSWORC | Subarrays with Sum |
| Adobe | LSWORC ⭐⭐⭐, Char Replacement | Fruits Into Baskets |

---

## OA Questions

- [Amazon OA](./OA-Qns/Amazon.md)
- [Google OA](./OA-Qns/Google.md)
- [Microsoft OA](./OA-Qns/Microsoft.md)
- [Goldman Sachs OA](./OA-Qns/Goldman%20Sachs.md)
- [Adobe OA](./OA-Qns/Adobe.md)

---

## Interview Problems

- [Amazon](./Interview%20Problems/Amazon.md)
- [Google](./Interview%20Problems/Google.md)
- [Microsoft](./Interview%20Problems/Microsoft.md)

---

## Interview Tips

- [Coding Tips](./Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](./Interview%20Tips/Common%20Mistakes.md)
- [Window Template Guide](./Interview%20Tips/Window%20Template.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)

---

## Most Recent Questions

- [2024](./Most%20Recent%20Questions/2024.md)
- [2025](./Most%20Recent%20Questions/2025.md)
- [2026](./Most%20Recent%20Questions/2026.md)

---

> **Last Updated:** 2026-06-26
> **Problems:** 12 (1E / 9M / 2H)
> [← Back to Root](../README.md)
