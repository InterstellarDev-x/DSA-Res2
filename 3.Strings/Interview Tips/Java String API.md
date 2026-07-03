# Rust String API Reference

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Most Used Methods (Interview Context)

| Method | Returns | Time | Notes |
|--------|---------|------|-------|
| `s.chars().nth(i)` or `s.as_bytes()[i] as char` | `char` | O(n) / O(1) | Direct char access; use bytes for ASCII-only strings |
| `s.len()` | `usize` | O(1) | |
| `&s[l..l+len]` or `s[l..l+len].to_string()` | `&str` / `String` | O(len) | Second arg is length, not end index; byte indices |
| `s.find(c)` | `Option<usize>` | O(n) | First occurrence; `None` if absent |
| `s.find(sub).is_some()` | `bool` | O(n×m) | Use KMP for O(n+m) |
| `s == t` | `bool` | O(n) | Content equality |
| `s.to_lowercase() == t.to_lowercase()` | `bool` | O(n) | Case-insensitive compare |
| `s.cmp(t)` | `Ordering` | O(n) | Lexicographic; `Ordering::Equal` = equal |
| `s.starts_with(p)` | `bool` | O(p.len) | Check prefix |
| `s.ends_with(p)` | `bool` | O(p.len) | Check suffix |
| `for c in s.chars()` | — | O(n) | Iterate chars; no separate copy needed |
| `s.trim()` | `&str` | O(n) | Leading/trailing whitespace |
| `s.trim()` | `&str` | O(n) | Unicode-aware by default |
| `s.split(delimiter).collect::<Vec<_>>()` | `Vec<&str>` | O(n) | Split on delimiter |
| string slicing + reconstruction | `String` | O(n) | No direct in-place replacement by pos/len |
| `s.to_lowercase()` or `s.make_ascii_lowercase()` | `String` / — | O(n) | Lowercase (new String or in-place for ASCII) |
| `s.to_uppercase()` or `s.make_ascii_uppercase()` | `String` / — | O(n) | Uppercase (new String or in-place for ASCII) |
| `n.to_string()` | `String` | O(digits) | Int → String |
| `s.parse::<i32>().unwrap()` | `i32` | O(n) | String → int |
| `parts.join(delimiter)` | `String` | O(total) | Join with delimiter |

## `String` (Rust equivalent of StringBuilder)

| Method | Notes |
|--------|-------|
| `s.push_str(x)` or `s += x` | O(1) amortized |
| `s.insert_str(i, str)` | O(n) |
| `s.remove(i)` or `s.drain(i..i+1)` | O(n) |
| `s = s.chars().rev().collect::<String>()` | O(n) |
| `s.clone()` or `s.to_string()` | O(1) — no conversion needed |
| `s.clear()` | Clear without new allocation |
| `s.chars().nth(i)` or `s.as_bytes()[i] as char` | O(n) / O(1) |
| `s.len()` | O(1) |

## Character Utility Methods

```rust
c.is_alphabetic()    // a-z, A-Z
c.is_ascii_digit()   // 0-9
c.is_alphanumeric()  // alphanumeric
c.is_whitespace()    // space, tab, newline
c.is_uppercase()
c.is_lowercase()
c.to_ascii_uppercase()
c.to_ascii_lowercase()
```

## Char Arithmetic

```rust
c as u8 - b'a'       // 0-25 for lowercase
c as u8 - b'A'       // 0-25 for uppercase
c as u8 - b'0'       // 0-9 for digit characters
(b'a' + k) as char   // k-th lowercase letter
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-07-03
