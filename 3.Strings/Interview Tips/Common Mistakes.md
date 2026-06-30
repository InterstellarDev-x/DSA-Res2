# Common Mistakes — Strings

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Top String Bugs

| # | Mistake | Buggy Code | Fix |
|---|---------|-----------|-----|
| 1 | String concatenation in loop | `result += s` in a loop | `StringBuilder.append()` |
| 2 | `==` for string equality | `if (s == t)` | `if (s.equals(t))` |
| 3 | `split(" ")` misses multiple spaces | `"  a  b".split(" ")` gives `["", "", "a", "", "b"]` | `split("\\s+")` then `trim()` |
| 4 | `c - 'a'` on uppercase char | `'A' - 'a'` = -32 → array index -32 | Lowercase first: `Character.toLowerCase(c) - 'a'` |
| 5 | `int[26]` for mixed-case input | Index out of bounds for uppercase | Use `int[128]` or normalize to lowercase |
| 6 | Not checking empty string | `s.charAt(0)` on `""` | Guard: `if (s.isEmpty()) return ...` |
| 7 | Sliding window: `if` instead of `while` to shrink | One shrink may not be enough | Always `while (invalid)` |
| 8 | KMP: `i++` after `len = lps[len-1]` | Wrong LPS values | Do NOT advance `i` when falling back |
| 9 | Palindrome: not trying both skip directions | `isPalin(s, l+1, r)` only | Also try `isPalin(s, l, r-1)` |
| 10 | `substring(l, r)` is exclusive of `r` | `s.substring(0, 3)` returns chars 0,1,2 | For inclusive `r`: `s.substring(l, r+1)` |

---

## Sliding Window String Mistakes

```java
// WRONG: frequency key bug
StringBuilder key = new StringBuilder();
for (int f : freq) key.append(f);
// "12" and "1" + "2" collision → use separator
key.append('#').append(f); // CORRECT: "#1#2#..."

// WRONG: formed counter uses >=
if (window[rc] >= need[rc]) formed++; // double-counts
// CORRECT: only exact match triggers formed
if (need[rc] > 0 && window[rc] == need[rc]) formed++;
```

---

## Palindrome Mistakes

```java
// WRONG: comparing with == for Character objects
if (s.charAt(l) == s.charAt(r)) // OK for char (primitive)
if (Character.valueOf(s.charAt(l)) == Character.valueOf(s.charAt(r))) // WRONG for boxed

// WRONG: palindrome II only trying one skip direction
if (s.charAt(l) != s.charAt(r))
    return isPalin(s, l+1, r); // WRONG - misses cases
    // CORRECT:
    return isPalin(s, l+1, r) || isPalin(s, l, r-1);
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Java String API](./Java%20String%20API.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
