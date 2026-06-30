# Google — Binary Search Interview Questions

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Problems
> **Tags:** `google` `interview` `binary-search`

---

## Table of Contents

1. [What Google Checks](#what-google-checks)
2. [Frequently Asked Questions](#frequently-asked-questions)
3. [Related Files](#related-files)

---

## What Interviewers Usually Check

| Dimension | What They Want |
|-----------|---------------|
| **Mathematical derivation** | Prove why the binary search invariant holds |
| **O(log n) from first principles** | Don't just state — derive |
| **Generalisation** | "Can you apply this to a different problem?" |
| **Correctness over speed** | A correct O(n log n) > buggy O(log n) |
| **Clean code** | Helper methods, no nested ternaries |
| **Handle negative + floating** | BS on continuous answer space |

---

## Frequently Asked Questions

### 1. Median of Two Sorted Arrays ⭐ (Flagship)

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Onsite R2, R3 |
| **Skill tested** | Partition-based binary search, mathematical reasoning |
| **Pattern** | [BS on Answer](../Patterns/Binary%20Search%20on%20Answer.md) |
| **LeetCode** | [LC 4](https://leetcode.com/problems/median-of-two-sorted-arrays/) |

**Key insight:**
> Binary search on the number of elements to take from the shorter array. Maintain the partition invariant: all elements in left partition ≤ all elements in right partition.

**Follow-ups:**
- "Can you do it in O(log(min(m,n)))?" → Yes — always binary search the shorter array
- "Generalise to Kth element of two sorted arrays?" → Same partition logic, stop when left partition has exactly k elements

---

### 2. Find Peak Element

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, R1 |
| **Skill tested** | Invariant-based reasoning |
| **Pattern** | [Peak Finding](../Patterns/Peak%20Finding.md) |
| **LeetCode** | [LC 162](https://leetcode.com/problems/find-peak-element/) |

**Follow-up:**
- "Why does this work for multiple peaks?" → The invariant guarantees a peak exists in the remaining range
- "What about 2D peak?" → LC 1901 — binary search on columns, linear scan for max row

---

### 3. Kth Smallest in Sorted Matrix

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1, R2 |
| **Skill tested** | BS on value range + staircase count |
| **Pattern** | [BS on 2D](../Patterns/Binary%20Search%20on%202D.md) |
| **LeetCode** | [LC 378](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix/) |

**Follow-up:**
- "Min-heap approach?" → O(k log n); worse for large k but easier to code
- "Is the answer always in the matrix?" → Yes — the count approach guarantees it

---

### 4. Find in Mountain Array

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | R2 |
| **Skill tested** | Combine peak finding + two binary searches |
| **Pattern** | [Peak Finding](../Patterns/Peak%20Finding.md) |
| **LeetCode** | [LC 1095](https://leetcode.com/problems/find-in-mountain-array/) |

---

## Related Files

- [Google OA Questions](../OA-Qns/Google.md)
- [Most Recent Questions 2025](../Most%20Recent%20Questions/2025.md)
- [Peak Finding Pattern](../Patterns/Peak%20Finding.md)
- [BS on Answer Pattern](../Patterns/Binary%20Search%20on%20Answer.md)

> **Last Updated:** 2026-06-26
