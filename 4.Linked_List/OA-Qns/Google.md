# Google OA — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** OA-Qns
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Difficulty | Frequency | Year | Pattern | LeetCode |
|---|---------|-----------|----------|------|---------|---------|
| 1 | Reverse Nodes in K-Group | Hard | ⭐⭐⭐⭐⭐ | 2023–2026 | [Reverse](../Patterns/Reverse%20Linked%20List.md) | [25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 2 | Merge K Sorted Lists | Hard | ⭐⭐⭐⭐⭐ | 2023–2026 | [Merge + Heap](../Patterns/Merge%20Linked%20Lists.md) | [23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 3 | Linked List Cycle II | Medium | ⭐⭐⭐⭐ | 2024–2026 | [Cycle Detection](../Patterns/Cycle%20Detection.md) | [142](https://leetcode.com/problems/linked-list-cycle-ii/) |
| 4 | Find the Duplicate Number | Medium | ⭐⭐⭐⭐ | 2023–2026 | [Cycle Detection](../Patterns/Cycle%20Detection.md) | [287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 5 | LFU Cache | Hard | ⭐⭐⭐ | 2024–2025 | [Design](../Design%20Data%20Structure%20Problems/LFU%20Cache.md) | [460](https://leetcode.com/problems/lfu-cache/) |
| 6 | Copy List with Random Pointer (O(1) space) | Medium | ⭐⭐⭐ | 2024–2025 | Interleave trick | [138](https://leetcode.com/problems/copy-list-with-random-pointer/) |
| 7 | Swap Nodes in Pairs | Medium | ⭐⭐⭐ | 2023–2025 | [Reverse](../Patterns/Reverse%20Linked%20List.md) | [24](https://leetcode.com/problems/swap-nodes-in-pairs/) |
| 8 | Flatten Multilevel Doubly LL | Medium | ⭐⭐ | 2024 | DFS | [430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |

---

## Google Interview Style Notes

Google focuses heavily on **correctness and edge cases**:

- Reverse K-Group: What happens if the last group has < k nodes? (leave in place)
- Merge K Sorted: Empty lists in the input array? (filter nulls before adding to heap)
- Find Duplicate: Can you solve it **without modifying** the array and in O(1) space? (Floyd's required)
- LFU: What's the tie-breaking rule? (LRU within same frequency)

---

## Google L3–L5 Problem Mapping

| Level | Expected Problem | Follow-up |
|-------|-----------------|-----------|
| L3 | Merge Two Sorted, Reverse LL | Explain space complexity |
| L4 | Reverse K-Group, Merge K Sorted | Handle edge cases, optimize space |
| L5 | LFU Cache, Find Duplicate (O(1) space) | Design alternatives, thread safety |

---

## Related Files

- [Google Interview Problems](../Interview%20Problems/Google.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
