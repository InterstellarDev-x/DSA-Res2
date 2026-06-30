> **Topic:** [Advanced Strings](../README.md) · **Section:** OA-Qns

# Adobe — Advanced Strings (Online Assessment)

Adobe OAs concentrate on the two most common string mediums: palindromic substrings and anagram windows. Expect clear problem statements and a focus on correct, idiomatic Java.

| Problem | Frequency | Difficulty | Pattern | LC | Source |
|---------|-----------|------------|---------|----|--------|
| Longest Palindromic Substring | ⭐⭐⭐ | Medium | [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md) | [5](https://leetcode.com/problems/longest-palindromic-substring/) | OA |
| Find All Anagrams in a String | ⭐⭐⭐ | Medium | [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) | [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | OA |

## Follow-ups Adobe asks

- **Longest Palindromic Substring:** "Return the substring, not just the length." Track best `(start, len)`; "what about an empty string?" → guard `s.length() < 1`.
- **Find All Anagrams:** "Return start indices of all permutations of `p` in `s`." → fixed-size frequency window; "how would you handle uppercase + lowercase?" → widen the frequency array or use a `HashMap`.

## What interviewers look for

- A single fixed-size sliding window with a frequency array, updated in O(1) per shift (not recomputed each window).
- Both odd and even palindrome centers handled.
- Correct comparison of the window frequency to the pattern frequency (a running `match` counter beats array equality each step).

## Related

- OA peers: [Amazon](Amazon.md) · [Google](Google.md) · [Microsoft](Microsoft.md) · [Goldman Sachs](Goldman%20Sachs.md)
- Sliding window basis: [Fixed Size Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
