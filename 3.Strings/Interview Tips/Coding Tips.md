# Coding Tips — Strings (Rust)

> **Topic:** [Strings](../README.md) · **Section:** Interview Tips

---

## Rust String Essentials

```rust
// String is owned and mutable; push_str is O(n) amortized
let mut result = String::new();
for s in &list { result.push_str(s); } // String is mutable in Rust

// For complex formatting, prefer a String buffer or collect
let mut sb = String::new();
for s in &list { sb.push_str(s); }
let result = sb;
```

## Char Access Patterns

```rust
let c = s.as_bytes()[i] as char;       // O(1) via byte index
// iterate string: for c in s.chars()
let freq = c as usize - 'a' as usize;  // 0-25 for lowercase letters
let ascii = c as u8;                   // full ASCII value

c.is_alphabetic()                      // true for a-z, A-Z
c.is_ascii_digit()                     // true for 0-9
c.is_alphanumeric()                    // used in palindrome problems
c.to_ascii_lowercase()                 // case-insensitive compare
c.to_ascii_uppercase()
```

## Common Frequency Array Patterns

```rust
// Lowercase only
let mut freq = vec![0i32; 26];
for c in s.chars() { freq[c as usize - 'a' as usize] += 1; }

// All ASCII
let mut freq = vec![0i32; 128];
for c in s.chars() { freq[c as usize] += 1; }

// Check equal frequency
freq1 == freq2;                    // O(26) — fast
```

## String Comparison & Operations

```rust
s == t                                          // content equality
// equalsIgnoreCase → convert both with to_ascii_lowercase() first
s.cmp(t)                                        // lexicographic; returns Ordering::Equal if equal
s.contains(sub)                                 // O(n×m) — use KMP for O(n+m)
s.starts_with(prefix)                           // startsWith equivalent
s.find(c)                                       // first occurrence, returns Option<usize>
&s[l..r]                                        // [l, r) exclusive; O(r-l)
// split on whitespace → s.split_whitespace()
let s = s.trim_start().to_string();             // trim leading
let s = s.trim_end().to_string();               // trim trailing
let s: String = s.chars().map(|c| if c == 'a' { 'b' } else { c }).collect(); // char replacement
let s: String = s.chars().rev().collect();      // reverse
```

## Interview Quick-Start Template

```rust
use std::collections::HashMap;
// 1. Always clarify charset first
// 2. Choose data structure:
//    - vec![0i32; 26]       → lowercase only
//    - vec![0i32; 128]      → ASCII
//    - HashMap              → Unicode or unknown

// 3. Sliding window skeleton
let mut l = 0usize;
let mut max_len = 0usize;
let s_bytes = s.as_bytes();
let mut window: HashMap<u8, i32> = HashMap::new();
for r in 0..s.len() {
    *window.entry(s_bytes[r]).or_insert(0) += 1;
    while /* invalid: fill in condition */ false {
        let count = window.entry(s_bytes[l]).or_insert(0);
        *count -= 1;
        if *count == 0 { window.remove(&s_bytes[l]); }
        l += 1;
    }
    max_len = max_len.max(r - l + 1);
}
```

---

## Related Files

- [Java String API](./Java%20String%20API.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Strings README](../README.md)

> **Last Updated:** 2026-06-26
