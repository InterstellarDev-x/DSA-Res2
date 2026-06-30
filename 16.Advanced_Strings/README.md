# Advanced Strings

> ← [Tries](../15.Tries/README.md) | back to [Topic Index](../README.md) (this is the last topic)

**Advanced Strings** is where naive character-by-character scanning gives way to linear-time string algorithms. The whole topic rests on three engines — the **prefix function (KMP)**, the **Z-array**, and **polynomial hashing (Rabin-Karp)** — plus the palindrome family (expand-around-center and Manacher). Once you internalize the prefix function, half of these problems become one-liners on top of it.

This topic covers **9 problems** across **3 patterns**, plus the **Rolling Hash (Rabin-Karp)** design question.

---

## Problem Table

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---------|----|-----------|---------|-----------|
| 1 | Implement strStr() | [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | [String Matching (KMP)](Patterns/String%20Matching%20(KMP).md) | Amazon, Microsoft, Goldman Sachs |
| 2 | Repeated String Match | [686](https://leetcode.com/problems/repeated-string-match/) | Medium | [String Matching (KMP)](Patterns/String%20Matching%20(KMP).md) | Amazon, Google |
| 3 | Longest Happy Prefix | [1392](https://leetcode.com/problems/longest-happy-prefix/) | Hard | [String Matching (KMP)](Patterns/String%20Matching%20(KMP).md) | Google |
| 4 | Repeated Substring Pattern | [459](https://leetcode.com/problems/repeated-substring-pattern/) | Easy | [String Matching (KMP)](Patterns/String%20Matching%20(KMP).md) | Microsoft, Amazon |
| 5 | Find All Anagrams in a String | [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | [Z Algorithm and Hashing](Patterns/Z%20Algorithm%20and%20Hashing.md) | Amazon, Adobe, Google |
| 6 | Longest Duplicate Substring | [1044](https://leetcode.com/problems/longest-duplicate-substring/) | Hard | [Z Algorithm and Hashing](Patterns/Z%20Algorithm%20and%20Hashing.md) | Google |
| 7 | Distinct Echo Substrings | [1316](https://leetcode.com/problems/distinct-echo-substrings/) | Hard | [Z Algorithm and Hashing](Patterns/Z%20Algorithm%20and%20Hashing.md) | Google |
| 8 | Shortest Palindrome | [214](https://leetcode.com/problems/shortest-palindrome/) | Hard | [Palindrome Algorithms](Patterns/Palindrome%20Algorithms.md) | Google |
| 9 | Longest Palindromic Substring | [5](https://leetcode.com/problems/longest-palindromic-substring/) | Medium | [Palindrome Algorithms](Patterns/Palindrome%20Algorithms.md) | Amazon, Microsoft, Adobe |
| 10 | Palindromic Substrings | [647](https://leetcode.com/problems/palindromic-substrings/) | Medium | [Palindrome Algorithms](Patterns/Palindrome%20Algorithms.md) | Goldman Sachs |

> The topic is sized at **9 core problems**; the table lists 10 LeetCode rows because Longest Palindromic Substring and Palindromic Substrings are taught as a pair under the same palindrome pattern.

---

## Patterns Summary

| Pattern | Core Idea | Problems |
|---------|-----------|----------|
| [String Matching (KMP)](Patterns/String%20Matching%20(KMP).md) | The **prefix function** `lps[i]` (longest proper prefix that is also a suffix) powers linear matching, periodicity detection, and "prefix == suffix" queries. | strStr (28), Repeated String Match (686), Longest Happy Prefix (1392), Repeated Substring Pattern (459) |
| [Z Algorithm and Hashing](Patterns/Z%20Algorithm%20and%20Hashing.md) | The **Z-array** `z[i]` (longest substring from `i` matching a prefix) and **polynomial hashing** let you compare substrings in O(1) after O(n) prep. | Find All Anagrams (438), Longest Duplicate Substring (1044), Distinct Echo Substrings (1316) |
| [Palindrome Algorithms](Patterns/Palindrome%20Algorithms.md) | **Expand-around-center** (O(n²), interview default) and **Manacher** (O(n)) plus the KMP-on-`s+#+reverse(s)` trick for shortest palindrome. | Shortest Palindrome (214), Longest Palindromic Substring (5), Palindromic Substrings (647) |

---

## Design Data Structure Problems

| Problem | File | What you implement |
|---------|------|---------------------|
| Rolling Hash (Rabin-Karp) | [Rolling Hash (Rabin-Karp).md](Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md) | Polynomial hash, rolling update, modular arithmetic, spurious-hit verification, double hashing |

Rolling hashing is the substring-comparison primitive behind Rabin-Karp matching, Longest Duplicate Substring (binary search + hash), Distinct Echo Substrings, and plagiarism/dedup framings.

---

## Company Coverage

| Company | OA-Qns | Interview Problems | Headline problems |
|---------|--------|--------------------|-------------------|
| Amazon | [link](OA-Qns/Amazon.md) | [link](Interview%20Problems/Amazon.md) | strStr, Longest Palindromic Substring, Find All Anagrams |
| Google | [link](OA-Qns/Google.md) | [link](Interview%20Problems/Google.md) | Shortest Palindrome, Longest Duplicate Substring, Longest Happy Prefix |
| Microsoft | [link](OA-Qns/Microsoft.md) | [link](Interview%20Problems/Microsoft.md) | strStr, Repeated Substring Pattern, Longest Palindromic Substring |
| Goldman Sachs | [link](OA-Qns/Goldman%20Sachs.md) | — | strStr, Palindromic Substrings |
| Adobe | [link](OA-Qns/Adobe.md) | — | Longest Palindromic Substring, Find All Anagrams |

---

## The String Algorithm Landscape

The whole point of this topic is knowing **which engine to reach for**. The table below is the mental model:

| Algorithm | Time | Space | What it solves | When to reach for it |
|-----------|------|-------|----------------|----------------------|
| Naive scan | O(nm) | O(1) | substring search | tiny inputs; "just get it correct first" |
| **KMP** (prefix function) | O(n+m) | O(m) | single-pattern search, periodicity, prefix=suffix | library calls forbidden; need worst-case linear; periodicity questions |
| **Z-algorithm** | O(n+m) | O(n+m) | single-pattern search, prefix-match queries | when Z-coordinates are more natural than LPS (concatenate `pattern + sep + text`) |
| **Rabin-Karp** (rolling hash) | O(n+m) avg, O(nm) worst | O(1) | substring search, multi-length dedup, equality checks | comparing **many** substrings; binary-search-on-length problems; "any duplicate of length k" |
| **Manacher** | O(n) | O(n) | all palindromic substrings / longest palindrome | only when O(n) palindrome is explicitly required |
| **Expand-around-center** | O(n²) | O(1) | longest / count of palindromic substrings | the interview default — simple, correct, usually fast enough |

**Decision in one breath:**
- Single pattern, exact match → **KMP** (or `indexOf` if allowed).
- "Is the string periodic?" / "prefix that is also a suffix" → **prefix function** `lps[n-1]`.
- "Longest/any duplicate substring", "compare substrings fast" → **Rabin-Karp** (verify hits; double hash).
- "Longest palindrome", "count palindromes" → **expand-around-center**, escalate to **Manacher** only if asked for O(n).

KMP and Z carry the same information in different coordinates: `lps` measures borders ending at `i`, `z` measures matches starting at `i`. Either solves single-pattern matching in O(n+m). For **multiple** patterns at once, mention **Aho-Corasick** (a trie of patterns plus KMP-style failure links).

---

## Cross-links

- **Sliding Window** — Find All Anagrams is a fixed-size frequency window; see [Fixed Size Window](../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md).
- **DP on Strings** — the palindrome problems have an O(n²) DP alternative (the classic `dp[i][j]` interval table); see [DP on Strings](../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md).
- **Tries** — for multi-pattern matching, the previous topic's trie underlies Aho-Corasick; see [Tries](../15.Tries/README.md).

---

## Directory Map

- **[Patterns/](Patterns/)** — the three core patterns with full Java and dry-runs.
- **[Design Data Structure Problems/](Design%20Data%20Structure%20Problems/)** — Rolling Hash (Rabin-Karp).
- **[OA-Qns/](OA-Qns/)** — online-assessment frequency by company.
- **[Interview Problems/](Interview%20Problems/)** — deep dives with follow-ups by company.
- **[Interview Tips/](Interview%20Tips/)** — coding tips, common mistakes, algorithm selection, complexity.
- **[Most Recent Questions/](Most%20Recent%20Questions/)** — 2024–2026 trend tracking.

---

> **Last Updated:** 2026-06-26
