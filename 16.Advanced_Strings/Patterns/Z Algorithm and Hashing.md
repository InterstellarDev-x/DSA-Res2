> **Topic:** [Advanced Strings](../README.md) · **Pattern 2 of 3**

# Z Algorithm & Polynomial Hashing

Two linear-time string tools that complement KMP. The **Z-array** is a per-index "how much of the prefix repeats here" function and gives an alternative O(n+m) matcher. **Polynomial rolling hash** turns any substring into an O(1)-comparable number, powering Rabin-Karp, duplicate detection, and binary-search-on-length problems.

---

## 1. The Z-array

For a string `s` of length `n`:

> `z[i]` = the length of the longest substring starting at index `i` that is **also a prefix** of `s`. By convention `z[0] = 0` (or `n`; we use 0 and treat the whole-string case separately).

Example: for `s = "aabxaab"`, `z = [_, 1, 0, 0, 3, 1, 0]` — at index 4 the substring `"aab"` matches the prefix `"aab"`, so `z[4] = 3`.

### Construction with the [L, R] window

We maintain a window `[L, R]` — the rightmost segment we have seen that is *known to match a prefix*. For a new `i`:

- If `i < R`, the mirror index `i - L` already has a `z` value we can reuse: `z[i] = min(R - i, z[i - L])`.
- Then we try to **extend** by comparing characters past `R`.
- If we extended past `R`, update the window `L = i, R = i + z[i]`.

```rust
fn build_z(s: &str) -> Vec<usize> {
    let s = s.as_bytes();
    let n = s.len();
    let mut z = vec![0usize; n];
    let mut l = 0usize;
    let mut r = 0usize; // current rightmost match window [l, r)
    for i in 1..n {
        if i < r {
            z[i] = (r - i).min(z[i - l]); // reuse mirror, capped at window
        }
        // try to extend the match beyond what we reused
        while i + z[i] < n && s[z[i]] == s[i + z[i]] {
            z[i] += 1;
        }
        if i + z[i] > r {
            // extended past r -> slide the window
            l = i;
            r = i + z[i];
        }
    }
    z
}
```

The `min(R - i, z[i - L])` cap is the subtle part: the mirror value may claim more than fits inside the window, so we clamp to `R - i` and verify the rest by explicit comparison. Each character is compared O(1) amortized → O(n) total.

---

## 2. Z-based pattern matching

To find pattern `P` inside text `T`, build the Z-array of `P + '$' + T` where `$` does not occur in either string. Every index `i` (in the `T` region) with `z[i] == |P|` marks an occurrence.

```rust
fn z_search(text: &str, pattern: &str) -> Vec<usize> {
    let combined: String = format!("{}${}", pattern, text);
    let z = build_z(&combined);
    let m = pattern.len();
    let mut result = Vec::new();
    for i in (m + 1)..combined.len() {
        if z[i] == m {
            result.push(i - m - 1); // offset back into the original text
        }
    }
    result
}
```

`z[i] == m` means the next `m` characters fully equal the pattern prefix — a match. The separator `$` guarantees no `z` value can spill across the boundary.

---

## 3. Polynomial Rolling Hash Fundamentals

Treat a string as a number in base `B`:

> `hash(s) = (s[0]*B^(m-1) + s[1]*B^(m-2) + ... + s[m-1]*B^0) mod M`

- **Base `B`:** a value larger than the alphabet, commonly `31`, `131`, or `256`.
- **Modulus `M`:** a large prime such as `1_000_000_007i64` to keep values in `i64` range and spread collisions.
- **Precompute powers** `B^0..B^(m)` once so a window hash updates in O(1).

```rust
const MOD: i64 = 1_000_000_007;
const BASE: i64 = 131;

// Precompute base powers up to length n.
fn powers(n: usize) -> Vec<i64> {
    let mut p = vec![0i64; n + 1];
    p[0] = 1;
    for i in 1..=n {
        p[i] = p[i - 1] * BASE % MOD;
    }
    p
}
```

Rolling forward by one character (drop leftmost, append rightmost):
`newHash = ((oldHash - s[i] * B^(m-1)) * B + s[i+m]) mod M`, always re-normalizing with `(x % MOD + MOD) % MOD` to kill negatives. Full mechanics live in [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md).

---

## 4. Problem 7 — Find All Anagrams in a String (LC 438, Medium)

> Return the start indices of all substrings of `s` that are anagrams of `p`.

An anagram is purely a **multiset of characters**, so a 26-slot frequency window is the natural, collision-free tool. This is fundamentally a fixed-size sliding window — see the [Sliding Window topic](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md).

```rust
struct Solution;

impl Solution {
    pub fn find_anagrams(s: String, p: String) -> Vec<i32> {
        let mut res = Vec::new();
        let s = s.as_bytes();
        let p_bytes = p.as_bytes();
        let n = s.len();
        let m = p_bytes.len();
        if n < m {
            return res;
        }
        let mut need = [0i32; 26];
        let mut win = [0i32; 26];
        for &c in p_bytes {
            need[(c - b'a') as usize] += 1;
        }
        for i in 0..n {
            win[(s[i] - b'a') as usize] += 1; // add right edge
            if i >= m {
                win[(s[i - m] - b'a') as usize] -= 1; // remove left edge
            }
            if i >= m - 1 && win == need {
                res.push((i - m + 1) as i32);
            }
        }
        res
    }
}
```

**Complexity:** O(n * 26) ≈ O(n) time, O(1) space. `==` on two `[i32; 26]` of size 26 is constant.

**Hashing alternative.** Instead of comparing arrays you can keep a *commutative* hash of the window — e.g. a fixed-length 26-vector difference counter, or a polynomial over the sorted multiset. Frequency comparison is simpler and exact, so prefer it for anagrams; rolling hash earns its keep when *order matters* (exact substring), not for anagrams.

---

## 5. Problem 9a — Longest Duplicate Substring (LC 1044, Hard)

> Find the longest substring that occurs at least twice (may overlap). Return any such substring (or "" if none).

**Strategy: binary search on the length + Rabin-Karp.** A duplicate of length `L` existing implies a duplicate of every length `< L` exists (a prefix of it), so the predicate "exists a duplicate of length `L`" is monotonic — binary-search `L` in `[1, n-1]`. For a fixed `L`, slide a rolling hash over all length-`L` windows and store hashes in a set; a repeat hash (verified) means a duplicate.

```rust
use std::collections::HashMap;

struct Solution;

impl Solution {
    const MOD: u64 = (1u64 << 61) - 1; // large Mersenne prime, room in u64
    const BASE: u64 = 131;

    pub fn longest_dup_substring(s: String) -> String {
        let n = s.len();
        let a: Vec<u64> = s.bytes().map(|b| (b - b'a' + 1) as u64).collect();
        let mut lo = 1i32;
        let mut hi = (n as i32) - 1;
        let mut start: i32 = -1;
        let mut best_len = 0usize;
        while lo <= hi {
            let mid = (lo + hi) >> 1;
            let pos = Self::search(&a, mid as usize);
            if pos != -1 {
                start = pos;
                best_len = mid as usize;
                lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
        if start == -1 {
            String::new()
        } else {
            s[start as usize..start as usize + best_len].to_string()
        }
    }

    // Returns the start index of some duplicated substring of length l, or -1.
    fn search(a: &[u64], l: usize) -> i32 {
        let mut pow = 1u64;
        for _ in 0..l {
            pow = Self::mulmod(pow, Self::BASE);
        }
        let mut h = 0u64;
        for i in 0..l {
            h = (Self::mulmod(h, Self::BASE) + a[i]) % Self::MOD;
        }
        let mut seen: HashMap<u64, Vec<usize>> = HashMap::new();
        seen.entry(h).or_default().push(0);
        for i in l..a.len() {
            h = (Self::mulmod(h, Self::BASE) + a[i] + Self::MOD * Self::MOD
                - Self::mulmod(a[i - l], pow) % Self::MOD)
                % Self::MOD;
            let start_idx = i - l + 1;
            // verify to defeat spurious hash hits (collision safety)
            let found = if let Some(positions) = seen.get(&h) {
                positions.iter().any(|&j| Self::equals_at(a, j, start_idx, l))
            } else {
                false
            };
            if found {
                return start_idx as i32;
            }
            seen.entry(h).or_default().push(start_idx);
        }
        -1
    }

    fn equals_at(a: &[u64], x: usize, y: usize, l: usize) -> bool {
        a[x..x + l] == a[y..y + l]
    }

    fn mulmod(x: u64, y: u64) -> u64 {
        ((x as u128 * y as u128) % (Self::MOD as u128)) as u64 // fits because MOD < 2^61, x,y < MOD
    }
}
```

**Complexity:** O(n log n) average — `log n` binary-search steps, each an O(n) rolling pass. **Double hashing** (two independent (BASE, MOD) pairs, compare both) makes a collision astronomically unlikely and lets you skip the explicit verification; we kept the verification here for a guaranteed-correct answer.

---

## 6. Problem 9b — Distinct Echo Substrings (LC 1316, Hard)

> Count the number of **distinct** substrings that are a concatenation of some string with itself (i.e. of the form `a + a`, an "echo").

A substring `s[i..i+2k-1]` is an echo iff its first half `s[i..i+k-1]` equals its second half `s[i+k..i+2k-1]`. With prefix hashes, comparing two halves is O(1); store every distinct echo as a (hash) to dedupe.

```rust
use std::collections::HashSet;

struct Solution;

impl Solution {
    pub fn distinct_echo_substrings(text: String) -> i32 {
        let n = text.len();
        let text = text.as_bytes();
        const MOD: i64 = 1_000_000_007;
        const BASE: i64 = 131;
        let mut h = vec![0i64; n + 1];
        let mut pw = vec![0i64; n + 1];
        pw[0] = 1;
        for i in 0..n {
            h[i + 1] = (h[i] * BASE + text[i] as i64) % MOD;
            pw[i + 1] = pw[i] * BASE % MOD;
        }
        // hash of text[l..r) from prefix hashes
        let sub = |l: usize, r: usize| -> i64 {
            ((h[r] - h[l] * pw[r - l]) % MOD + MOD) % MOD
        };
        let mut distinct: HashSet<i64> = HashSet::new();
        for len in 1..=(n / 2) { // half-length
            for i in 0..=(n - 2 * len) {
                let first  = sub(i, i + len);
                let second = sub(i + len, i + 2 * len);
                if first == second {
                    distinct.insert(first); // dedupe by half-hash
                }
            }
        }
        distinct.len() as i32
    }
}
```

**Complexity:** O(n^2) hashing comparisons, O(n^2) distinct entries worst-case. Note the `(... + MOD) % MOD` to fix the negative produced by `h[r] - h[l]*pw[]`. For full collision safety in interviews, mention double hashing on the stored key.

---

## 7. Dry-run — Z-array of `"aabxaabxcaabxaabxay"`

Indices: `a a b x a a b x c a a b x a a b x a y` (length 19).

| i | reuse `min(R-i, z[i-L])` | extend? | z[i] | window [L,R] |
|---|--------------------------|---------|------|--------------|
| 1 | i≥R → 0 | s[0]='a'==s[1]='a', then s[1]='a'≠s[2]='b' → 1 | 1 | [1,2] |
| 2 | i≥R → 0 | s[0]='a'≠'b' | 0 | unchanged |
| 3 | 0 | 'a'≠'x' | 0 | unchanged |
| 4 | 0 | "aabx" matches prefix "aabx", stops at index 8 'c' | 4 | [4,8] |
| 5 | i<R: min(8-5, z[1]=1)=1 | s[1]='a'≠s[6]='b' | 1 | unchanged |
| 6 | min(8-6, z[2]=0)=0 | 'a'≠'b' | 0 | unchanged |
| 7 | min(8-7, z[3]=0)=0 | 'a'≠'x' | 0 | unchanged |
| 9 | i≥R → 0 | "aabxaabxa" matches the prefix for 9, stops at end-ish | 9 | [9,18] |
| 10..17 | reused from mirror inside [9,18] | clamp + verify | mirrors z[1..8] | — |

The big wins are `z[4]=4` (`"aabx"` repeats) and `z[9]=9` (`"aabxaabxa"` echoes the prefix). Inside the window `[9,18]` every later index copies its mirror in O(1), which is exactly why the algorithm is linear.

---

## Recognition Signals

| Signal in the prompt | Reach for |
|----------------------|-----------|
| "match prefix at each position", linear pattern search | Z-array on `P + '$' + T` |
| "anagrams", "permutation of p", order-independent | frequency window (see [Sliding Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md)) |
| "longest duplicated substring", overlapping allowed | binary search on length + Rabin-Karp |
| "echo" / "concatenation of a string with itself" | prefix-hash half comparison |
| "compare many substrings for equality fast" | polynomial prefix hashes, O(1) per compare |
| huge text, need average linear matching, hashing OK | rolling hash (Rabin-Karp) |

---

## Cross-links

- [String Matching (KMP)](String%20Matching%20(KMP).md) — Z and LPS are interchangeable linear matchers.
- [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md) — the hashing engine these problems lean on.
- [Sliding Window — Fixed Size Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md) — the canonical home of Find All Anagrams.

> **Last Updated:** 2026-06-26
