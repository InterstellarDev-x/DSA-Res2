> **Topic:** [Advanced Strings](../README.md) · **Tips 1 of 4**

# Coding Tips — Advanced Strings

Eight battle-tested tips for writing correct string-algorithm code under interview pressure.

---

### Tip 1 — KMP vs just using `find`

`haystack.find(needle)` already does O(n+m) substring search in Rust. Reach for **hand-written KMP** only when: library calls are forbidden, the interviewer asks for the algorithm, you need guaranteed worst-case linear, or you search the *same* pattern across many texts (precompute `lps` once). State the pragmatic answer first, then offer KMP.

```rust
let idx: Option<usize> = haystack.find(needle); // O(n+m), use this unless told otherwise
```

---

### Tip 2 — Memorize the LPS construction template

The prefix function is the spine of this whole topic. Drill this until it is muscle memory:

```rust
let s: Vec<u8> = s.as_bytes().to_vec();
let mut lps = vec![0usize; n];
let mut len = 0usize;
let mut i = 1usize;
while i < n {
    if s[i] == s[len] {
        len += 1;
        lps[i] = len;
        i += 1;
    } else if len > 0 {
        len = lps[len - 1]; // fall back; do NOT advance i
    } else {
        lps[i] = 0;
        i += 1;
    }
}
```

The single most common bug is advancing `i` on the fallback branch. On a mismatch with `len > 0`, only `len` changes.

---

### Tip 3 — Expand-around-center is usually enough

For Longest Palindromic Substring / Palindromic Substrings, **expand-around-center** (O(n²), O(1) space) is the right default — short and obviously correct. Mention **Manacher (O(n))** only if the interviewer explicitly asks for linear time. Implementing Manacher unprompted wastes time and invites bugs.

```rust
fn expand(s: &[u8], mut l: i32, mut r: i32) -> i32 {
    while l >= 0 && (r as usize) < s.len() && s[l as usize] == s[r as usize] {
        l -= 1;
        r += 1;
    }
    r - l - 1
}
```

---

### Tip 4 — Rolling hash: always `i64` + prime mod

A polynomial hash overflows 32-bit `i32` instantly. Use `i64`, a prime modulus, and reduce after every multiply.

```rust
const MOD: i64 = 1_000_000_007;
const BASE: i64 = 131; // or 31 / 256
hash = (hash * BASE + s[i] as i64) % MOD;
```

---

### Tip 5 — `(x % m + m) % m` for negative remainders

Rolling-hash updates **subtract** the leaving character, which can go negative. Rust's `%` keeps the sign of the dividend, so normalize:

```rust
window_hash = ((window_hash - leaving) % MOD + MOD) % MOD; // negative-safe
```

Forgetting this gives sporadic wrong answers that pass small tests and fail large ones.

---

### Tip 6 — Verify on a hash collision

A hash match is **necessary, not sufficient**. Two distinct substrings can share a hash (spurious hit). On every hash match, either confirm the characters or use double hashing.

```rust
if window_hash == pattern_hash && &text[i..i + m] == pattern {
    return Some(i);
}
```

---

### Tip 7 — Z-array and LPS carry the same information

`lps[i]` = longest border *ending* at `i`; `z[i]` = longest prefix-match *starting* at `i`. Either solves single-pattern matching in O(n+m) — pick whichever coordinate system the problem makes natural (concatenate `pattern + '#' + text` for Z; build `lps` on the pattern for KMP). Don't waste energy converting between them mid-interview.

---

### Tip 8 — A string of length `n` has `2n-1` palindrome centers

`n` odd-length centers (each character) plus `n-1` even-length centers (each gap). When counting or searching palindromes, iterate *both* per index:

```rust
for i in 0..n {
    count += expand(&s, i as i32, i as i32);       // odd center
    count += expand(&s, i as i32, i as i32 + 1);   // even center
}
```

Missing the even centers silently drops every even-length palindrome — a classic off-by-one bug.

---

## Related

- [Common Mistakes](Common%20Mistakes.md) · [Algorithm Selection](Algorithm%20Selection.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
