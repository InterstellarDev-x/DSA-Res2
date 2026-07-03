> **Topic:** [Advanced Strings](../README.md) · **Section:** Design

# Rolling Hash (Rabin-Karp)

A **rolling hash** turns a string into a number so that two substrings can be compared in **O(1)** after O(n) preprocessing. The classic use is **Rabin-Karp** substring matching, but the same primitive powers Longest Duplicate Substring, Distinct Echo Substrings, plagiarism detection, and substring deduplication.

The design question almost always reduces to: *"compare many substrings for equality, fast."* Hashing answers it — provided you handle overflow, negative remainders, and spurious collisions correctly.

---

## 1. Polynomial Hash

Treat a string `s = s[0] s[1] ... s[m-1]` as a base-`B` number under a prime modulus `MOD`:

```
hash(s) = (s[0]·B^(m-1) + s[1]·B^(m-2) + ... + s[m-1]·B^0) mod MOD
```

- **Base `B`:** any value larger than the alphabet works. `31` is the canonical small base for lowercase strings; `256` treats each byte as a digit. Both are fine — pick one and stay consistent.
- **Modulus `MOD`:** a large prime, `1_000_000_007`. A prime modulus spreads hashes uniformly and keeps the probability of a random collision near `1/MOD`.
- **`i64` everywhere:** `B·hash` can reach `31 · 10^9 ≈ 3.1·10^10`, which overflows a 32-bit `i32`. Use `i64` and reduce mod `MOD` after every multiply.

---

## 2. Rolling Update Formula

The "rolling" part: when the window slides one character to the right (drop `s[i]`, add `s[i+m]`), recompute the hash in O(1) instead of O(m):

```
newHash = ( (oldHash - s[i]·B^(m-1)) · B + s[i+m] ) mod MOD
```

- Subtract the contribution of the leaving character (its weight is the top power `B^(m-1)`).
- Multiply the whole window by `B` (every remaining character shifts up one power).
- Add the entering character (weight `B^0`).

Because subtraction can produce a **negative** value under modular arithmetic, always normalize:

```rust
hash = ((hash % MOD) + MOD) % MOD;
```

---

## 3. Full Rabin-Karp — Single Hash

```rust
struct RabinKarp;

impl RabinKarp {
    const MOD: i64 = 1_000_000_007;
    const BASE: i64 = 256; // each char treated as a base-256 digit

    // Returns the first index where pattern occurs in text, or None.
    fn search(text: &str, pattern: &str) -> Option<usize> {
        let text = text.as_bytes();
        let pattern = pattern.as_bytes();
        let n = text.len();
        let m = pattern.len();
        if m == 0 { return Some(0); }
        if m > n { return None; }

        // high_pow = BASE^(m-1) mod MOD  -> weight of the leftmost character.
        let mut high_pow: i64 = 1;
        for _ in 0..m - 1 {
            high_pow = (high_pow * Self::BASE) % Self::MOD;
        }

        let mut pattern_hash: i64 = 0;
        let mut window_hash: i64 = 0;
        for i in 0..m {
            pattern_hash = (pattern_hash * Self::BASE + pattern[i] as i64) % Self::MOD;
            window_hash = (window_hash * Self::BASE + text[i] as i64) % Self::MOD;
        }

        for i in 0..=(n - m) {
            // Hash match -> verify the actual characters (guard against spurious hits).
            if window_hash == pattern_hash && Self::matches(text, i, pattern) {
                return Some(i);
            }
            // Roll the window forward by one, unless we are at the last position.
            if i + m < n {
                let leaving = (text[i] as i64 * high_pow) % Self::MOD;
                window_hash = ((window_hash - leaving) % Self::MOD + Self::MOD) % Self::MOD; // negative-safe
                window_hash = (window_hash * Self::BASE + text[i + m] as i64) % Self::MOD;
            }
        }
        None
    }

    // Character-by-character verification; only called when hashes collide.
    fn matches(text: &[u8], start: usize, pattern: &[u8]) -> bool {
        for k in 0..pattern.len() {
            if text[start + k] != pattern[k] { return false; }
        }
        true
    }
}
```

**Why `matches`?** Two distinct substrings can share a hash (a **spurious hit** / collision). A hash match is *necessary* but not *sufficient* — never return on a hash match alone. The verification keeps Rabin-Karp **correct**; it stays O(n+m) on average because collisions are rare.

---

## 4. Spurious Hits and Collision Probability

With a single prime modulus near `10^9`, the chance any one comparison collides is about `1/MOD ≈ 10^-9`. Over `q` comparisons the expected number of false hits is about `q/MOD` — negligible for normal inputs, which is why verification is cheap.

But adversarial inputs (or anti-hash test cases on Codeforces, and occasionally on hard LeetCode problems) can be crafted to collide a single fixed `(BASE, MOD)`. The defense is **double hashing**.

---

## 5. Double Hashing

Maintain two independent hashes with different `(BASE, MOD)` pairs and treat two substrings as equal only when **both** match. The effective collision probability becomes roughly `1/(MOD1·MOD2) ≈ 10^-18`, small enough to skip character verification in many competitive settings (though verifying is still safest in interviews).

```rust
struct DoubleHash {
    h1: Vec<i64>, // prefix hashes
    h2: Vec<i64>,
    p1: Vec<i64>, // base powers
    p2: Vec<i64>,
    n: usize,
}

impl DoubleHash {
    const MOD1: i64 = 1_000_000_007;
    const MOD2: i64 = 998_244_353;
    const BASE1: i64 = 131;
    const BASE2: i64 = 137;

    fn new(s: &str) -> Self {
        let s = s.as_bytes();
        let n = s.len();
        let mut h1 = vec![0i64; n + 1];
        let mut h2 = vec![0i64; n + 1];
        let mut p1 = vec![0i64; n + 1];
        let mut p2 = vec![0i64; n + 1];
        p1[0] = 1;
        p2[0] = 1;
        for i in 0..n {
            h1[i + 1] = (h1[i] * Self::BASE1 + s[i] as i64) % Self::MOD1;
            h2[i + 1] = (h2[i] * Self::BASE2 + s[i] as i64) % Self::MOD2;
            p1[i + 1] = (p1[i] * Self::BASE1) % Self::MOD1;
            p2[i + 1] = (p2[i] * Self::BASE2) % Self::MOD2;
        }
        DoubleHash { h1, h2, p1, p2, n }
    }

    // Packed hash of substring s[l..r) (half-open), 0 <= l <= r <= n.
    fn hash(&self, l: usize, r: usize) -> i64 {
        let a = ((self.h1[r] - self.h1[l] * self.p1[r - l]) % Self::MOD1 + Self::MOD1) % Self::MOD1;
        let b = ((self.h2[r] - self.h2[l] * self.p2[r - l]) % Self::MOD2 + Self::MOD2) % Self::MOD2;
        a * Self::MOD2 + b // pack two 32-bit-ish values into one comparable i64
    }

    // True if s[l1..l1+len) equals s[l2..l2+len).
    fn equal(&self, l1: usize, l2: usize, len: usize) -> bool {
        self.hash(l1, l1 + len) == self.hash(l2, l2 + len)
    }
}
```

The **prefix-hash** form above (`h[r] - h[l]·B^(r-l)`) is the most reusable shape: build once in O(n), then query the hash of *any* substring in O(1). This is exactly what Longest Duplicate Substring needs.

---

## 6. Complexity

| Operation | Average | Worst |
|-----------|---------|-------|
| Rabin-Karp search | O(n + m) | O(nm) (all hashes collide, every window verified fully) |
| Prefix-hash build | O(n) | O(n) |
| Substring hash query | O(1) | O(1) |
| Space | O(1) (rolling) or O(n) (prefix array) | same |

The O(nm) worst case is the reason double hashing matters: it makes adversarial all-collide inputs astronomically unlikely, so the average case holds in practice.

---

## 7. Follow-ups

### Substring deduplication ("count distinct substrings of length k")
Slide a window of length `k`, push each window's hash into a `HashSet<i64>`, and the set size is the count of distinct length-`k` substrings. Use double hashing to avoid counting two different substrings as one. This generalizes to Distinct Echo Substrings (LC 1316), where you hash both halves and require them equal.

```rust
use std::collections::HashSet;

fn distinct_substrings_of_length(s: &str, k: usize) -> usize {
    let n = s.len();
    if k > n { return 0; }
    let dh = DoubleHash::new(s);
    let mut seen: HashSet<i64> = HashSet::new();
    for i in 0..=(n - k) {
        seen.insert(dh.hash(i, i + k));
    }
    seen.len()
}
```

### Longest Duplicate Substring (LC 1044) — binary search on length + Rabin-Karp
The answer length is **monotone**: if a duplicate of length `L` exists, one of length `L-1` exists too. Binary search `L` in `[1, n-1]`; for each `L`, hash every length-`L` window and check a `HashSet` for a repeat in O(n). Total O(n log n). Double hashing is essential here because LeetCode's hard cases probe single-hash collisions.

### Plagiarism / fingerprinting framing
Break each document into overlapping `k`-grams, hash each with a rolling hash, and compare the resulting sets of fingerprints (Jaccard similarity). Winnowing selects a deterministic subset of fingerprints so two documents that share a passage share fingerprints regardless of alignment — the production form of "compare many substrings fast."

---

## Cross-links

- [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) — the deterministic O(n+m) alternative to hashing for single-pattern search.
- [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) — uses these exact prefix hashes for Longest Duplicate Substring and Distinct Echo Substrings.
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md) — average vs worst case for all string algorithms.

---

> **Last Updated:** 2026-06-26
