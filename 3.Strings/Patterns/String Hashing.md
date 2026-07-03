# String Hashing (Rabin-Karp)

> **Topic:** [Strings](../README.md) · **Difficulty:** Hard
> **Tags:** `rabin-karp` `rolling-hash` `polynomial-hash` `substring-search`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Rust Templates](#rust-templates)
6. [Common Mistakes](#common-mistakes)
7. [Applications](#applications)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

String Hashing maps a string to a numeric fingerprint (hash) such that:
- Equal strings → same hash (guaranteed)
- Different strings → different hash (with very high probability if hash chosen carefully)

**Polynomial rolling hash:**
```
hash(s[0..n-1]) = s[0]*B^(n-1) + s[1]*B^(n-2) + ... + s[n-1]*B^0   (mod M)
```

**Rolling hash** enables O(1) hash of any substring `s[l..r]` after O(n) preprocessing:
```
hash(s[l..r]) = (prefix[r+1] - prefix[l] * B^(r-l+1)) mod M
```

**Rabin-Karp algorithm** uses rolling hashes to find pattern occurrences in O(n + m) average time.

---

## When to Use

- Find all occurrences of pattern in text (same power as KMP)
- Count **distinct substrings**
- Find **longest common substring** between two strings
- Check if two large strings are equal after transformation
- Binary search on answer length + hash verification

---

## Recognition Cues

| Cue | Approach |
|-----|----------|
| "count distinct substrings" | Hash all substrings, count unique hashes |
| "longest repeated substring" | Binary search on length + rolling hash set |
| "find pattern, avoid worst case" | Rabin-Karp |
| "longest common substring" | Binary search + hashing |
| "check if rotation" | Hash of s == hash of any rotation |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix hash | O(n) | O(n) |
| Substring hash query | O(1) | — |
| Rabin-Karp search | O(n + m) average, O(nm) worst | O(m) |
| Count distinct substrings | O(n²) | O(n²) hashes stored |
| Longest repeated substring | O(n log n) | O(n) |

---

## Rust Templates

### 1. Polynomial Prefix Hash (Build + Query)

```rust
struct StringHasher {
    prefix: Vec<i64>,
    power: Vec<i64>,
}

impl StringHasher {
    const BASE: i64 = 131;
    const MOD: i64 = 1_000_000_007;

    fn new(s: &str) -> Self {
        let n = s.len();
        let bytes = s.as_bytes();
        let mut prefix = vec![0i64; n + 1];
        let mut power = vec![0i64; n + 1];
        power[0] = 1;

        for i in 0..n {
            prefix[i + 1] = (prefix[i] * Self::BASE + bytes[i] as i64) % Self::MOD;
            power[i + 1] = power[i] * Self::BASE % Self::MOD;
        }

        StringHasher { prefix, power }
    }

    // Hash of s[l..r] inclusive, 0-indexed
    fn hash_range(&self, l: usize, r: usize) -> i64 {
        (self.prefix[r + 1] - self.prefix[l] * self.power[r - l + 1] % Self::MOD
            + Self::MOD * Self::MOD)
            % Self::MOD
    }
}
// Build: O(n) | Query: O(1)
```

### 2. Rabin-Karp Pattern Search

```rust
fn rabin_karp(text: &str, pattern: &str) -> Vec<usize> {
    let mut result = Vec::new();
    let n = text.len();
    let m = pattern.len();
    if m > n {
        return result;
    }

    let th = StringHasher::new(text);
    let ph = StringHasher::new(pattern);
    let pat_hash = ph.hash_range(0, m - 1);
    let text_bytes = text.as_bytes();
    let pat_bytes = pattern.as_bytes();

    for i in 0..=(n - m) {
        if th.hash_range(i, i + m - 1) == pat_hash {
            // Verify to handle hash collisions
            if &text_bytes[i..i + m] == pat_bytes {
                result.push(i);
            }
        }
    }
    result
}
// Time: O(n + m) average | O(nm) worst (hash collisions)
```

### 3. Count Distinct Substrings

```rust
use std::collections::HashSet;

fn count_distinct_substrings(s: &str) -> usize {
    let n = s.len();
    let mut seen: HashSet<i64> = HashSet::new();
    let hasher = StringHasher::new(s);

    for len in 1..=n {
        for i in 0..=(n - len) {
            seen.insert(hasher.hash_range(i, i + len - 1));
        }
    }
    seen.len()
    // Time: O(n²) | Space: O(n²)
}
```

### 4. Longest Repeated Substring (Binary Search + Hash)

```rust
use std::collections::HashSet;

fn find_repeated(h: &StringHasher, n: usize, len: usize) -> Option<usize> {
    let mut seen: HashSet<i64> = HashSet::new();
    for i in 0..=(n - len) {
        let hash_val = h.hash_range(i, i + len - 1);
        if !seen.insert(hash_val) {
            return Some(i - 1); // collision possible; accept for now
        }
    }
    None
}

fn longest_repeated_substring(s: &str) -> String {
    let n = s.len();
    let hasher = StringHasher::new(s);
    let mut lo = 1usize;
    let mut hi = n - 1;
    let mut best_start: Option<usize> = None;
    let mut best_len = 0usize;

    while lo <= hi {
        let mid = lo + (hi - lo) / 2;
        if let Some(start) = find_repeated(&hasher, n, mid) {
            best_start = Some(start);
            best_len = mid;
            lo = mid + 1;
        } else {
            hi = mid - 1;
        }
    }
    match best_start {
        None => String::new(),
        Some(start) => s[start..start + best_len].to_string(),
    }
}
// Time: O(n log n) | Space: O(n)
```

### 5. Double Hashing (Reduce Collision Probability)

```rust
// Use two independent hash functions to reduce false positive probability
fn double_hash(s: &str, l: usize, r: usize) -> (i64, i64) {
    let h1 = hash_with(s, l, r, 131, 1_000_000_007);
    let h2 = hash_with(s, l, r, 137, 1_000_000_009);
    (h1, h2)
}
// Collision probability ≈ 1/(1e9+7) × 1/(1e9+9) ≈ 10^{-18}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Negative hash values | Add `MOD` before taking `% MOD`: `(val % MOD + MOD) % MOD` |
| Using `i32` for hash values | Use `i64` — intermediate products overflow `i32` |
| Not verifying on hash match | Always do `==` check when hash matches (collision guard) |
| Single hash with small MOD | Use double hashing or MOD ≈ 10^9+7 |
| `power` array not initialized | `power[0] = 1` then `power[i+1] = power[i] * BASE % MOD` |
| Substring bounds: `&s[l..l+len]` vs `&s[l..r+1]` | Rust's `&s[l..r+1]` uses exclusive end; use `&s[l..=r]` or `&s[l..r+1]` for inclusive `r` |

---

## Applications

| Application | Description |
|-------------|-------------|
| Plagiarism detection | Hash all substrings; find matching windows |
| Longest common substring | Binary search length + hash sets |
| Count distinct palindromes | Combine with palindrome expansion |
| Substring equality in O(1) | Two string hashers, compare queries |
| Rolling hash in competitive programming | Combine with other algorithms |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Implement strStr()](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | LC 28 |
| [Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) | Hard | LC 1044 |
| [Repeated String Match](https://leetcode.com/problems/repeated-string-match/) | Medium | LC 686 |
| [Longest Repeating Substring](https://leetcode.com/problems/longest-repeating-substring/) | Medium | LC 1062 |
| Count Distinct Substrings | Medium | GFG |

---

## Related Patterns

- [KMP and Z-Algorithm](./KMP%20and%20Z%20Algorithm.md) — deterministic O(n+m) alternative
- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — fixed-length window
- [Frequency Count](./Frequency%20Count.md) — when char counts identify strings

---

> **Interview Tip:** In product-company interviews, Rabin-Karp is rarely asked directly. The rolling hash technique appears as a building block for "longest duplicate substring" or "count distinct substrings" — problems that require O(1) substring comparison. Know the `(prefix[r+1] - prefix[l] * power[r-l+1]) % MOD` formula and the negative-modulo fix.

> **Last Updated:** 2026-06-26
