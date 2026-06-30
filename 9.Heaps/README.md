# Heaps

> **Topic 11 of 18** — 17 problems (2E / 10M / 5H)
> Java-centric, pattern-driven reference for interview preparation.
> [← Sliding Window](../8.Sliding_Window/README.md) · [→ Greedy](../10.Greedy/README.md)

---

## Quick Navigation

| Section | Files |
|---------|-------|
| [Patterns](#patterns) | Heap Basics · Top K Elements · K-Way Merge · Two Heaps · Heap + Frequency |
| [Design Problems](#design-data-structure-problems) | MedianFinder · Twitter Feed (Top K per Group) |
| [OA Questions](#oa-questions) | Amazon · Google · Microsoft · Goldman Sachs · Adobe |
| [Interview Problems](#interview-problems) | Amazon · Google · Microsoft |
| [Interview Tips](#interview-tips) | Coding Tips · Common Mistakes · Heap Patterns · Complexity Analysis |
| [Recent Questions](#most-recent-questions) | 2024 · 2025 · 2026 |

---

## Problem List

| # | Problem | Difficulty | Pattern | Companies |
|---|---------|-----------|---------|-----------|
| 1 | [Kth Largest Element in a Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | Easy | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon |
| 2 | [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | Easy | [Heap Basics](./Patterns/Heap%20Basics.md) | Amazon, Adobe |
| 3 | [Kth Largest Element in Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Medium | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon, Google, Microsoft |
| 4 | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | Medium | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon, Google |
| 5 | [Sort an Almost Sorted Array](https://leetcode.com/problems/sort-an-almost-sorted-array/) | Medium | [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Amazon, Microsoft |
| 6 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | Medium | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon, Google, Microsoft |
| 7 | [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/) | Medium | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon, Google |
| 8 | [Find K Pairs with Smallest Sums](https://leetcode.com/problems/find-k-pairs-with-smallest-sums/) | Medium | [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Google, Amazon |
| 9 | [Kth Smallest Element in Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) | Medium | [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Google, Amazon |
| 10 | [Reorganize String](https://leetcode.com/problems/reorganize-string/) | Medium | [Heap + Freq](./Patterns/Heap%20and%20Frequency.md) | Google, Amazon |
| 11 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | Medium | [Heap + Freq](./Patterns/Heap%20and%20Frequency.md) | Amazon, Google, Microsoft |
| 12 | [Find the Kth Largest Integer in the Array](https://leetcode.com/problems/find-the-kth-largest-integer-in-the-array/) | Medium | [Top K](./Patterns/Top%20K%20Elements.md) | Amazon |
| 13 | [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | Hard | [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Amazon, Google, Microsoft |
| 14 | [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | Hard | [Two Heaps](./Patterns/Two%20Heaps.md) | Amazon, Google |
| 15 | [Sliding Window Median](https://leetcode.com/problems/sliding-window-median/) | Hard | [Two Heaps](./Patterns/Two%20Heaps.md) | Google |
| 16 | [IPO](https://leetcode.com/problems/ipo/) | Hard | [Two Heaps](./Patterns/Two%20Heaps.md) | Google, Amazon |
| 17 | [Smallest Range Covering Elements from K Lists](https://leetcode.com/problems/smallest-range-covering-elements-from-k-lists/) | Hard | [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Google |

---

## Patterns

| Pattern | Key Problems | Core Idea |
|---------|-------------|-----------|
| [Heap Basics](./Patterns/Heap%20Basics.md) | Last Stone Weight | PriorityQueue API; min/max heap; custom comparators |
| [Top K Elements](./Patterns/Top%20K%20Elements.md) | Kth Largest, K Closest, Top K Frequent | Min-heap of size k; pop when size > k |
| [K-Way Merge](./Patterns/K%20Way%20Merge.md) | Merge K Sorted, K Pairs, Kth Smallest Matrix | Min-heap of (value, list-index, element-index) |
| [Two Heaps](./Patterns/Two%20Heaps.md) | Median Stream, Sliding Window Median, IPO | Max-heap for lower half + min-heap for upper half |
| [Heap + Frequency](./Patterns/Heap%20and%20Frequency.md) | Reorganize String, Task Scheduler | HashMap freq count + max-heap by frequency |

---

## Design Data Structure Problems

| Design | Key Operations | Complexity |
|--------|---------------|------------|
| [MedianFinder](./Design%20Data%20Structure%20Problems/Median%20Finder.md) | `addNum` O(log n) · `findMedian` O(1) | Two heaps |
| [Twitter Feed](./Design%20Data%20Structure%20Problems/Twitter%20Feed.md) | `postTweet`, `getNewsFeed` | K-way merge via min-heap |

---

## Company Coverage

| Company | Top OA Problems | Top Interview Problems |
|---------|----------------|------------------------|
| Amazon | Top K Frequent ⭐⭐⭐⭐⭐, Task Scheduler ⭐⭐⭐⭐ | Find Median from Stream, Merge K Lists |
| Google | Find Median ⭐⭐⭐⭐⭐, Smallest Range ⭐⭐⭐⭐ | Sliding Window Median, IPO, K Pairs |
| Microsoft | Kth Largest ⭐⭐⭐⭐, Merge K Lists ⭐⭐⭐⭐ | Task Scheduler, Top K Frequent |
| Goldman Sachs | Top K Frequent Words ⭐⭐⭐⭐ | K Closest Points, Sort K-sorted |
| Adobe | Last Stone Weight ⭐⭐⭐, Reorganize String ⭐⭐⭐ | Kth Largest |

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
- [Heap Patterns Guide](./Interview%20Tips/Heap%20Patterns.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)

---

## Most Recent Questions

- [2024](./Most%20Recent%20Questions/2024.md)
- [2025](./Most%20Recent%20Questions/2025.md)
- [2026](./Most%20Recent%20Questions/2026.md)

---

> **Last Updated:** 2026-06-26
> **Problems:** 17 (2E / 10M / 5H)
> [← Back to Root](../README.md)
