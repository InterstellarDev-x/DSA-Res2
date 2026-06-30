# Arrays

> **Step 3 of 18** on [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> 40 problems · Easy → Medium → Hard

---

## Table of Contents

1. [Topic Overview](#topic-overview)
2. [Patterns in this Topic](#patterns-in-this-topic)
3. [Problem List by Difficulty](#problem-list-by-difficulty)
4. [Design Problems](#design-problems)
5. [Company Coverage](#company-coverage)
6. [Interview Tips](#interview-tips)
7. [Navigation](#navigation)

---

## Topic Overview

Arrays are the most fundamental data structure and appear in virtually every technical interview. Mastery of arrays requires recognizing 8 core patterns that reduce seemingly distinct problems to known templates.

**Key skills tested:**
- In-place manipulation
- Index arithmetic
- Prefix/suffix computations
- Sorting-based tricks
- Two-pass and constant-space algorithms

---

## Patterns in this Topic

| Pattern | Difficulty | Problems |
|---------|-----------|----------|
| [Prefix Sum](./Patterns/Prefix%20Sum.md) | Easy–Medium | Subarray sum = K, Range queries |
| [Sliding Window](./Patterns/Sliding%20Window.md) | Medium | Max sum subarray of size K |
| [Two Pointers](./Patterns/Two%20Pointers.md) | Easy–Medium | Two Sum, 3Sum, Container with Water |
| [Kadane's Algorithm](./Patterns/Kadane's%20Algorithm.md) | Medium | Max subarray sum, Max product |
| [Dutch National Flag](./Patterns/Dutch%20National%20Flag.md) | Medium | Sort Colors, Partition |
| [Merge Intervals](./Patterns/Merge%20Intervals.md) | Medium–Hard | Overlapping intervals |
| [Moore's Voting](./Patterns/Moore's%20Voting.md) | Medium | Majority element |
| [Cyclic Sort](./Patterns/Cyclic%20Sort.md) | Medium | Missing/duplicate numbers |

---

## Problem List by Difficulty

### Easy

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Largest Element in Array | Traversal | — |
| 2 | Second Largest Element | Traversal | — |
| 3 | Check Sorted Array | Traversal | [LC 896](https://leetcode.com/problems/monotonic-array/) |
| 4 | Remove Duplicates from Sorted Array | Two Pointers | [LC 26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) |
| 5 | Left Rotate Array by One | In-place | — |
| 6 | Left Rotate Array by K Places | In-place | [LC 189](https://leetcode.com/problems/rotate-array/) |
| 7 | Move Zeros to End | Two Pointers | [LC 283](https://leetcode.com/problems/move-zeroes/) |
| 8 | Linear Search | Traversal | — |
| 9 | Union of Two Sorted Arrays | Two Pointers | — |
| 10 | Intersection of Two Sorted Arrays | Two Pointers | [LC 349](https://leetcode.com/problems/intersection-of-two-arrays/) |
| 11 | Find Missing Number | Cyclic Sort / Math | [LC 268](https://leetcode.com/problems/missing-number/) |
| 12 | Maximum Consecutive Ones | Sliding Window | [LC 485](https://leetcode.com/problems/max-consecutive-ones/) |
| 13 | Find Number appearing once | Bit Manipulation | [LC 136](https://leetcode.com/problems/single-number/) |
| 14 | Longest Subarray with Sum K (Positives) | Prefix Sum | — |

### Medium

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Two Sum | Two Pointers / HashMap | [LC 1](https://leetcode.com/problems/two-sum/) |
| 2 | Sort Array of 0s, 1s, 2s | Dutch National Flag | [LC 75](https://leetcode.com/problems/sort-colors/) |
| 3 | Majority Element (> n/2) | Moore's Voting | [LC 169](https://leetcode.com/problems/majority-element/) |
| 4 | Kadane's Algorithm | Kadane | [LC 53](https://leetcode.com/problems/maximum-subarray/) |
| 5 | Stock Buy & Sell I | Greedy | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| 6 | Rearrange +ve and -ve | Two Pointers | [LC 2149](https://leetcode.com/problems/rearrange-array-elements-by-sign/) |
| 7 | Next Permutation | In-place | [LC 31](https://leetcode.com/problems/next-permutation/) |
| 8 | Leaders in an Array | Suffix max | — |
| 9 | Longest Consecutive Sequence | HashSet | [LC 128](https://leetcode.com/problems/longest-consecutive-sequence/) |
| 10 | Set Matrix Zeroes | In-place markers | [LC 73](https://leetcode.com/problems/set-matrix-zeroes/) |
| 11 | Rotate Matrix 90° | In-place | [LC 48](https://leetcode.com/problems/rotate-image/) |
| 12 | Print Matrix Spiral | Simulation | [LC 54](https://leetcode.com/problems/spiral-matrix/) |
| 13 | Count Subarrays with XOR = K | Prefix XOR | — |
| 14 | Merge Overlapping Intervals | Merge Intervals | [LC 56](https://leetcode.com/problems/merge-intervals/) |
| 15 | Merge Sorted Arrays | Two Pointers | [LC 88](https://leetcode.com/problems/merge-sorted-array/) |
| 16 | Find Duplicate | Floyd's Cycle | [LC 287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 17 | Count Inversions | Merge Sort | — |
| 18 | Search in 2D Matrix | Binary Search | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) |

### Hard

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Pascal's Triangle | DP / Math | [LC 118](https://leetcode.com/problems/pascals-triangle/) |
| 2 | Majority Element (> n/3) | Moore's Voting | [LC 229](https://leetcode.com/problems/majority-element-ii/) |
| 3 | 3Sum | Two Pointers | [LC 15](https://leetcode.com/problems/3sum/) |
| 4 | 4Sum | Two Pointers | [LC 18](https://leetcode.com/problems/4sum/) |
| 5 | Largest Subarray with 0 Sum | Prefix Sum + HashMap | — |
| 6 | Maximum Product Subarray | Kadane Variant | [LC 152](https://leetcode.com/problems/maximum-product-subarray/) |
| 7 | Grid Unique Paths | DP | [LC 62](https://leetcode.com/problems/unique-paths/) |
| 8 | Reverse Pairs | Merge Sort | [LC 493](https://leetcode.com/problems/reverse-pairs/) |

---

## Design Problems

| Problem | File |
|---------|------|
| Design Dynamic Array | [Dynamic Array.md](./Design%20Data%20Structure%20Problems/Dynamic%20Array.md) |
| Design Sparse Table (Range Min Query) | [Sparse Table.md](./Design%20Data%20Structure%20Problems/Sparse%20Table.md) |

---

## Company Coverage

| Company | OA Questions | Interview Questions |
|---------|-------------|---------------------|
| Amazon | [Amazon OA](./OA-Qns/Amazon.md) | [Amazon Interview](./Interview%20Problems/Amazon.md) |
| Google | [Google OA](./OA-Qns/Google.md) | [Google Interview](./Interview%20Problems/Google.md) |
| Microsoft | [Microsoft OA](./OA-Qns/Microsoft.md) | [Microsoft Interview](./Interview%20Problems/Microsoft.md) |
| Adobe | [Adobe OA](./OA-Qns/Adobe.md) | — |
| Goldman Sachs | [Goldman OA](./OA-Qns/Goldman%20Sachs.md) | — |
| Flipkart | [Flipkart OA](./OA-Qns/Flipkart.md) | — |
| Uber | [Uber OA](./OA-Qns/Uber.md) | — |
| Walmart | [Walmart OA](./OA-Qns/Walmart.md) | — |

---

## Interview Tips

- [Coding Tips for Arrays](./Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](./Interview%20Tips/Common%20Mistakes.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)
- [Dry Run Technique](./Interview%20Tips/Dry%20Run.md)

---

## Navigation

| Previous | Home | Next |
|----------|------|------|
| [Sorting Techniques](../Sorting/README.md) | [Repository Root](../README.md) | [Binary Search](../2.Binary_Search/README.md) |

---

> **Last Updated:** 2026-06-26
> **Tags:** `arrays` `two-pointers` `prefix-sum` `kadane` `dutch-national-flag`
