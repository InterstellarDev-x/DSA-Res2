# Common Mistakes — Strings

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Top String Bugs

| # | Mistake | Buggy Code | Fix |
|---|---------|-----------|-----|
| 1 | String concatenation in loop | `result += s` in a loop | use `String::with_capacity()` + `push_str()` |
| 2 | `==` for string equality (Java pitfall) | `if (s == t)` | In Rust, `==` works correctly for `String` and `&str` value comparison |
| 3 | `split(" ")` misses multiple spaces | `"  a  b".split(" ")` gives empty tokens | Use `.split_whitespace()` to skip whitespace tokens |
| 4 | `c - 'a'` on uppercase char | `'A' as usize - 'a' as usize` = wraps/wrong → array index error | Lowercase first: `c.to_ascii_lowercase() as usize - 'a' as usize` |
| 5 | `[i32; 26]` for mixed-case input | Index out of bounds for uppercase | Use `[i32; 128]` or normalize to lowercase |
| 6 | Not checking empty string | `s.as_bytes()[0]` on `""` | Guard: `if s.is_empty() { return ... }` |
| 7 | Sliding window: `if` instead of `while` to shrink | One shrink may not be enough | Always `while (invalid)` |
| 8 | KMP: `i += 1` after `len = lps[len-1]` | Wrong LPS values | Do NOT advance `i` when falling back |
| 9 | Palindrome: not trying both skip directions | `is_palin(s, l+1, r)` only | Also try `is_palin(s, l, r-1)` |
| 10 | `&s[start..start+len]` takes byte offset, not char count | `&s[0..3]` returns bytes 0,1,2 (length=3) | For inclusive `r`: `&s[l..r+1]` (byte-safe) or collect chars |

---

## Sliding Window String Mistakes

```rust
// WRONG: frequency key bug
let mut key = String::new();
for f in &freq { key += &f.to_string(); }
// "12" and "1" + "2" collision → use separator
key.push('#'); key += &f.to_string(); // CORRECT: "#1#2#..."

// WRONG: formed counter uses >=
if window[rc] >= need[rc] { formed += 1; } // double-counts
// CORRECT: only exact match triggers formed
if need[rc] > 0 && window[rc] == need[rc] { formed += 1; }
```

---

## Palindrome Mistakes

```rust
// OK: comparing chars directly (no boxing issue in Rust)
let chars: Vec<char> = s.chars().collect();
if chars[l] == chars[r] { } // OK for char (Rust chars are Unicode scalar values)
// Note: Rust chars are plain values, no boxing issue

// WRONG: palindrome II only trying one skip direction
if chars[l] != chars[r] {
    return is_palin(s, l + 1, r); // WRONG - misses cases
    // CORRECT:
    return is_palin(s, l + 1, r) || is_palin(s, l, r - 1);
}
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Java String API](./Java%20String%20API.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
