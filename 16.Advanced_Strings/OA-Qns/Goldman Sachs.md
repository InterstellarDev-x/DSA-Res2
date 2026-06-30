> **Topic:** [Advanced Strings](../README.md) · **Section:** OA-Qns

# Goldman Sachs — Advanced Strings (Online Assessment)

Goldman Sachs OAs (often on HackerRank) keep strings practical: exact matching and counting palindromes. They reward correct, readable code and complexity awareness over algorithmic flash.

| Problem | Frequency | Difficulty | Pattern | LC | Source |
|---------|-----------|------------|---------|----|--------|
| Implement strStr() | ⭐⭐⭐ | Easy | [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) | [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | OA (HackerRank) |
| Palindromic Substrings | ⭐⭐⭐ | Medium | [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md) | [647](https://leetcode.com/problems/palindromic-substrings/) | OA |

## Follow-ups Goldman Sachs asks

- **strStr:** "What is the time complexity of your approach and the naive one?" → O(n+m) for KMP vs O(nm) naive; `indexOf` typically allowed.
- **Palindromic Substrings:** "Count all palindromic substrings." → expand-around-center counting, `2n-1` centers; "now return the longest one" pivots to LC 5.

## What interviewers look for

- Counting palindromes by expanding from each of the `2n-1` centers, incrementing on every successful expansion.
- Off-by-one discipline in the expand loops (`while (l >= 0 && r < n && s[l]==s[r])`).
- Clear stated complexity: O(n²) time, O(1) extra space.

## Related

- OA peers: [Amazon](Amazon.md) · [Google](Google.md) · [Microsoft](Microsoft.md) · [Adobe](Adobe.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
