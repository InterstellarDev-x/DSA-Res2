> **Topic:** [Advanced Strings](../README.md) · **Tips 4 of 4**

# Complexity Analysis — Advanced Strings

Time and space for every problem and every algorithm in this topic, with the naive baseline for comparison.

---

## Per-Problem Complexity

| # | Problem | LC | Algorithm | Time | Space |
|---|---------|----|-----------|------|-------|
| 1 | Implement strStr() | 28 | KMP | O(n + m) | O(m) |
| 2 | Repeated String Match | 686 | build + KMP | O(\|a\| + \|b\|) | O(\|a\| + \|b\|) |
| 3 | Longest Happy Prefix | 1392 | prefix function | O(n) | O(n) |
| 4 | Repeated Substring Pattern | 459 | prefix function (period) | O(n) | O(n) |
| 5 | Find All Anagrams | 438 | sliding frequency window | O(n) | O(1) (26) |
| 6 | Longest Duplicate Substring | 1044 | binary search + Rabin-Karp | O(n log n) avg | O(n) |
| 7 | Distinct Echo Substrings | 1316 | double hashing + set | O(n²) | O(n²) |
| 8 | Shortest Palindrome | 214 | KMP on `s#reverse(s)` | O(n) | O(n) |
| 9 | Longest Palindromic Substring | 5 | expand-around-center | O(n²) | O(1) |
| 10 | Palindromic Substrings | 647 | expand-around-center (count) | O(n²) | O(1) |

`n`, `m` are text and pattern lengths respectively. Problems 9 and 10 drop to **O(n)** with Manacher when required.

---

## Algorithm Reference

| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| **Naive substring search** | O(nm) | O(1) | re-scans pattern at every start position |
| **KMP** (prefix function) | O(n + m) | O(m) | text pointer never moves backward |
| **Z-algorithm** | O(n + m) | O(n + m) | equivalent power to KMP, different coordinates |
| **Rabin-Karp** | O(n + m) **avg**, O(nm) **worst** | O(1) rolling / O(n) prefix | worst case = all hashes collide; double hashing makes it effectively average |
| **Manacher** | O(n) | O(n) | linear palindrome, transformed string + mirror |
| **Expand-around-center** | O(n²) | O(1) | the interview default for palindromes |
| **Aho-Corasick** | O(n + Σm + matches) | O(Σm · Σ) | multi-pattern; trie + failure links |

---

## Naive vs Linear — the headline comparison

| Task | Naive | Better | Speedup intuition |
|------|-------|--------|-------------------|
| Substring search | O(nm) | KMP / Z O(n+m) | reuse matched border instead of restarting |
| Longest palindrome | O(n³) (check every substring) | expand O(n²), Manacher O(n) | grow from centers; reuse mirror radii |
| Longest duplicate substring | O(n²·n) compare-all | O(n log n) binary search + hash | monotone length + O(1) substring equality |
| Multi-pattern search | O(k·n) (k separate scans) | Aho-Corasick O(n + Σm) | one pass over text for all patterns |

---

## Why Rabin-Karp is O(n+m) average but O(nm) worst

Each window hash is O(1) to roll, so hashing all windows is O(n). Verification runs only on a hash match. With a good prime modulus, false matches are ~`1/MOD ≈ 10^-9`, so total verification cost is negligible → **O(n+m) average**. In the adversarial worst case where *every* window collides, each match triggers an O(m) verification → **O(nm)**. **Double hashing** drops collision probability to ~`10^-18`, which is why it restores the average case on hostile inputs.

---

## Space notes

- KMP needs O(m) for `lps`; the search itself is O(1) extra.
- Expand-around-center is the only **O(1) space** palindrome method — a tie-breaker when memory is constrained.
- Distinct Echo Substrings (1316) stores hashes of candidate substrings; the set can hold O(n²) entries, dominating space.

---

## Related

- [Coding Tips](Coding%20Tips.md) · [Common Mistakes](Common%20Mistakes.md) · [Algorithm Selection](Algorithm%20Selection.md)
- [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md) — average vs worst derivation
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
