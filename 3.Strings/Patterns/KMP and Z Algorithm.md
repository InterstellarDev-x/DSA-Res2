# KMP Algorithm & Z-Algorithm

> **Topic:** [Strings](../README.md) · **Difficulty:** Hard
> **Tags:** `KMP` `z-algorithm` `pattern-matching` `failure-function` `LPS`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [KMP — Rust Implementation](#kmp--rust-implementation)
6. [Z-Algorithm — Rust Implementation](#z-algorithm--rust-implementation)
7. [Common Mistakes](#common-mistakes)
8. [Applications](#applications)
9. [Practice Problems](#practice-problems)
10. [Related Patterns](#related-patterns)

---

## Pattern Overview

**KMP (Knuth-Morris-Pratt):**
Find all occurrences of pattern `p` in text `t` in O(n + m) by pre-computing the **Longest Proper Prefix which is also Suffix (LPS / failure function)** for the pattern. When a mismatch occurs, instead of restarting, the LPS array tells us the furthest valid restart point.

**Z-Algorithm:**
Computes an array `Z[i]` = length of the longest substring starting at `i` that is also a prefix of the string. O(n) construction. Equivalent power to KMP but often simpler to reason about.

```
KMP:  lps[i] = length of longest proper prefix of p[0..i] that is also a suffix
Z:    z[i]   = length of longest string starting from s[i] matching a prefix of s
```

---

## When to Use

- Find occurrences of a pattern in a text — O(n + m)
- Check if string A is a rotation of string B (search in A+A)
- Find shortest palindrome by prepending characters (reverse + KMP)
- Repeated string structure — check if pattern is a repeated unit
- Longest prefix that is also a suffix

---

## Recognition Cues

| Cue | Algorithm |
|-----|-----------|
| "find pattern in string" | KMP or Z |
| "is s a rotation of t?" | Search in `s+s` with KMP |
| "shortest palindrome by prepending" | Reverse + KMP LPS |
| "repeated substring pattern" | KMP LPS: if `n % (n - lps[n-1]) == 0` |
| "longest prefix = suffix" | KMP LPS array directly |
| "count distinct substrings" | [String Hashing](./String%20Hashing.md) |

---

## Complexity

| Algorithm | Build | Search | Space |
|-----------|-------|--------|-------|
| Naive search | — | O(n × m) | O(1) |
| KMP | O(m) | O(n) | O(m) |
| Z-Algorithm | O(n) | O(n) | O(n) |
| Rabin-Karp | O(n) | O(n) avg | O(1) |

---

## KMP — Rust Implementation

### Step 1: Build LPS (Failure Function)

```rust
// lps[i] = length of longest proper prefix of p[0..i] that is also a suffix
fn build_lps(pattern: &[u8]) -> Vec<usize> {
    let m = pattern.len();
    let mut lps = vec![0usize; m];
    let mut len = 0usize; // length of previous longest prefix suffix
    let mut i = 1;

    while i < m {
        if pattern[i] == pattern[len] {
            len += 1;
            lps[i] = len;
            i += 1;
        } else if len > 0 {
            len = lps[len - 1]; // don't increment i — try shorter prefix
        } else {
            lps[i] = 0;
            i += 1;
        }
    }
    lps
}
```

### Step 2: KMP Search

```rust
fn kmp_search(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    let mut indices = Vec::new();
    let n = text.len();
    let m = pattern.len();
    if m == 0 {
        return indices;
    }

    let lps = build_lps(pattern);
    let mut i = 0usize; // text index
    let mut j = 0usize; // pattern index

    while i < n {
        if text[i] == pattern[j] {
            i += 1;
            j += 1;
        }
        if j == m {
            indices.push(i - j); // found at i - m
            j = lps[j - 1]; // continue with partial match
        } else if i < n && text[i] != pattern[j] {
            if j > 0 {
                j = lps[j - 1]; // use LPS to skip
            } else {
                i += 1;
            }
        }
    }
    indices
}
// Total: O(n + m) | Space: O(m)
```

### Application 1: Repeated Substring Pattern (LC 459)

```rust
fn repeated_substring_pattern(s: &str) -> bool {
    let n = s.len();
    let lps = build_lps(s.as_bytes());
    let len = lps[n - 1];
    // If len > 0 and n is divisible by (n - len), s is built from a repeating unit
    len > 0 && n % (n - len) == 0
}
// Example: "abababab" → lps[7]=6 → n%(n-6) = 8%2 = 0 → true
```

### Application 2: Shortest Palindrome by Prepending (LC 214)

```rust
fn shortest_palindrome(s: &str) -> String {
    // Find longest palindromic prefix using KMP
    let rev: String = s.chars().rev().collect();
    let combined = format!("{}#{}", s, rev); // '#' prevents overlap
    let lps = build_lps(combined.as_bytes());
    let longest_palin_prefix = lps[combined.len() - 1];
    format!("{}{}", &rev[..s.len() - longest_palin_prefix], s)
}
// Time: O(n) | Space: O(n)
```

### Application 3: Is Rotation? (LC 796)

```rust
fn is_rotation(s: &str, goal: &str) -> bool {
    if s.len() != goal.len() {
        return false;
    }
    let doubled = format!("{}{}", s, s);
    !kmp_search(doubled.as_bytes(), goal.as_bytes()).is_empty()
}
// Time: O(n) | Space: O(n)
```

---

## Z-Algorithm — Rust Implementation

```rust
// z[i] = length of longest substring starting at s[i] that matches a prefix of s
// z[0] = 0 by convention (or n, depending on definition)
fn z_function(s: &[u8]) -> Vec<usize> {
    let n = s.len();
    let mut z = vec![0usize; n];
    let mut l = 0usize;
    let mut r = 0usize;

    for i in 1..n {
        if i < r {
            z[i] = (r - i).min(z[i - l]);
        }
        while i + z[i] < n && s[z[i]] == s[i + z[i]] {
            z[i] += 1;
        }
        if i + z[i] > r {
            l = i;
            r = i + z[i];
        }
    }
    z
}
```

### Z-Algorithm Pattern Search

```rust
fn z_search(text: &[u8], pattern: &[u8]) -> Vec<usize> {
    let m = pattern.len();
    let mut combined = Vec::with_capacity(m + 1 + text.len());
    combined.extend_from_slice(pattern);
    combined.push(b'$'); // '$' not in alphabet
    combined.extend_from_slice(text);
    let z = z_function(&combined);
    let mut result = Vec::new();
    for i in (m + 1)..combined.len() {
        if z[i] == m {
            result.push(i - m - 1); // offset in text
        }
    }
    result
}
// Time: O(n + m) | Space: O(n + m)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| KMP LPS: using `i++` after `len = lps[len-1]` | Do NOT increment `i` when falling back — re-check same `i` |
| KMP search: resetting `j = 0` on mismatch | Use `j = lps[j-1]` — preserves partial match information |
| Rotation check: `s + s` without `#` separator | `shortestPalindrome` must have separator to prevent LPS crossing boundary |
| Z-function: `z[0]` definition varies | Some define `z[0] = n`, some = 0 — be explicit |
| Repeated substring: checking `len > 0` only | Must also check `n % (n - len) == 0` |

---

## Applications

| Problem | Algorithm | Key Step |
|---------|-----------|----------|
| Pattern matching | KMP or Z | Standard |
| Rotation check | KMP | Search pattern in `s+s` |
| Shortest palindrome | KMP | LPS of `s + "#" + reverse(s)` |
| Repeated unit | KMP LPS | `n % (n - lps[n-1]) == 0` |
| Longest happy prefix | KMP LPS | LPS array value at last index |
| Count pattern occurrences | KMP | Count times `j == m` |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Find the Index of the First Occurrence](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | LC 28 |
| [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) | Easy | LC 459 |
| [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard | LC 214 |
| [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/) | Hard | LC 1392 |
| [String Matching in an Array](https://leetcode.com/problems/string-matching-in-an-array/) | Easy | LC 1408 |
| [Rotate String](https://leetcode.com/problems/rotate-string/) | Easy | LC 796 |
| Minimum Characters to Make Palindrome | Medium | GFG |

---

## Related Patterns

- [String Hashing](./String%20Hashing.md) — alternative O(n) pattern matching
- [Two Pointers on Strings](./Two%20Pointers%20on%20Strings.md) — palindrome problems
- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — window-based pattern search

---

> **Interview Tip:** KMP is rarely asked "implement KMP from scratch" in product-company interviews (Google/Meta/Amazon). It IS asked at quant/competitive firms (Jane Street, Citadel). Know it conceptually and be able to code the LPS array. More commonly, interviewers ask applications: repeated substring, rotation, shortest palindrome — all are KMP in disguise.

> **Last Updated:** 2026-06-26
