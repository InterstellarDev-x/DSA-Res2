# Microsoft — Strings Interview Questions

> **Topic:** [Strings](../README.md) · **Section:** Interview Problems
> **Tags:** `microsoft` `interview` `strings`

---

## Frequently Asked Questions

### 1. Valid Palindrome / Valid Palindrome II

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy / Easy |
| **Round** | Phone, R1 |
| **Pattern** | [Two Pointers](../Patterns/Two%20Pointers%20on%20Strings.md) |
| **LeetCode** | [LC 125](https://leetcode.com/problems/valid-palindrome/) / [LC 680](https://leetcode.com/problems/valid-palindrome-ii/) |
| **Follow-ups** | "At most 2 deletions?" → DP |

---

### 2. Longest Substring Without Repeating Chars

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, R1 |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) |

---

### 3. Reverse Words in a String

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1 |
| **Pattern** | [Two Pointers](../Patterns/Two%20Pointers%20on%20Strings.md) — split + reverse |
| **LeetCode** | [LC 151](https://leetcode.com/problems/reverse-words-in-a-string/) |
| **Follow-ups** | "O(1) space, in-place?" → Three-reverse trick on char array |

---

### 4. String to Integer (atoi)

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1 |
| **Skill** | State machine parsing, INT overflow |
| **LeetCode** | [LC 8](https://leetcode.com/problems/string-to-integer-atoi/) |

---

### 5. Permutation in String

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | R1 |
| **Pattern** | Fixed [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) |
| **LeetCode** | [LC 567](https://leetcode.com/problems/permutation-in-string/) |

---

## What Interviewers Usually Check

| Dimension | Detail |
|-----------|--------|
| **StringBuilder usage** | Never `String +=` in a loop |
| **Edge cases** | `""`, `" "`, `"a"`, all same chars |
| **Charset awareness** | Ask: lowercase only? ASCII? |
| **Two approaches** | Always give brute-force → optimal path |

---

## Related Files

- [Microsoft OA Questions](../OA-Qns/Microsoft.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window%20on%20Strings.md)
- [Two Pointers Pattern](../Patterns/Two%20Pointers%20on%20Strings.md)

> **Last Updated:** 2026-06-26
