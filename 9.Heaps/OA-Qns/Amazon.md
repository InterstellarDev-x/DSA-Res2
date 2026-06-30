# Amazon — Heaps OA Questions

> **Topic:** [Heaps](../README.md) · **Company:** Amazon

---

## OA Question Bank

| # | Question | Frequency | Difficulty | Pattern | Link |
|---|---------|-----------|-----------|---------|------|
| 1 | [Top K Frequent Elements](https://leetcode.com/problems/top-k-frequent-elements/) | ⭐⭐⭐⭐⭐ | Medium | [Top K](../Patterns/Top%20K%20Elements.md) | LC 347 |
| 2 | [Task Scheduler](https://leetcode.com/problems/task-scheduler/) | ⭐⭐⭐⭐ | Medium | [Heap + Freq](../Patterns/Heap%20and%20Frequency.md) | LC 621 |
| 3 | [Find Median from Data Stream](https://leetcode.com/problems/find-median-from-data-stream/) | ⭐⭐⭐⭐ | Hard | [Two Heaps](../Patterns/Two%20Heaps.md) | LC 295 |
| 4 | [Kth Largest Element in Array](https://leetcode.com/problems/kth-largest-element-in-an-array/) | ⭐⭐⭐⭐ | Medium | [Top K](../Patterns/Top%20K%20Elements.md) | LC 215 |
| 5 | [K Closest Points to Origin](https://leetcode.com/problems/k-closest-points-to-origin/) | ⭐⭐⭐ | Medium | [Top K](../Patterns/Top%20K%20Elements.md) | LC 973 |
| 6 | [Merge K Sorted Lists](https://leetcode.com/problems/merge-k-sorted-lists/) | ⭐⭐⭐ | Hard | [K-Way Merge](../Patterns/K%20Way%20Merge.md) | LC 23 |
| 7 | [Top K Frequent Words](https://leetcode.com/problems/top-k-frequent-words/) | ⭐⭐⭐ | Medium | [Top K](../Patterns/Top%20K%20Elements.md) | LC 692 |
| 8 | [Kth Largest in Stream](https://leetcode.com/problems/kth-largest-element-in-a-stream/) | ⭐⭐⭐ | Easy | [Top K](../Patterns/Top%20K%20Elements.md) | LC 703 |
| 9 | [Last Stone Weight](https://leetcode.com/problems/last-stone-weight/) | ⭐⭐ | Easy | [Heap Basics](../Patterns/Heap%20Basics.md) | LC 1046 |
| 10 | [Maximum Frequency Stack](https://leetcode.com/problems/maximum-frequency-stack/) | ⭐⭐ | Hard | Cross-topic | LC 895 |

---

## Top 4 Must-Know for Amazon

### 1. Top K Frequent Elements ⭐⭐⭐⭐⭐
Amazon's most-asked heap problem. Appears in SDE OA and phone screens regularly.
- Know both min-heap O(n log k) and bucket sort O(n) approaches
- Follow-up: Top K Frequent Words with lexicographic tie-breaking

### 2. Task Scheduler ⭐⭐⭐⭐
Amazon frames this as "CPU scheduling" or "order fulfillment with cooldowns."
- Know the formula approach O(1) AND the heap simulation approach
- Formula: `max((maxFreq-1)*(n+1)+maxCount, tasks.length)`

### 3. Find Median from Data Stream ⭐⭐⭐⭐
Amazon SDE-II design question.
- Two heaps with size-balance invariant
- Follow-up: overflow-safe median calculation

### 4. Kth Largest Element ⭐⭐⭐⭐
Amazon asks both heap (O(n log k)) and quickselect (O(n) average) approaches.
- Be ready to justify when to use each

---

## Related Files

- [Amazon Interview Problems](../Interview%20Problems/Amazon.md)
- [Google OA](./Google.md)

> **Last Updated:** 2026-06-26
