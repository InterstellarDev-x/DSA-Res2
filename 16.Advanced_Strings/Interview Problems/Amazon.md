> **Topic:** [Advanced Strings](../README.md) · **Section:** Interview Problems

# Amazon — Advanced Strings (Interview Deep Dives)

Amazon's string bar is about **clean, correct, well-explained** solutions that map to the Leadership Principles (especially *Dive Deep* and *Insist on the Highest Standards*). The three headline problems are `strStr`, Longest Palindromic Substring, and Find All Anagrams.

---

## 1. Implement strStr() (LC 28) — naive vs KMP

> Return the index of the first occurrence of `needle` in `haystack`, or `-1`.

### Naive scan

```rust
struct Solution;

impl Solution {
    fn str_str_naive(haystack: String, needle: String) -> i32 {
        let n = haystack.len();
        let m = needle.len();
        if m == 0 { return 0; }
        let h: Vec<u8> = haystack.bytes().collect();
        let nd: Vec<u8> = needle.bytes().collect();
        for i in 0..n {
            if i + m > n { break; }
            let mut j = 0;
            while j < m && h[i + j] == nd[j] { j += 1; }
            if j == m { return i as i32; }
        }
        -1
    }
}
```

O(nm) worst case — consider `haystack = "aaaa...a"`, `needle = "aaa...b"`: each of the `n` start positions re-scans almost all of `needle`. For most real inputs it is fine, but Amazon will probe the worst case.

### KMP — O(n+m)

```rust
struct Solution;

impl Solution {
    fn str_str(haystack: String, needle: String) -> i32 {
        if needle.is_empty() { return 0; }
        let lps = Self::build_lps(&needle);
        let h: Vec<u8> = haystack.bytes().collect();
        let nd: Vec<u8> = needle.bytes().collect();
        let (n, m) = (h.len(), nd.len());
        let (mut i, mut j) = (0usize, 0usize);
        while i < n {
            if h[i] == nd[j] {
                i += 1;
                j += 1;
                if j == m { return (i - j) as i32; }
            } else if j > 0 {
                j = lps[j - 1]; // reuse the matched border; i does NOT move back
            } else {
                i += 1;
            }
        }
        -1
    }

    fn build_lps(s: &str) -> Vec<usize> {
        let s: Vec<u8> = s.bytes().collect();
        let n = s.len();
        let mut lps = vec![0usize; n];
        let (mut len, mut i) = (0usize, 1usize);
        while i < n {
            if s[i] == s[len] {
                len += 1;
                lps[i] = len;
                i += 1;
            } else if len > 0 {
                len = lps[len - 1];
            } else {
                lps[i] = 0;
                i += 1;
            }
        }
        lps
    }
}
```

**When does KMP matter?** Only when (a) library calls are forbidden, (b) the interviewer asks for guaranteed worst-case linear, or (c) you will search the *same* needle in many haystacks (precompute `lps` once). Otherwise `s.find(needle)` is the honest answer — say so, then offer KMP. See the full LPS derivation in [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md).

---

## 2. Longest Palindromic Substring (LC 5) — expand-around-center

> Return the longest substring of `s` that is a palindrome.

Expand-around-center is the **interview-preferred** solution: it is short, obviously correct, and O(1) space. Every palindrome has a center — a single character (odd length) or a gap between two characters (even length) — giving `2n-1` centers. Expand outward from each.

```rust
struct Solution;

impl Solution {
    fn longest_palindrome(s: String) -> String {
        if s.is_empty() { return String::new(); }
        let bytes: Vec<u8> = s.bytes().collect();
        let (mut start, mut end) = (0usize, 0usize);
        for i in 0..bytes.len() {
            let odd = Self::expand(&bytes, i as i64, i as i64);       // odd-length center at i
            let even = Self::expand(&bytes, i as i64, i as i64 + 1);  // even-length center between i and i+1
            let len = odd.max(even);
            if len > end - start + 1 {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        s[start..=end].to_string()
    }

    // Returns the length of the palindrome that expands from (l, r).
    fn expand(s: &[u8], mut l: i64, mut r: i64) -> usize {
        while l >= 0 && r < s.len() as i64 && s[l as usize] == s[r as usize] {
            l -= 1;
            r += 1;
        }
        (r - l - 1) as usize // characters strictly between the failed bounds
    }
}
```

**Complexity:** O(n²) time, O(1) space.

### When to mention Manacher

Bring up **Manacher's algorithm** only when the interviewer explicitly asks for **O(n)**. It transforms the string with separators (`^#a#b#a#$`) so every palindrome is odd-length, then reuses a mirror to avoid recomputing. It is correct but error-prone under interview pressure — state that you *know* it exists and could implement it, but that expand-around-center is the pragmatic choice unless O(n) is required. Full treatment in [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md).

### Edge cases
- Empty string → `""`.
- Single character → itself.
- All identical characters `"aaaa"` → the whole string (expansion never short-circuits early).

---

## 3. Find All Anagrams in a String (LC 438) — sliding window frequency

> Return the start indices of every substring of `s` that is an anagram of `p`.

This is a **fixed-size sliding window** of length `p.len()` over a 26-letter frequency array. Maintain a running count of how many letters currently have the correct frequency, so each shift is O(1).

```rust
struct Solution;

impl Solution {
    fn find_anagrams(s: String, p: String) -> Vec<i32> {
        let mut res = Vec::new();
        let s: Vec<u8> = s.bytes().collect();
        let p: Vec<u8> = p.bytes().collect();
        let (n, m) = (s.len(), p.len());
        if n < m { return res; }

        let mut need = vec![0i32; 26];
        let mut win = vec![0i32; 26];
        for i in 0..m {
            need[(p[i] - b'a') as usize] += 1;
            win[(s[i] - b'a') as usize] += 1;
        }
        if need == win { res.push(0); }

        for i in m..n {
            win[(s[i] - b'a') as usize] += 1;         // add the entering char
            win[(s[i - m] - b'a') as usize] -= 1;     // remove the leaving char
            if need == win { res.push((i - m + 1) as i32); }
        }
        res
    }
}
```

**Complexity:** O(n · 26) = O(n) time, O(26) = O(1) space. The `Vec` `==` comparison over 26 entries is constant; for a stricter O(1)-per-step, track a `matches` counter that increments/decrements as individual letter counts hit/leave equality. Window basis: [Fixed Size Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md).

### Edge cases
- `p` longer than `s` → empty list (the `n < m` guard).
- Repeated answers overlap (`s = "abab", p = "ab"` → `[0,1,2]`); the window naturally finds all.

---

## Amazon Leadership Principle Alignment

| Problem | LP demonstrated | How |
|---------|-----------------|-----|
| strStr (naive → KMP) | **Dive Deep** | Explaining *why* KMP avoids re-scanning, not just coding it |
| Longest Palindromic Substring | **Invent and Simplify** | Choosing expand-around-center over Manacher unless O(n) is required |
| Find All Anagrams | **Insist on the Highest Standards** | O(1)-per-step window instead of recounting each window |
| All three | **Bias for Action** | Stating the library/pragmatic answer first, then the algorithmic one |

---

## Related

- OA frequency: [OA-Qns → Amazon](../OA-Qns/Amazon.md)
- Other deep dives: [Google](Google.md) · [Microsoft](Microsoft.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
