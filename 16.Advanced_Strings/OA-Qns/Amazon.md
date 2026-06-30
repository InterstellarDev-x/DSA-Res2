> **Topic:** [Advanced Strings](../README.md) · **Section:** OA-Qns

# Amazon — Advanced Strings (Online Assessment)

Amazon OAs lean on the two most "interviewable" string skills: exact substring search (`strStr`) and palindromes (expand-around-center). Anagram sliding windows show up in the medium slot. Library calls are usually allowed, but the better candidates state the underlying algorithm.

| Problem | Frequency | Difficulty | Pattern | LC | Source |
|---------|-----------|------------|---------|----|--------|
| Implement strStr() | ⭐⭐⭐⭐ | Easy | [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) | [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | OA / phone screen |
| Longest Palindromic Substring | ⭐⭐⭐⭐ | Medium | [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md) | [5](https://leetcode.com/problems/longest-palindromic-substring/) | OA |
| Find All Anagrams in a String | ⭐⭐⭐ | Medium | [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) | [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | OA |

## Follow-ups Amazon asks

- **strStr:** "Do it without `indexOf`." → KMP, O(n+m). Then "what's the time complexity of the naive version and why is KMP better?"
- **Longest Palindromic Substring:** "Can you do it in O(n)?" → mention Manacher, but expand-around-center O(n²) is accepted; "return the count instead" pivots to LC 647.
- **Find All Anagrams:** "What if the alphabet is the full Unicode range?" → use a `HashMap` frequency instead of `int[26]`; "stream the text" → keep the rolling frequency window.

## What interviewers look for

- Correct empty-string and `needle.length() > haystack.length()` handling.
- Expand-around-center handling **both** odd and even centers (`2n-1` total centers).
- A clean O(1)-per-step sliding frequency window for anagrams — not re-counting each window.
- Honest complexity: the candidate states O(n+m) / O(n²) without prompting.

## Related

- Deep dives: [Interview Problems → Amazon](../Interview%20Problems/Amazon.md)
- Other companies: [Google](Google.md) · [Microsoft](Microsoft.md) · [Goldman Sachs](Goldman%20Sachs.md) · [Adobe](Adobe.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
