# Linked List

> **Topic:** Step 6 of [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> **Problems:** 31 | **Difficulty split:** 8 Easy · 15 Medium · 8 Hard
> **Last Updated:** 2026-06-26

---

## Table of Contents

1. [Core Patterns](#core-patterns)
2. [Problem List](#problem-list)
3. [Company Coverage](#company-coverage)
4. [Design Problems](#design-problems)
5. [Navigation](#navigation)

---

## Core Patterns

| Pattern | Key Idea | Problems |
|---------|----------|----------|
| [Fast & Slow Pointer](./Patterns/Fast%20and%20Slow%20Pointer.md) | Two pointers at 1x/2x speed; meet inside a cycle | Middle, Cycle, Happy Number |
| [Reverse Linked List](./Patterns/Reverse%20Linked%20List.md) | Three-pointer iterative or stack-based | Reverse LL, K-Group, Palindrome |
| [Merge Linked Lists](./Patterns/Merge%20Linked%20Lists.md) | Dummy head + min-merge + k-way merge | Merge Two, Merge K Sorted |
| [Cycle Detection](./Patterns/Cycle%20Detection.md) | Floyd's + entry-finding trick | Cycle II, Duplicate Number |
| [Two Pointers on LL](./Patterns/Two%20Pointers%20on%20LL.md) | Gap-then-sync, intersection detection | Nth from End, Intersection |

---

## Problem List

### Easy (8)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Middle of the Linked List | [Fast & Slow](./Patterns/Fast%20and%20Slow%20Pointer.md) | [876](https://leetcode.com/problems/middle-of-the-linked-list/) |
| 2 | Reverse Linked List | [Reverse](./Patterns/Reverse%20Linked%20List.md) | [206](https://leetcode.com/problems/reverse-linked-list/) |
| 3 | Palindrome Linked List | [Fast & Slow + Reverse](./Patterns/Reverse%20Linked%20List.md) | [234](https://leetcode.com/problems/palindrome-linked-list/) |
| 4 | Remove Duplicates from Sorted List | Two Pointer | [83](https://leetcode.com/problems/remove-duplicates-from-sorted-list/) |
| 5 | Linked List Cycle | [Cycle Detection](./Patterns/Cycle%20Detection.md) | [141](https://leetcode.com/problems/linked-list-cycle/) |
| 6 | Merge Two Sorted Lists | [Merge](./Patterns/Merge%20Linked%20Lists.md) | [21](https://leetcode.com/problems/merge-two-sorted-lists/) |
| 7 | Intersection of Two Linked Lists | [Two Pointers](./Patterns/Two%20Pointers%20on%20LL.md) | [160](https://leetcode.com/problems/intersection-of-two-linked-lists/) |
| 8 | Delete Node in a Linked List | In-place trick | [237](https://leetcode.com/problems/delete-node-in-a-linked-list/) |

### Medium (15)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Add Two Numbers | [Merge](./Patterns/Merge%20Linked%20Lists.md) | [2](https://leetcode.com/problems/add-two-numbers/) |
| 2 | Remove Nth Node from End | [Two Pointers](./Patterns/Two%20Pointers%20on%20LL.md) | [19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) |
| 3 | Swap Nodes in Pairs | [Reverse](./Patterns/Reverse%20Linked%20List.md) | [24](https://leetcode.com/problems/swap-nodes-in-pairs/) |
| 4 | Odd Even Linked List | Reorder | [328](https://leetcode.com/problems/odd-even-linked-list/) |
| 5 | Sort List | [Merge](./Patterns/Merge%20Linked%20Lists.md) | [148](https://leetcode.com/problems/sort-list/) |
| 6 | Rotate List | [Two Pointers](./Patterns/Two%20Pointers%20on%20LL.md) | [61](https://leetcode.com/problems/rotate-list/) |
| 7 | Linked List Cycle II | [Cycle Detection](./Patterns/Cycle%20Detection.md) | [142](https://leetcode.com/problems/linked-list-cycle-ii/) |
| 8 | Reorder List | [Fast & Slow + Merge](./Patterns/Merge%20Linked%20Lists.md) | [143](https://leetcode.com/problems/reorder-list/) |
| 9 | Copy List with Random Pointer | HashMap | [138](https://leetcode.com/problems/copy-list-with-random-pointer/) |
| 10 | Find the Duplicate Number | [Cycle Detection](./Patterns/Cycle%20Detection.md) | [287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 11 | Flatten a Multilevel Doubly LL | DFS / Stack | [430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |
| 12 | Delete Duplicates (keep none) | Sentinel | [82](https://leetcode.com/problems/remove-duplicates-from-sorted-list-ii/) |
| 13 | Partition List | [Two Pointers](./Patterns/Two%20Pointers%20on%20LL.md) | [86](https://leetcode.com/problems/partition-list/) |
| 14 | Add Two Numbers II (reversed) | Stack | [445](https://leetcode.com/problems/add-two-numbers-ii/) |
| 15 | Swapping Nodes in a LL (by val) | [Two Pointers](./Patterns/Two%20Pointers%20on%20LL.md) | [1721](https://leetcode.com/problems/swapping-nodes-in-a-linked-list/) |

### Hard (8)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Merge K Sorted Lists | [Merge + Heap](./Patterns/Merge%20Linked%20Lists.md) | [23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 2 | Reverse Nodes in k-Group | [Reverse](./Patterns/Reverse%20Linked%20List.md) | [25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 3 | Reverse Alternating K-Group | [Reverse](./Patterns/Reverse%20Linked%20List.md) | — |
| 4 | Flatten Binary Tree to LL | Preorder Relink | [114](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/) |
| 5 | LRU Cache | [Design](./Design%20Data%20Structure%20Problems/LRU%20Cache.md) | [146](https://leetcode.com/problems/lru-cache/) |
| 6 | LFU Cache | [Design](./Design%20Data%20Structure%20Problems/LFU%20Cache.md) | [460](https://leetcode.com/problems/lfu-cache/) |
| 7 | All O`one Data Structure | Design | [432](https://leetcode.com/problems/all-oone-data-structure/) |
| 8 | Design Skiplist | Design | [1206](https://leetcode.com/problems/design-skiplist/) |

---

## Company Coverage

| Company | OA Questions | Interview Questions |
|---------|-------------|---------------------|
| Amazon | [OA-Qns](./OA-Qns/Amazon.md) | [Interview Problems](./Interview%20Problems/Amazon.md) |
| Google | [OA-Qns](./OA-Qns/Google.md) | [Interview Problems](./Interview%20Problems/Google.md) |
| Microsoft | [OA-Qns](./OA-Qns/Microsoft.md) | [Interview Problems](./Interview%20Problems/Microsoft.md) |
| Goldman Sachs | [OA-Qns](./OA-Qns/Goldman%20Sachs.md) | — |
| Adobe | [OA-Qns](./OA-Qns/Adobe.md) | — |
| Flipkart | [OA-Qns](./OA-Qns/Flipkart.md) | — |

---

## Design Problems

| Problem | Complexity | File |
|---------|-----------|------|
| LRU Cache | O(1) get/put via HashMap + DoublyLinkedList | [LRU Cache.md](./Design%20Data%20Structure%20Problems/LRU%20Cache.md) |
| LFU Cache | O(1) get/put via freq-bucketed DLL | [LFU Cache.md](./Design%20Data%20Structure%20Problems/LFU%20Cache.md) |

---

## Navigation

| Section | Files |
|---------|-------|
| [Patterns](./Patterns/) | Fast & Slow · Reverse · Merge · Cycle Detection · Two Pointers on LL |
| [Design](./Design%20Data%20Structure%20Problems/) | LRU Cache · LFU Cache |
| [OA-Qns](./OA-Qns/) | Amazon · Google · Microsoft · Goldman Sachs · Adobe · Flipkart |
| [Interview Problems](./Interview%20Problems/) | Amazon · Google · Microsoft |
| [Interview Tips](./Interview%20Tips/) | Coding Tips · Common Mistakes · Dummy Node Technique · Complexity Analysis |
| [Most Recent Questions](./Most%20Recent%20Questions/) | 2024 · 2025 · 2026 |

---

**Previous:** [Strings](../3.Strings/README.md) | **Next:** [Recursion](../5.Recursion/README.md) | **Home:** [Repository Root](../README.md)

> **Last Updated:** 2026-06-26
