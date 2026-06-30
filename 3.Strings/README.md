# Strings

> **Step 5 of 18** on [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
> 15 problems (Basic + Medium) + 9 Advanced String problems (Step 18)

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

Strings are arrays of characters with one critical difference: **immutability in Java**. Every concatenation creates a new object. This forces O(n²) naive solutions that must be replaced with `StringBuilder`, frequency arrays, or window-based approaches.

**Key Java facts for interviews:**

| Concept | Detail |
|---------|--------|
| `String` | Immutable; `+` in a loop = O(n²) |
| `StringBuilder` | Mutable; `.append()` = O(1) amortized |
| `char[]` | Most space-efficient for in-place manipulation |
| `s.charAt(i)` | O(1) character access |
| `s.toCharArray()` | O(n) — copy once, index freely |
| Char arithmetic | `'a' + k` and `c - 'a'` for freq arrays |

**Core patterns that cover 90% of string problems:**

| Pattern | Problems |
|---------|---------|
| [Sliding Window](./Patterns/Sliding%20Window%20on%20Strings.md) | Longest substring, anagram window, min window |
| [Two Pointers](./Patterns/Two%20Pointers%20on%20Strings.md) | Palindrome check, reverse words |
| [Frequency / Char Count](./Patterns/Frequency%20Count.md) | Anagram, permutation, ransom note |
| [KMP / Z-algorithm](./Patterns/KMP%20and%20Z%20Algorithm.md) | Pattern matching, repeated substring |
| [Hashing / Rabin-Karp](./Patterns/String%20Hashing.md) | Substring search, plagiarism detection |
| [Trie-based](./Patterns/Trie%20Patterns.md) | Prefix matching, autocomplete (see Tries topic) |

---

## Patterns in this Topic

| Pattern | Difficulty | Key Problems |
|---------|-----------|-------------|
| [Sliding Window on Strings](./Patterns/Sliding%20Window%20on%20Strings.md) | Medium–Hard | Min Window, Longest No-Repeat, Anagram |
| [Two Pointers on Strings](./Patterns/Two%20Pointers%20on%20Strings.md) | Easy–Medium | Palindrome, Reverse, Valid Palindrome II |
| [Frequency Count](./Patterns/Frequency%20Count.md) | Easy–Medium | Anagram, Permutation in String |
| [KMP & Z-Algorithm](./Patterns/KMP%20and%20Z%20Algorithm.md) | Hard | Pattern search, Repeated substring |
| [String Hashing (Rabin-Karp)](./Patterns/String%20Hashing.md) | Hard | Substring search O(n), Repeated string |

---

## Problem List by Difficulty

### Basic / Easy

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Remove Outermost Parentheses | Stack / Counter | [LC 1021](https://leetcode.com/problems/remove-outermost-parentheses/) |
| 2 | Reverse Words in a String | Two Pointers | [LC 151](https://leetcode.com/problems/reverse-words-in-a-string/) |
| 3 | Largest Odd Number in String | Greedy | [LC 1903](https://leetcode.com/problems/largest-odd-number-in-string/) |
| 4 | Longest Common Prefix | Trie / Vertical scan | [LC 14](https://leetcode.com/problems/longest-common-prefix/) |
| 5 | Isomorphic Strings | HashMap | [LC 205](https://leetcode.com/problems/isomorphic-strings/) |
| 6 | Check Anagram | [Frequency Count](./Patterns/Frequency%20Count.md) | [LC 242](https://leetcode.com/problems/valid-anagram/) |
| 7 | Sort Characters By Frequency | Frequency + Sort | [LC 451](https://leetcode.com/problems/sort-characters-by-frequency/) |
| 8 | Maximum Nesting Depth of Parentheses | Counter | [LC 1614](https://leetcode.com/problems/maximum-nesting-depth-of-the-parentheses/) |
| 9 | Roman to Integer | HashMap | [LC 13](https://leetcode.com/problems/roman-to-integer/) |
| 10 | Integer to Roman | Greedy | [LC 12](https://leetcode.com/problems/integer-to-roman/) |

### Medium

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Longest Palindromic Substring | Expand Around Center / DP | [LC 5](https://leetcode.com/problems/longest-palindromic-substring/) |
| 2 | Sum of Beauty of all Substrings | Frequency Count | [LC 1781](https://leetcode.com/problems/sum-of-beauty-of-all-substrings/) |
| 3 | Minimum Characters for Palindrome | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | GFG |
| 4 | Count and Say | Simulation | [LC 38](https://leetcode.com/problems/count-and-say/) |
| 5 | Compare Version Numbers | Two Pointers | [LC 165](https://leetcode.com/problems/compare-version-numbers/) |

### Advanced Strings (Step 18 — Pattern Matching)

| # | Problem | Pattern | LeetCode |
|---|---------|---------|----------|
| 1 | Implement ATOI | Parsing | [LC 8](https://leetcode.com/problems/string-to-integer-atoi/) |
| 2 | Count occurrences of anagrams | [Sliding Window](./Patterns/Sliding%20Window%20on%20Strings.md) | GFG |
| 3 | KMP — Pattern Matching | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | [LC 28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) |
| 4 | Minimum Characters to Add (Palindrome) | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | GFG |
| 5 | Shortest Palindrome | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | [LC 214](https://leetcode.com/problems/shortest-palindrome/) |
| 6 | Longest Happy Prefix | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | [LC 1392](https://leetcode.com/problems/longest-happy-prefix/) |
| 7 | String Matching in Array | [KMP](./Patterns/KMP%20and%20Z%20Algorithm.md) | [LC 1408](https://leetcode.com/problems/string-matching-in-an-array/) |
| 8 | Count Distinct Substrings | [String Hashing](./Patterns/String%20Hashing.md) | GFG |
| 9 | Z-Function | [Z-Algorithm](./Patterns/KMP%20and%20Z%20Algorithm.md) | GFG |

---

## Design Problems

| Problem | File |
|---------|------|
| Design Autocomplete System | [Autocomplete System.md](./Design%20Data%20Structure%20Problems/Autocomplete%20System.md) |
| Design Pattern Matcher | [Pattern Matcher.md](./Design%20Data%20Structure%20Problems/Pattern%20Matcher.md) |

---

## Company Coverage

| Company | OA | Interview |
|---------|-----|-----------|
| Amazon | [Amazon OA](./OA-Qns/Amazon.md) | [Amazon Interview](./Interview%20Problems/Amazon.md) |
| Google | [Google OA](./OA-Qns/Google.md) | [Google Interview](./Interview%20Problems/Google.md) |
| Microsoft | [Microsoft OA](./OA-Qns/Microsoft.md) | [Microsoft Interview](./Interview%20Problems/Microsoft.md) |
| Goldman Sachs | [Goldman OA](./OA-Qns/Goldman%20Sachs.md) | — |
| Adobe | [Adobe OA](./OA-Qns/Adobe.md) | — |

---

## Interview Tips

- [Coding Tips for Strings](./Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](./Interview%20Tips/Common%20Mistakes.md)
- [Java String API Reference](./Interview%20Tips/Java%20String%20API.md)
- [Complexity Analysis](./Interview%20Tips/Complexity%20Analysis.md)

---

## Navigation

| Previous | Home | Next |
|----------|------|------|
| [Binary Search](../2.Binary_Search/README.md) | [Repository Root](../README.md) | [Linked List](../4.Linked_List/README.md) |

---

> **Last Updated:** 2026-06-26
> **Tags:** `strings` `sliding-window` `two-pointers` `kmp` `anagram` `palindrome`
