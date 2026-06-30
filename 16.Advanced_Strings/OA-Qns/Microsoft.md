> **Topic:** [Advanced Strings](../README.md) · **Section:** OA-Qns

# Microsoft — Advanced Strings (Online Assessment)

Microsoft favors clean, well-explained classics over exotic algorithms. `strStr` is the staple; Repeated Substring Pattern tests whether you understand string **periodicity**; Longest Palindromic Substring is the medium anchor.

| Problem | Frequency | Difficulty | Pattern | LC | Source |
|---------|-----------|------------|---------|----|--------|
| Implement strStr() | ⭐⭐⭐⭐ | Easy | [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) | [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | OA |
| Repeated Substring Pattern | ⭐⭐⭐ | Easy | [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) | [459](https://leetcode.com/problems/repeated-substring-pattern/) | OA |
| Longest Palindromic Substring | ⭐⭐⭐ | Medium | [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md) | [5](https://leetcode.com/problems/longest-palindromic-substring/) | OA |

## Follow-ups Microsoft asks

- **strStr:** "Explain the failure function as if I'm not an algorithms person." → the interviewer literally rates communication; see [Interview Problems → Microsoft](../Interview%20Problems/Microsoft.md).
- **Repeated Substring Pattern:** "Prove `n % (n - lps[n-1]) == 0` detects periodicity." Also the slick trick `(s+s).indexOf(s,1) != n` — be ready to justify it.
- **Longest Palindromic Substring:** "Handle even-length palindromes." → two center types; "return the length only" simplifies the return.

## What interviewers look for

- Communication: can you teach KMP to a non-expert?
- The `lps[n-1] != 0` guard in Repeated Substring Pattern (else `"abc"` falsely passes).
- Both odd and even centers in expand-around-center.
- Defensive checks for empty inputs and single-character strings.

## Related

- Deep dives: [Interview Problems → Microsoft](../Interview%20Problems/Microsoft.md)
- Other companies: [Amazon](Amazon.md) · [Google](Google.md) · [Goldman Sachs](Goldman%20Sachs.md) · [Adobe](Adobe.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
