# Recursion

> **Topic:** Step 7 of [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> **Problems:** 25 | **Difficulty split:** 6 Easy · 12 Medium · 7 Hard
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
| [Basic Recursion](./Patterns/Basic%20Recursion.md) | Trust the recursive call; define base case + recurrence | Factorial, Fibonacci, Power, Palindrome |
| [Backtracking](./Patterns/Backtracking.md) | Try → recurse → undo; prune invalid states early | N-Queens, Sudoku, Word Search, Rat in Maze |
| [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | Include/exclude decision tree; start index prevents duplicates | Subsets I/II, Combination Sum I/II |
| [Permutations](./Patterns/Permutations.md) | Swap-based or used[] array; handle duplicates by sorting + skip | Permutations I/II, Next Permutation |
| [Divide & Conquer](./Patterns/Divide%20and%20Conquer.md) | Split problem in half; merge results | Merge Sort, Quick Sort, Pow(x,n) |

---

## Problem List

### Easy (6)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Fibonacci Number | [Basic Recursion](./Patterns/Basic%20Recursion.md) | [509](https://leetcode.com/problems/fibonacci-number/) |
| 2 | Power of Two | [Basic Recursion](./Patterns/Basic%20Recursion.md) | [231](https://leetcode.com/problems/power-of-two/) |
| 3 | Reverse String | [Basic Recursion](./Patterns/Basic%20Recursion.md) | [344](https://leetcode.com/problems/reverse-string/) |
| 4 | Subsets | [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | [78](https://leetcode.com/problems/subsets/) |
| 5 | Letter Combinations of Phone Number | [Backtracking](./Patterns/Backtracking.md) | [17](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) |
| 6 | Flood Fill | [Basic Recursion](./Patterns/Basic%20Recursion.md) | [733](https://leetcode.com/problems/flood-fill/) |

### Medium (12)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | Subsets II | [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | [90](https://leetcode.com/problems/subsets-ii/) |
| 2 | Combination Sum | [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | [39](https://leetcode.com/problems/combination-sum/) |
| 3 | Combination Sum II | [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | [40](https://leetcode.com/problems/combination-sum-ii/) |
| 4 | Combination Sum III | [Subsets & Combinations](./Patterns/Subsets%20and%20Combinations.md) | [216](https://leetcode.com/problems/combination-sum-iii/) |
| 5 | Permutations | [Permutations](./Patterns/Permutations.md) | [46](https://leetcode.com/problems/permutations/) |
| 6 | Permutations II | [Permutations](./Patterns/Permutations.md) | [47](https://leetcode.com/problems/permutations-ii/) |
| 7 | Generate Parentheses | [Backtracking](./Patterns/Backtracking.md) | [22](https://leetcode.com/problems/generate-parentheses/) |
| 8 | Word Search | [Backtracking](./Patterns/Backtracking.md) | [79](https://leetcode.com/problems/word-search/) |
| 9 | Palindrome Partitioning | [Backtracking](./Patterns/Backtracking.md) | [131](https://leetcode.com/problems/palindrome-partitioning/) |
| 10 | Pow(x, n) | [Divide & Conquer](./Patterns/Divide%20and%20Conquer.md) | [50](https://leetcode.com/problems/powx-n/) |
| 11 | Sort an Array (Merge Sort) | [Divide & Conquer](./Patterns/Divide%20and%20Conquer.md) | [912](https://leetcode.com/problems/sort-an-array/) |
| 12 | Decode Ways | [Basic Recursion](./Patterns/Basic%20Recursion.md) | [91](https://leetcode.com/problems/decode-ways/) |

### Hard (7)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|---------|
| 1 | N-Queens | [Backtracking](./Patterns/Backtracking.md) | [51](https://leetcode.com/problems/n-queens/) |
| 2 | N-Queens II | [Backtracking](./Patterns/Backtracking.md) | [52](https://leetcode.com/problems/n-queens-ii/) |
| 3 | Sudoku Solver | [Backtracking](./Patterns/Backtracking.md) | [37](https://leetcode.com/problems/sudoku-solver/) |
| 4 | Word Break II | [Backtracking](./Patterns/Backtracking.md) | [140](https://leetcode.com/problems/word-break-ii/) |
| 5 | Expression Add Operators | [Backtracking](./Patterns/Backtracking.md) | [282](https://leetcode.com/problems/expression-add-operators/) |
| 6 | Regular Expression Matching | [Divide & Conquer](./Patterns/Divide%20and%20Conquer.md) | [10](https://leetcode.com/problems/regular-expression-matching/) |
| 7 | K-th Symbol in Grammar | [Divide & Conquer](./Patterns/Divide%20and%20Conquer.md) | [779](https://leetcode.com/problems/k-th-symbol-in-grammar/) |

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
| Recursive Descent Parser | Grammar rules as mutual recursion | [Recursive Descent Parser.md](./Design%20Data%20Structure%20Problems/Recursive%20Descent%20Parser.md) |
| Expression Evaluator | Operator precedence via recursion depth | [Expression Evaluator.md](./Design%20Data%20Structure%20Problems/Expression%20Evaluator.md) |

---

## Navigation

| Section | Files |
|---------|-------|
| [Patterns](./Patterns/) | Basic Recursion · Backtracking · Subsets & Combinations · Permutations · Divide & Conquer |
| [Design](./Design%20Data%20Structure%20Problems/) | Recursive Descent Parser · Expression Evaluator |
| [OA-Qns](./OA-Qns/) | Amazon · Google · Microsoft · Goldman Sachs · Adobe |
| [Interview Problems](./Interview%20Problems/) | Amazon · Google · Microsoft |
| [Interview Tips](./Interview%20Tips/) | Coding Tips · Common Mistakes · Recursion Tree Visualization · Complexity Analysis |
| [Most Recent Questions](./Most%20Recent%20Questions/) | 2024 · 2025 · 2026 |

---

**Previous:** [Linked List](../4.Linked_List/README.md) | **Next:** [Bit Manipulation](../6.Bit_Manipulation/README.md) | **Home:** [Repository Root](../README.md)

> **Last Updated:** 2026-06-26
