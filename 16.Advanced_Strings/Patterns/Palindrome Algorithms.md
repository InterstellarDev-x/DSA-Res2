> **Topic:** [Advanced Strings](../README.md) · **Pattern 3 of 3**

# Palindrome Algorithms

Palindrome problems split into three tiers of technique:

1. **Expand around center** — O(n²), the interview default; simple and almost always accepted.
2. **Manacher's algorithm** — O(n), the optimal longest-palindromic-substring solver; mention/derive it only when O(n) is demanded.
3. **KMP on `s + reverse(s)`** — turns the *shortest palindrome by prepending* problem into a prefix-function computation.

---

## 1. Expand Around Center

Every palindrome has a center. A string of length `n` has **2n − 1** centers: `n` single-character centers (odd-length palindromes) and `n − 1` between-character centers (even-length palindromes). Expanding outward from each center while characters match finds the longest palindrome with that center.

### Problem 5 — Longest Palindromic Substring (LC 5, Medium)

```rust
struct Solution;

impl Solution {
    pub fn longest_palindrome(s: String) -> String {
        let chars: Vec<char> = s.chars().collect();
        if chars.is_empty() { return String::new(); }
        let n = chars.len() as i32;
        let mut start = 0i32;
        let mut end = 0i32;
        for i in 0..n {
            let odd  = Self::expand(&chars, i, i);         // odd-length, center at i
            let even = Self::expand(&chars, i, i + 1);     // even-length, center between i, i+1
            let len = odd.max(even);
            if len > end - start + 1 {
                start = i - (len - 1) / 2;     // recover left boundary from length
                end = i + len / 2;
            }
        }
        chars[start as usize..=end as usize].iter().collect()
    }

    // Returns the length of the palindrome expanding from (left, right).
    fn expand(s: &[char], mut left: i32, mut right: i32) -> i32 {
        let n = s.len() as i32;
        while left >= 0 && right < n && s[left as usize] == s[right as usize] {
            left -= 1;
            right += 1;
        }
        right - left - 1   // chars between the over-stepped bounds
    }
}
```

**Complexity:** O(n²) time, O(1) space. The two `expand` calls (odd + even) per index are exactly the 2n−1 centers. The boundary recovery `start = i - (len-1)/2` works for both parities — memorize it.

### Problem 6 — Palindromic Substrings, count (LC 647, Medium)

> Count how many substrings are palindromes (different positions count separately).

Each successful single-character extension during expansion is one more palindrome, so count expansions instead of tracking the longest.

```rust
struct Solution;

impl Solution {
    pub fn count_substrings(s: String) -> i32 {
        let chars: Vec<char> = s.chars().collect();
        let n = chars.len() as i32;
        let mut count = 0;
        for i in 0..n {
            count += Self::count_from(&chars, i, i);       // odd centers
            count += Self::count_from(&chars, i, i + 1);   // even centers
        }
        count
    }

    fn count_from(s: &[char], mut left: i32, mut right: i32) -> i32 {
        let n = s.len() as i32;
        let mut count = 0;
        while left >= 0 && right < n && s[left as usize] == s[right as usize] {
            count += 1;
            left -= 1;
            right += 1;
        }
        count
    }
}
```

**Complexity:** O(n²) time, O(1) space.

> **DP alternative.** Both problems also have a classic O(n²) time / O(n²) space DP: `dp[i][j] = (s[i]==s[j]) && (j-i<2 || dp[i+1][j-1])`. Expand-around-center is preferred because it is O(1) space and just as fast. See [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) for the tabulation form.

---

## 2. Manacher's Algorithm — O(n) Longest Palindrome

Manacher removes the odd/even split by **transforming** the string: insert a separator (`'#'`) between every character and at both ends. `"aba"` → `"^#a#b#a#$"` (sentinels `^`, `$` avoid bounds checks). Now *every* palindrome in the transformed string has odd length and a single center, and its radius in the transformed string equals the palindrome length in the original.

We keep a `center C` and `right boundary R` of the rightmost palindrome found, plus an array `p[i]` = radius at `i`. The **mirror trick**: for `i < R`, its mirror is `mirror = 2*C - i`, and `p[i]` can be seeded from `p[mirror]`, capped by `R - i`.

```rust
struct Solution;

impl Solution {
    pub fn longest_palindrome(s: String) -> String {
        if s.is_empty() { return String::new(); }
        let chars: Vec<char> = s.chars().collect();
        // Build transformed string: ^ # a # b # ... $
        let mut t: Vec<char> = vec!['^'];
        for c in &chars {
            t.push('#');
            t.push(*c);
        }
        t.push('#');
        t.push('$');
        let n = t.len();
        let mut p = vec![0i32; n];     // p[i] = palindrome radius centered at i
        let mut c = 0i32;
        let mut r = 0i32;              // current center and right boundary
        for i in 1..n - 1 {
            let i = i as i32;
            let mirror = 2 * c - i;
            if i < r { p[i as usize] = (r - i).min(p[mirror as usize]); }   // mirror seed, capped
            // expand around i past the seeded radius
            while t[(i + p[i as usize] + 1) as usize] == t[(i - p[i as usize] - 1) as usize] {
                p[i as usize] += 1;
            }
            if i + p[i as usize] > r { c = i; r = i + p[i as usize]; }       // slide the window
        }
        // find max radius and map back to original indices
        let mut max_len = 0i32;
        let mut center_index = 0i32;
        for i in 1..n - 1 {
            if p[i] > max_len { max_len = p[i]; center_index = i as i32; }
        }
        let start = ((center_index - max_len) / 2) as usize;   // convert transformed -> original
        chars[start..start + max_len as usize].iter().collect()
    }
}
```

**Complexity:** O(n) time, O(n) space. The sentinels `^`/`$` mean the inner `while` never reads out of bounds (they never match `#` or a real char). The mapping `start = (centerIndex - maxLen) / 2` undoes the `#`-insertion. **Interview note:** code this only if asked for linear time — it is fiddly and easy to get wrong under pressure; expand-around-center is the safe default.

---

## 3. Problem 3 — Shortest Palindrome (LC 214, Hard)

> By prepending characters in front of `s`, form the shortest palindrome and return it.

**Insight.** We want the **longest palindromic prefix** of `s`. The characters after that prefix must be reversed and prepended. Finding the longest palindromic prefix reduces to a prefix-function (KMP) computation:

> Build `combined = s + '#' + reverse(s)`. Compute its LPS array. `lps[last]` = length of the longest prefix of `s` that equals a suffix of `reverse(s)` — which is precisely the **longest palindromic prefix** of `s`.

The `'#'` separator (a char not in `s`) is essential: without it the border could spill from `s` into `reverse(s)` and overcount, producing a wrong (and possibly non-palindromic) answer.

```rust
struct Solution;

impl Solution {
    pub fn shortest_palindrome(s: String) -> String {
        if s.is_empty() { return s; }
        let rev: String = s.chars().rev().collect();
        let combined = s.clone() + "#" + &rev;
        let lps = Self::build_lps(&combined);
        let pal_prefix_len = lps[combined.len() - 1];   // longest palindromic prefix
        // characters of s after the palindromic prefix, reversed, go in front
        let suffix: String = s[pal_prefix_len..].chars().rev().collect();
        suffix + &s
    }

    fn build_lps(t: &str) -> Vec<usize> {
        let t: Vec<char> = t.chars().collect();
        let n = t.len();
        let mut lps = vec![0usize; n];
        let mut len = 0usize;
        let mut i = 1usize;
        while i < n {
            if t[i] == t[len] {
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

**Complexity:** O(n) time, O(n) space. Example: `s = "aacecaaa"`, `rev = "aaacecaa"`, `combined = "aacecaaa#aaacecaa"`. The LPS ends at 7 (`"aacecaa"` is the longest palindromic prefix), leaving the trailing `"a"`; prepend `reverse("a") = "a"` → `"aaacecaaa"`.

---

## 4. Dry-run — Expand Around Center on `"babad"`

Indices: `b(0) a(1) b(2) a(3) d(4)`.

| Center | Type | Expansion | Palindrome | Length |
|--------|------|-----------|------------|--------|
| i=0 | odd | `b` only (b vs nothing left) | `b` | 1 |
| i=0/1 | even | `b`≠`a` | — | 0 |
| i=1 | odd | `a`; then s[0]=b == s[2]=b → `bab`; then out of... s[-1] | `bab` | 3 |
| i=2 | odd | `b`; then s[1]=a == s[3]=a → `aba`; then s[0]=b ≠ s[4]=d | `aba` | 3 |
| i=3 | odd | `a`; then s[2]=b ≠ s[4]=d | `a` | 1 |
| i=4 | odd | `d` | `d` | 1 |

First maximal length 3 is found at center 1 (`"bab"`); center 2 ties with `"aba"`. We keep the first, so the answer is `"bab"` (either is accepted by LC 5). The even centers all fail immediately here because no two adjacent characters are equal.

---

## Recognition Signals

| Signal in the prompt | Reach for |
|----------------------|-----------|
| "longest palindromic substring" | expand around center (Manacher if O(n) required) |
| "count palindromic substrings" | expand around center, sum extensions |
| "shortest palindrome by prepending / appending" | KMP/LPS on `s + '#' + reverse(s)` |
| "is it a palindrome", basic check | two-pointer compare (no advanced tool needed) |
| explicitly "in linear time" for longest palindrome | Manacher's algorithm |
| "palindrome partitioning / min cuts" | DP — see [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) |

---

## Cross-links

- [String Matching (KMP)](String%20Matching%20(KMP).md) — Shortest Palindrome is built directly on the prefix function.
- [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) — the DP tabulation alternative for palindromic substrings and partitioning.

> **Last Updated:** 2026-06-26
