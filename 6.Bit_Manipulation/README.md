# Bit Manipulation

> **Topic:** Step 8 of [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> **Problems:** 18 | **Difficulty split:** 6 Easy · 8 Medium · 4 Hard
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
| [Bit Basics](./Patterns/Bit%20Basics.md) | AND/OR/XOR/NOT/shifts; set/clear/check/toggle | Hamming Weight, Reverse Bits, Power of 2 |
| [XOR Tricks](./Patterns/XOR%20Tricks.md) | `a ^ a = 0`, `a ^ 0 = a`; cancel duplicates | Single Number I/II/III, Missing Number |
| [Counting Bits](./Patterns/Counting%20Bits.md) | Brian Kernighan `n & (n-1)`, DP recurrence | Counting Bits, Hamming Distance, popcount |
| [Bitmask DP](./Patterns/Bitmask%20DP.md) | Enumerate subsets with integer bitmask | Subsets, TSP, Min XOR Sum |
| [Bit Search / Trie](./Patterns/Bit%20Search%20and%20Trie.md) | Binary Trie on bit prefixes; maximize XOR | Max XOR of Two Numbers, XOR queries |

---

## Problem List

### Easy (6)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Number of 1 Bits (Hamming Weight) | [Counting Bits](./Patterns/Counting%20Bits.md) | [191](https://leetcode.com/problems/number-of-1-bits/) |
| 2 | Reverse Bits | [Bit Basics](./Patterns/Bit%20Basics.md) | [190](https://leetcode.com/problems/reverse-bits/) |
| 3 | Power of Two | [Bit Basics](./Patterns/Bit%20Basics.md) | [231](https://leetcode.com/problems/power-of-two/) |
| 4 | Single Number | [XOR Tricks](./Patterns/XOR%20Tricks.md) | [136](https://leetcode.com/problems/single-number/) |
| 5 | Missing Number | [XOR Tricks](./Patterns/XOR%20Tricks.md) | [268](https://leetcode.com/problems/missing-number/) |
| 6 | Counting Bits | [Counting Bits](./Patterns/Counting%20Bits.md) | [338](https://leetcode.com/problems/counting-bits/) |

### Medium (8)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Single Number II | [XOR Tricks](./Patterns/XOR%20Tricks.md) | [137](https://leetcode.com/problems/single-number-ii/) |
| 2 | Single Number III | [XOR Tricks](./Patterns/XOR%20Tricks.md) | [260](https://leetcode.com/problems/single-number-iii/) |
| 3 | Sum of Two Integers (no +/-) | [Bit Basics](./Patterns/Bit%20Basics.md) | [371](https://leetcode.com/problems/sum-of-two-integers/) |
| 4 | Divide Two Integers | [Bit Basics](./Patterns/Bit%20Basics.md) | [29](https://leetcode.com/problems/divide-two-integers/) |
| 5 | Bitwise AND of Numbers Range | [Bit Basics](./Patterns/Bit%20Basics.md) | [201](https://leetcode.com/problems/bitwise-and-of-numbers-range/) |
| 6 | XOR Queries of a Subarray | [XOR Tricks](./Patterns/XOR%20Tricks.md) | [1310](https://leetcode.com/problems/xor-queries-of-a-subarray/) |
| 7 | Subsets (bitmask approach) | [Bitmask DP](./Patterns/Bitmask%20DP.md) | [78](https://leetcode.com/problems/subsets/) |
| 8 | Hamming Distance | [Counting Bits](./Patterns/Counting%20Bits.md) | [461](https://leetcode.com/problems/hamming-distance/) |

### Hard (4)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Maximum XOR of Two Numbers in an Array | [Bit Search / Trie](./Patterns/Bit%20Search%20and%20Trie.md) | [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |
| 2 | Minimum XOR Sum of Two Arrays | [Bitmask DP](./Patterns/Bitmask%20DP.md) | [1879](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) |
| 3 | Concatenation of Consecutive Binary Numbers | [Bit Basics](./Patterns/Bit%20Basics.md) | [1680](https://leetcode.com/problems/concatenation-of-consecutive-binary-numbers/) |
| 4 | Total Hamming Distance | [Counting Bits](./Patterns/Counting%20Bits.md) | [477](https://leetcode.com/problems/total-hamming-distance/) |

---

## Company Coverage

| Company | OA Questions | Interview Questions |
|---------|-------------|---------------------|
| Amazon | [OA-Qns](./OA-Qns/Amazon.md) | [Interview Problems](./Interview%20Problems/Amazon.md) |
| Google | [OA-Qns](./OA-Qns/Google.md) | [Interview Problems](./Interview%20Problems/Google.md) |
| Microsoft | [OA-Qns](./OA-Qns/Microsoft.md) | [Interview Problems](./Interview%20Problems/Microsoft.md) |
| Goldman Sachs | [OA-Qns](./OA-Qns/Goldman%20Sachs.md) | — |
| Adobe | [OA-Qns](./OA-Qns/Adobe.md) | — |

---

## Design Problems

| Problem | Key Technique | File |
|---------|--------------|------|
| Bitset | Compact boolean array using `long[]`; O(1) per bit op | [Bitset.md](./Design%20Data%20Structure%20Problems/Bitset.md) |
| XOR Linked List | Memory-efficient DLL: store `prev XOR next` in each node | [XOR%20Linked%20List.md](./Design%20Data%20Structure%20Problems/XOR%20Linked%20List.md) |

---

## Bit Manipulation Cheatsheet

| Operation | Expression | Notes |
|-----------|-----------|-------|
| Get bit k | `(n >> k) & 1` | 0 or 1 |
| Set bit k | `n \| (1 << k)` | Force to 1 |
| Clear bit k | `n & ~(1 << k)` | Force to 0 |
| Toggle bit k | `n ^ (1 << k)` | Flip |
| Clear lowest set bit | `n & (n - 1)` | Kernighan trick |
| Isolate lowest set bit | `n & (-n)` | Or `n & ~(n-1)` |
| Check power of 2 | `n > 0 && (n & (n-1)) == 0` | |
| Count set bits | loop `n & (n-1)` | Brian Kernighan |
| XOR cancels pairs | `a ^ a = 0` | |
| Left shift = ×2 | `n << 1` | |
| Arithmetic right shift | `n >> 1` | Sign-extends |
| Logical right shift | `n >>> 1` | Java `>>>` fills with 0 |

---

## Navigation

| Section | Files |
|---------|-------|
| [Patterns](./Patterns/) | Bit Basics · XOR Tricks · Counting Bits · Bitmask DP · Bit Search & Trie |
| [Design](./Design%20Data%20Structure%20Problems/) | Bitset · XOR Linked List |
| [OA-Qns](./OA-Qns/) | Amazon · Google · Microsoft · Goldman Sachs · Adobe |
| [Interview Problems](./Interview%20Problems/) | Amazon · Google · Microsoft |
| [Interview Tips](./Interview%20Tips/) | Coding Tips · Common Mistakes · Java Bit Operators · Complexity Analysis |
| [Most Recent Questions](./Most%20Recent%20Questions/) | 2024 · 2025 · 2026 |

---

**Previous:** [Recursion](../5.Recursion/README.md) | **Next:** [Stacks & Queues](../7.Stacks_and_Queues/README.md) | **Home:** [Repository Root](../README.md)

> **Last Updated:** 2026-06-26
