> **Topic:** [Advanced Strings](../README.md) · **Tips 3 of 4**

# Algorithm Selection — Advanced Strings

Choosing the right string engine is half the battle. This is the decision table plus the **signal phrases** that point to each.

---

## Decision Table

| Problem shape | Reach for | Why | Complexity |
|---------------|-----------|-----|-----------|
| Single pattern, exact match | **KMP** (or `indexOf`) | linear, no re-scan of text | O(n+m) |
| **Many** patterns at once | **Aho-Corasick** (mention) | trie of patterns + KMP-style failure links matches all in one pass | O(n + total pattern len + matches) |
| Substring dedup / "any duplicate of length k" / "longest duplicate" | **Rabin-Karp** (rolling hash) | O(1) substring equality after O(n) prep; pairs with binary search on length | O(n) avg per check |
| Longest / count palindromic substrings | **Expand-around-center** | simple, O(1) space, interview default | O(n²) |
| Palindromes in **O(n)** (asked explicitly) | **Manacher** | mirror reuse avoids recomputation | O(n) |
| "Prefix that is also a suffix" / periodicity | **Prefix function** `lps[n-1]` | the border length *is* the answer | O(n) |
| Prefix-match queries / `pattern` inside `text` via concatenation | **Z-algorithm** | `z[i]` over `pattern + '#' + text` flags full matches | O(n+m) |

---

## Signal Phrases → Algorithm

| If the prompt says... | It's pointing at... |
|------------------------|---------------------|
| "first occurrence", "implement strStr", "index of substring" | KMP / `indexOf` |
| "search for these *several* words", "dictionary of patterns" | Aho-Corasick |
| "longest repeated substring", "any substring appearing twice", "fingerprint / plagiarism" | Rabin-Karp + (often) binary search on length |
| "longest palindromic substring", "count palindromic substrings" | Expand-around-center |
| "...in linear time" attached to a palindrome problem | Manacher |
| "happy prefix", "border", "prefix equal to suffix" | prefix function `lps[n-1]` |
| "is the string made of repeated copies", "smallest period" | prefix function, period `n - lps[n-1]` |
| "shortest palindrome by prepending" | KMP on `s + '#' + reverse(s)` |
| "are these two substrings equal?" repeated many times | precomputed prefix hashes (rolling hash) |

---

## Worked decisions

- **"Find the longest substring that appears at least twice."** → "longest" + "duplicate" + monotone in length → **binary search on length + Rabin-Karp** with double hashing. Not KMP (no fixed pattern), not Manacher (not a palindrome).
- **"Can `s` be built from repeated copies of a substring?"** → "repeated copies" → **prefix function**, check `k != 0 && n % (n-k) == 0`. Not hashing — the LPS answers it directly.
- **"Return the longest palindrome in `s`."** → palindrome, no O(n) demand → **expand-around-center**. Only switch to Manacher if they push on time complexity.
- **"Match 10,000 keywords against a log stream."** → many patterns, one pass → **Aho-Corasick**. A loop of `indexOf` would be O(patterns × n).

---

## Rule of thumb hierarchy

1. Can a **library call** (`indexOf`, `contains`) solve it and is it allowed? Use it, state the complexity.
2. Is it about **borders / periodicity / prefix=suffix**? → prefix function.
3. Is it about **comparing many substrings / duplicates**? → rolling hash.
4. Is it about **palindromes**? → expand-around-center (Manacher if O(n) demanded).
5. Is it **multiple patterns**? → Aho-Corasick.

---

## Related

- [Coding Tips](Coding%20Tips.md) · [Common Mistakes](Common%20Mistakes.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Patterns: [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) · [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) · [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
