# Binary Search

> **Step 4 of 18** on [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> 32 problems · 1D Arrays · 2D Arrays · Search Space Reduction

---

## Table of Contents

1. [Topic Overview](#topic-overview)
2. [Patterns in this Topic](#patterns-in-this-topic)
3. [Problem List by Category](#problem-list-by-category)
4. [Design Problems](#design-problems)
5. [Company Coverage](#company-coverage)
6. [Interview Tips](#interview-tips)
7. [Navigation](#navigation)

---

## Topic Overview

Binary Search is not just "search in sorted array" — it is a **decision framework**: whenever the answer space is monotone (a predicate flips from false → true at some threshold), binary search finds that threshold in O(log n).

**Three levels of mastery:**

| Level | Skill | Example |
|-------|-------|---------|
| L1 | Classic search in sorted array | LC 704 |
| L2 | Search on answer / search space | Min days to make bouquets |
| L3 | Binary search on 2D / complex structures | Median of two sorted arrays |

**Core invariant to maintain in every problem:**
> At every step, the answer must lie within `[lo, hi]`. Never exclude a potential answer.

---

## Patterns in this Topic

| Pattern | Difficulty | Problems |
|---------|-----------|----------|
| [Classic Binary Search](./Patterns/Classic%20Binary%20Search.md) | Easy | Search in sorted, rotated sorted |
| [Lower & Upper Bound](./Patterns/Lower%20and%20Upper%20Bound.md) | Easy–Medium | Floor/ceil, insertion point |
| [Binary Search on Answer](./Patterns/Binary%20Search%20on%20Answer.md) | Medium–Hard | Min max problems, allocation |
| [Binary Search on 2D](./Patterns/Binary%20Search%20on%202D.md) | Medium | Row-wise sorted, search matrix |
| [Peak Finding](./Patterns/Peak%20Finding.md) | Medium | Peak element, mountain array |

---

## Problem List by Category

### 1D Binary Search

| # | Problem | Difficulty | Pattern | LeetCode |
|---|---------|-----------|---------|----------|
| 1 | Binary Search | Easy | Classic | [LC 704](https://leetcode.com/problems/binary-search/) |
| 2 | Lower Bound | Easy | Lower/Upper Bound | — |
| 3 | Upper Bound | Easy | Lower/Upper Bound | — |
| 4 | Search Insert Position | Easy | Lower Bound | [LC 35](https://leetcode.com/problems/search-insert-position/) |
| 5 | Floor / Ceil in Sorted Array | Easy | Lower/Upper Bound | GFG |
| 6 | First and Last Occurrence | Easy | Lower/Upper Bound | [LC 34](https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/) |
| 7 | Count Occurrences | Easy | Lower/Upper Bound | GFG |
| 8 | Search in Rotated Sorted Array I | Medium | Classic Variant | [LC 33](https://leetcode.com/problems/search-in-rotated-sorted-array/) |
| 9 | Search in Rotated Sorted Array II (duplicates) | Medium | Classic Variant | [LC 81](https://leetcode.com/problems/search-in-rotated-sorted-array-ii/) |
| 10 | Find Minimum in Rotated Sorted Array | Medium | Classic Variant | [LC 153](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/) |
| 11 | Find out how many times array is rotated | Medium | Classic Variant | GFG |
| 12 | Single Element in Sorted Array | Medium | Classic Variant | [LC 540](https://leetcode.com/problems/single-element-in-a-sorted-array/) |
| 13 | Find Peak Element | Medium | [Peak Finding](./Patterns/Peak%20Finding.md) | [LC 162](https://leetcode.com/problems/find-peak-element/) |

### Binary Search on Answer (Search Space)

| # | Problem | Difficulty | Pattern | LeetCode |
|---|---------|-----------|---------|----------|
| 1 | Find Square Root | Easy | BS on Answer | [LC 69](https://leetcode.com/problems/sqrtx/) |
| 2 | Find Nth Root of M | Easy | BS on Answer | GFG |
| 3 | Koko Eating Bananas | Medium | BS on Answer | [LC 875](https://leetcode.com/problems/koko-eating-bananas/) |
| 4 | Minimum Days to Make M Bouquets | Medium | BS on Answer | [LC 1482](https://leetcode.com/problems/minimum-number-of-days-to-make-m-bouquets/) |
| 5 | Find the Smallest Divisor | Medium | BS on Answer | [LC 1283](https://leetcode.com/problems/find-the-smallest-divisor-given-a-threshold/) |
| 6 | Capacity to Ship Packages in D Days | Medium | BS on Answer | [LC 1011](https://leetcode.com/problems/capacity-to-ship-packages-within-d-days/) |
| 7 | Kth Missing Positive Number | Easy | BS on Answer | [LC 1539](https://leetcode.com/problems/kth-missing-positive-number/) |
| 8 | Aggressive Cows | Medium | BS on Answer | GFG / SPOJ |
| 9 | Book Allocation Problem | Medium | BS on Answer | GFG |
| 10 | Split Array Largest Sum | Hard | BS on Answer | [LC 410](https://leetcode.com/problems/split-array-largest-sum/) |
| 11 | Painter's Partition Problem | Hard | BS on Answer | GFG |
| 12 | Row with Max 1s | Easy | BS on Answer | GFG |
| 13 | Minimize Max Distance to Gas Station | Hard | BS on Answer | [LC 774](https://leetcode.com/problems/minimize-max-distance-to-gas-station/) |
| 14 | Median of Two Sorted Arrays | Hard | BS on Answer | [LC 4](https://leetcode.com/problems/median-of-two-sorted-arrays/) |
| 15 | Kth Element of Two Sorted Arrays | Hard | BS on Answer | GFG |

### Binary Search on 2D Arrays

| # | Problem | Difficulty | Pattern | LeetCode |
|---|---------|-----------|---------|----------|
| 1 | Search in a 2D Matrix | Medium | [BS on 2D](./Patterns/Binary%20Search%20on%202D.md) | [LC 74](https://leetcode.com/problems/search-a-2d-matrix/) |
| 2 | Search in a 2D Matrix II | Medium | [BS on 2D](./Patterns/Binary%20Search%20on%202D.md) | [LC 240](https://leetcode.com/problems/search-a-2d-matrix-ii/) |
| 3 | Find Peak Element II (2D) | Hard | Peak Finding | [LC 1901](https://leetcode.com/problems/find-a-peak-element-ii/) |
| 4 | Median in Row-wise Sorted Matrix | Hard | BS on Answer | GFG |

---

## Design Problems

| Problem | File |
|---------|------|
| Design Binary Search Tree | [BST Design](./Design%20Data%20Structure%20Problems/BST%20Design.md) |
| Design Order-Statistics Structure (Kth element) | [Order Statistics.md](./Design%20Data%20Structure%20Problems/Order%20Statistics.md) |

---

## Company Coverage

| Company | OA | Interview |
|---------|-----|-----------|
| Amazon | [Amazon OA](./OA-Qns/Amazon.md) | [Amazon Interview](./Interview%20Problems/Amazon.md) |
| Google | [Google OA](./OA-Qns/Google.md) | [Google Interview](./Interview%20Problems/Google.md) |
| Microsoft | [Microsoft OA](./OA-Qns/Microsoft.md) | [Microsoft Interview](./Interview%20Problems/Microsoft.md) |
| Goldman Sachs | [Goldman OA](./OA-Qns/Goldman%20Sachs.md) | — |
| Adobe | [Adobe OA](./OA-Qns/Adobe.md) | — |
| Flipkart | [Flipkart OA](./OA-Qns/Flipkart.md) | — |

---

## Interview Tips

- [Coding Tips for Binary Search](./Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](./Interview%20Tips/Common%20Mistakes.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)
- [Identifying Search Space Problems](./Interview%20Tips/Identifying%20Search%20Space.md)

---

## Navigation

| Previous | Home | Next |
|----------|------|------|
| [Arrays](../1.Arrays/README.md) | [Repository Root](../README.md) | [Strings](../3.Strings/README.md) |

---

> **Last Updated:** 2026-06-26
> **Tags:** `binary-search` `search-space` `rotated-array` `2D-matrix` `peak-element`
