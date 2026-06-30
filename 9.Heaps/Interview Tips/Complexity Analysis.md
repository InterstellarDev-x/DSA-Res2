# Complexity Analysis — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Pattern-to-Complexity Quick Reference

| Problem / Operation | Time | Space | Notes |
|---------------------|------|-------|-------|
| Heap offer / poll / peek | O(log n) / O(log n) / O(1) | O(n) | Standard heap ops |
| Build heap from n elements | O(n) | O(n) | Bottom-up heapify |
| `PriorityQueue.remove(x)` | O(n) | — | Linear search; avoid |
| [Last Stone Weight](../Patterns/Heap%20Basics.md) | O(n log n) | O(n) | n polls, each O(log n) |
| [Kth Largest (heap)](../Patterns/Top%20K%20Elements.md) | O(n log k) | O(k) | k-size min-heap |
| [Kth Largest (quickselect)](../Patterns/Top%20K%20Elements.md) | O(n) avg / O(n²) worst | O(1) | Randomized pivot avoids worst |
| [K Closest Points (heap)](../Patterns/Top%20K%20Elements.md) | O(n log k) | O(k) | k-size max-heap |
| [K Closest Points (sort)](../Patterns/Top%20K%20Elements.md) | O(n log n) | O(1) | Simpler code |
| [Top K Frequent (heap)](../Patterns/Top%20K%20Elements.md) | O(n log k) | O(n) | n = distinct elements in heap |
| [Top K Frequent (bucket)](../Patterns/Top%20K%20Elements.md) | O(n) | O(n) | Frequency buckets |
| [Top K Frequent Words](../Patterns/Top%20K%20Elements.md) | O(n log k) | O(n) | String comparison in comparator |
| [Kth Largest in Stream](../Patterns/Top%20K%20Elements.md) | O(log k) per add | O(k) | k-size min-heap maintained |
| [Merge K Sorted Lists](../Patterns/K%20Way%20Merge.md) | O(n log k) | O(k) | n = total nodes |
| [Sort K-Sorted Array](../Patterns/K%20Way%20Merge.md) | O(n log k) | O(k) | Window of k+1 |
| [K Pairs Smallest Sums](../Patterns/K%20Way%20Merge.md) | O(k log k) | O(k) | Only k polls needed |
| [Kth Smallest in Matrix (heap)](../Patterns/K%20Way%20Merge.md) | O(k log k) | O(k) | k polls from n-row heap |
| [Kth Smallest in Matrix (binary search)](../Patterns/K%20Way%20Merge.md) | O(n log(max-min)) | O(1) | n = matrix side length |
| [Smallest Range from K Lists](../Patterns/K%20Way%20Merge.md) | O(n log k) | O(k) | n = total elements |
| [Find Median Stream](../Patterns/Two%20Heaps.md) | O(log n) add, O(1) median | O(n) | Two heaps |
| [Sliding Window Median](../Patterns/Two%20Heaps.md) | O(n log k) | O(k) | TreeMap for O(log k) removal |
| [IPO](../Patterns/Two%20Heaps.md) | O(n log n + k log n) | O(n) | Sort + k heap operations |
| [Reorganize String (heap)](../Patterns/Heap%20and%20Frequency.md) | O(n log 26) = O(n) | O(26) | Fixed alphabet |
| [Reorganize String (interleave)](../Patterns/Heap%20and%20Frequency.md) | O(n) | O(n) | Place most frequent first |
| [Task Scheduler (formula)](../Patterns/Heap%20and%20Frequency.md) | O(26 log 26) = O(1) | O(1) | Sort 26 frequencies |
| [Task Scheduler (heap sim)](../Patterns/Heap%20and%20Frequency.md) | O(n log 26) = O(n) | O(26) | Heap + cooldown queue |
| [Design Twitter getNewsFeed](../Design%20Data%20Structure%20Problems/Twitter%20Feed.md) | O(F log F + 10 log F) | O(F) | F = followees count |

---

## When to Choose Heap vs Sort vs Quickselect

| Constraint | Best Approach |
|-----------|--------------|
| Streaming / online data | Heap (sort/quickselect need all data) |
| Offline, k << n | Heap O(n log k) beats sort O(n log n) |
| Offline, k ≈ n | Sort may be simpler |
| k = 1 (just the minimum/maximum) | Linear scan O(n) — no heap needed |
| O(1) space required | Quickselect (modifies array) |
| Guaranteed O(n log k) worst case | Heap (quickselect has O(n²) worst) |
| n is known and bounded | Bucket sort O(n) |

---

## O(n log k) vs O(n log n) — When Does It Matter?

| k | log k | Savings vs log n (n=10⁶) |
|---|-------|--------------------------|
| k = 10 | ≈ 3.3 | 6× faster |
| k = 100 | ≈ 6.6 | 3× faster |
| k = 1000 | ≈ 10 | 2× faster |
| k = n/2 | ≈ log n - 1 | Negligible |

Heap matters most when k is small relative to n.

---

## Two Heaps — Space Analysis

Two heaps together hold all n elements:
- `lo.size()` = ⌈n/2⌉
- `hi.size()` = ⌊n/2⌋
- Total space: O(n)

For Sliding Window Median with TreeMap: O(k) space (only window elements in both TreeMaps).

---

## How to Explain Heap Complexity in Interviews

**For Top K:**
> "I maintain a min-heap of size k. Each of the n elements is offered to the heap — O(log k). When size exceeds k, I poll — O(log k). Total: O(n log k). The heap never grows beyond k, so space is O(k)."

**For K-Way Merge:**
> "The heap has at most k entries (one per list). Each of the n total elements is pushed once and popped once — O(log k) per operation. Total: O(n log k), space O(k)."

**For Two Heaps Median:**
> "Each number is offered once and possibly moved once during rebalancing. Each operation is O(log n). For n numbers, total addNum cost is O(n log n). findMedian is O(1) — just peek at the two heap tops."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Heap Patterns](./Heap%20Patterns.md)

> **Last Updated:** 2026-06-26
