> **Topic:** [Advanced Strings](../README.md) · **Section:** OA-Qns

# Google — Advanced Strings (Online Assessment)

Google is the one company that genuinely tests the hard string algorithms in an OA: KMP-derived palindrome tricks, prefix-function periodicity, and Rabin-Karp + binary search. The differentiators here are Shortest Palindrome and Longest Duplicate Substring.

| Problem | Frequency | Difficulty | Pattern | LC | Source |
|---------|-----------|------------|---------|----|--------|
| Shortest Palindrome | ⭐⭐⭐⭐ | Hard | [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md) | [214](https://leetcode.com/problems/shortest-palindrome/) | OA / onsite |
| Longest Duplicate Substring | ⭐⭐⭐⭐ | Hard | [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) | [1044](https://leetcode.com/problems/longest-duplicate-substring/) | OA / onsite |
| Longest Happy Prefix | ⭐⭐⭐ | Hard | [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) | [1392](https://leetcode.com/problems/longest-happy-prefix/) | OA |

## Follow-ups Google asks

- **Shortest Palindrome:** "Derive why `lps` of `s + '#' + reverse(s)` gives the longest palindromic prefix." Then "why is the `'#'` separator necessary?" → prevents the prefix and reversed-suffix from overlapping/colliding.
- **Longest Duplicate Substring:** "Prove the answer length is monotone." → binary search on length. "Your single hash fails this test case — fix it." → double hashing, verify on hit.
- **Longest Happy Prefix:** "What does `lps[n-1]` mean and why is it the answer?" → it is the longest border of the whole string by definition.

## What interviewers look for

- Recognizing that palindrome and periodicity problems collapse onto the **prefix function**.
- Modular arithmetic done right: `long`, prime mod `1_000_000_007L`, `(x%MOD+MOD)%MOD`.
- Awareness of **collisions** — the candidate verifies hash matches or uses two moduli.
- A correct binary-search-on-answer with a clean O(n) feasibility check.

## Related

- Deep dives: [Interview Problems → Google](../Interview%20Problems/Google.md)
- Other companies: [Amazon](Amazon.md) · [Microsoft](Microsoft.md) · [Goldman Sachs](Goldman%20Sachs.md) · [Adobe](Adobe.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
