# Complexity Analysis — Strings

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Pattern-to-Complexity Quick Reference

| Pattern | Time | Aux Space | Notes |
|---------|------|-----------|-------|
| [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) — fixed | O(n) | O(1) — `int[26]` | `Arrays.equals` is O(26) per step |
| [Sliding Window](../Patterns/Sliding%20Window%20on%20Strings.md) — variable | O(n) | O(charset) | `formed` counter avoids map scan |
| [Two Pointers](../Patterns/Two%20Pointers%20on%20Strings.md) | O(n) | O(1) if on char[] | `toCharArray()` costs O(n) space |
| [Frequency Count](../Patterns/Frequency%20Count.md) | O(n) | O(1) — fixed array | |
| Group Anagrams (sort key) | O(n × k log k) | O(n × k) | Sort each string |
| Group Anagrams (freq key) | O(n × k) | O(n × k) | No sort |
| [KMP](../Patterns/KMP%20and%20Z%20Algorithm.md) — build LPS | O(m) | O(m) | m = pattern length |
| [KMP](../Patterns/KMP%20and%20Z%20Algorithm.md) — search | O(n) | O(1) | After LPS built |
| [Z-Algorithm](../Patterns/KMP%20and%20Z%20Algorithm.md) | O(n) | O(n) | Combined build+search |
| [Rabin-Karp](../Patterns/String%20Hashing.md) | O(n+m) avg | O(m) | O(n×m) worst |
| Expand around center (palindrome) | O(n) | O(1) | Preferred over Manacher's |
| DP palindrome (longest palindromic substring) | O(n²) | O(n²) | Worse than expand |
| Count distinct substrings (hashing) | O(n²) | O(n²) | All n² hashes stored |
| Longest duplicate substring (BS + hash) | O(n log n) | O(n) | |

---

## Space Complexity Traps

| Scenario | Appears to be O(1) | Actually |
|----------|-------------------|---------|
| `s.toCharArray()` | — | O(n) — copies the string |
| `s.substring(l, r)` | — | O(r-l) — new string object |
| `String.split(...)` | — | O(n) — creates array of strings |
| `new StringBuilder(s)` | — | O(n) — copies content |
| `int[26]` freq array | ✅ O(1) | O(1) — constant 26 slots |

---

## How to State Complexity for Sliding Window

> "This is O(n) time. Each character enters the window once (r advances n times) and leaves once (l advances at most n times). So the total work is O(2n) = O(n). The frequency array is `int[26]`, which is O(1) space."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
