# Java String API Reference

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Most Used Methods (Interview Context)

| Method | Returns | Time | Notes |
|--------|---------|------|-------|
| `s.charAt(i)` | `char` | O(1) | Preferred over `toCharArray()` for single access |
| `s.length()` | `int` | O(1) | |
| `s.substring(l, r)` | `String` | O(r-l) | Exclusive `r` |
| `s.indexOf(c)` | `int` | O(n) | First occurrence; -1 if absent |
| `s.contains(sub)` | `boolean` | O(n×m) | Use KMP for O(n+m) |
| `s.equals(t)` | `boolean` | O(n) | Content equality |
| `s.equalsIgnoreCase(t)` | `boolean` | O(n) | |
| `s.compareTo(t)` | `int` | O(n) | Lexicographic; 0 = equal |
| `s.startsWith(p)` | `boolean` | O(p.len) | |
| `s.endsWith(p)` | `boolean` | O(p.len) | |
| `s.toCharArray()` | `char[]` | O(n) | One-time copy |
| `s.trim()` | `String` | O(n) | Leading/trailing whitespace |
| `s.strip()` | `String` | O(n) | Unicode-aware trim (Java 11+) |
| `s.split(regex)` | `String[]` | O(n) | `"\\s+"` for multiple spaces |
| `s.replace(old, new)` | `String` | O(n) | Returns new string |
| `s.toLowerCase()` | `String` | O(n) | |
| `s.toUpperCase()` | `String` | O(n) | |
| `String.valueOf(n)` | `String` | O(digits) | Int → String |
| `Integer.parseInt(s)` | `int` | O(n) | |
| `String.join(delim, arr)` | `String` | O(total) | |

## StringBuilder (Use Instead of String +)

| Method | Notes |
|--------|-------|
| `sb.append(s)` | O(1) amortized |
| `sb.insert(i, s)` | O(n) |
| `sb.deleteCharAt(i)` | O(n) |
| `sb.reverse()` | O(n) |
| `sb.toString()` | O(n) |
| `sb.setLength(0)` | Clear without new allocation |
| `sb.charAt(i)` | O(1) |
| `sb.length()` | O(1) |

## Character Utility Methods

```java
Character.isLetter(c)          // a-z, A-Z
Character.isDigit(c)           // 0-9
Character.isLetterOrDigit(c)   // alphanumeric
Character.isWhitespace(c)      // space, tab, newline
Character.isUpperCase(c)
Character.isLowerCase(c)
Character.toUpperCase(c)
Character.toLowerCase(c)
```

## Char Arithmetic

```java
c - 'a'              // 0-25 for lowercase
c - 'A'              // 0-25 for uppercase
c - '0'              // 0-9 for digit characters
(char)('a' + k)      // k-th lowercase letter
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
