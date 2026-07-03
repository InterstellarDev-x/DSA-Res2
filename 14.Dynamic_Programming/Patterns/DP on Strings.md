> **Topic:** [Dynamic Programming](../README.md) · **Pattern 8 of 8**

# DP on Strings

String DP problems revolve around comparing, matching, partitioning, or transforming one or more strings. The defining trait is a **state indexed by positions** within the string(s) — typically `dp[i][j]` describing the relationship between a prefix of length `i` of one string and a prefix of length `j` of another, or `dp[i]` describing a property of the first `i` characters of a single string. From these states we derive transitions by inspecting one character at a time (`s[i - 1]`), branching on equality, special pattern symbols (`?`, `*`, `.`), or dictionary membership.

This pattern is the natural extension of [Longest Common Subsequence](./Longest%20Common%20Subsequence.md): once you internalize the `i`/`j` prefix-state formulation, edit distance, wildcard matching, regex matching, and interleaving all become variations on the same skeleton. Single-string problems (word break, decode ways, palindromic substrings) reuse the prefix idea with a 1D state or a 2D substring table `dp[i][j]` over the span `[i, j]`.

This document covers eight canonical problems (48–55). Each section gives the **state definition**, **recurrence**, **base cases**, a **complexity table**, full compilable Rust (recursion → memoization → tabulation → space optimization where relevant), and at least one worked dry-run.

---

## 1. Edit Distance (LC 72)

Given two strings `word1` and `word2`, return the minimum number of operations — **insert**, **delete**, or **replace** a single character — to convert `word1` into `word2`.

### State

`dp[i][j]` = minimum edits to convert the first `i` characters of `word1` into the first `j` characters of `word2`.

### Recurrence

Let `m = word1.len()`, `n = word2.len()`. Consider the last characters `word1[i - 1]` and `word2[j - 1]`:

- If they match: `dp[i][j] = dp[i - 1][j - 1]` (no operation needed).
- Otherwise take the best of three operations:
  - **Replace:** `dp[i - 1][j - 1] + 1`
  - **Delete** from `word1`: `dp[i - 1][j] + 1`
  - **Insert** into `word1`: `dp[i][j - 1] + 1`

### Base Cases

- `dp[0][j] = j` — empty `word1` needs `j` insertions.
- `dp[i][0] = i` — emptying `word1` needs `i` deletions.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Recursion (naive) | O(3^(m+n)) | O(m + n) stack |
| Memoization | O(m · n) | O(m · n) + O(m + n) stack |
| Tabulation | O(m · n) | O(m · n) |
| 1D space optimized | O(m · n) | O(n) |

### Rust — Recursion

```rust
struct EditDistanceRecursion;

impl EditDistanceRecursion {
    fn min_distance(word1: String, word2: String) -> i32 {
        let w1: Vec<char> = word1.chars().collect();
        let w2: Vec<char> = word2.chars().collect();
        Self::solve(&w1, &w2, w1.len(), w2.len())
    }

    fn solve(w1: &[char], w2: &[char], i: usize, j: usize) -> i32 {
        if i == 0 { return j as i32; } // insert remaining j chars
        if j == 0 { return i as i32; } // delete remaining i chars

        if w1[i - 1] == w2[j - 1] {
            return Self::solve(w1, w2, i - 1, j - 1);
        }
        let replace = Self::solve(w1, w2, i - 1, j - 1);
        let del = Self::solve(w1, w2, i - 1, j);
        let ins = Self::solve(w1, w2, i, j - 1);
        1 + replace.min(del).min(ins)
    }
}
```

### Rust — Memoization

```rust
struct EditDistanceMemo;

impl EditDistanceMemo {
    fn min_distance(word1: String, word2: String) -> i32 {
        let w1: Vec<char> = word1.chars().collect();
        let w2: Vec<char> = word2.chars().collect();
        let m = w1.len();
        let n = w2.len();
        let mut dp = vec![vec![-1i32; n + 1]; m + 1];
        Self::solve(&w1, &w2, m, n, &mut dp)
    }

    fn solve(w1: &[char], w2: &[char], i: usize, j: usize, dp: &mut Vec<Vec<i32>>) -> i32 {
        if i == 0 { return j as i32; }
        if j == 0 { return i as i32; }
        if dp[i][j] != -1 { return dp[i][j]; }

        let result = if w1[i - 1] == w2[j - 1] {
            Self::solve(w1, w2, i - 1, j - 1, dp)
        } else {
            let replace = Self::solve(w1, w2, i - 1, j - 1, dp);
            let del = Self::solve(w1, w2, i - 1, j, dp);
            let ins = Self::solve(w1, w2, i, j - 1, dp);
            1 + replace.min(del).min(ins)
        };
        dp[i][j] = result;
        result
    }
}
```

### Rust — Tabulation

```rust
struct EditDistanceTabulation;

impl EditDistanceTabulation {
    fn min_distance(word1: String, word2: String) -> i32 {
        let w1: Vec<char> = word1.chars().collect();
        let w2: Vec<char> = word2.chars().collect();
        let m = w1.len();
        let n = w2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];

        for j in 0..=n { dp[0][j] = j as i32; } // base row
        for i in 0..=m { dp[i][0] = i as i32; } // base column

        for i in 1..=m {
            for j in 1..=n {
                if w1[i - 1] == w2[j - 1] {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    let replace = dp[i - 1][j - 1];
                    let del = dp[i - 1][j];
                    let ins = dp[i][j - 1];
                    dp[i][j] = 1 + replace.min(del).min(ins);
                }
            }
        }
        dp[m][n]
    }
}
```

### Rust — 1D Space Optimized

```rust
struct EditDistanceSpaceOptimized;

impl EditDistanceSpaceOptimized {
    fn min_distance(word1: String, word2: String) -> i32 {
        let w1: Vec<char> = word1.chars().collect();
        let w2: Vec<char> = word2.chars().collect();
        let m = w1.len();
        let n = w2.len();
        let mut prev: Vec<i32> = (0..=(n as i32)).collect(); // dp[0][j]
        let mut curr = vec![0i32; n + 1];

        for i in 1..=m {
            curr[0] = i as i32; // dp[i][0]
            for j in 1..=n {
                if w1[i - 1] == w2[j - 1] {
                    curr[j] = prev[j - 1];
                } else {
                    curr[j] = 1 + prev[j - 1].min(prev[j]).min(curr[j - 1]);
                }
            }
            std::mem::swap(&mut prev, &mut curr); // reuse the array
        }
        prev[n]
    }
}
```

### Dry Run

`word1 = "horse"`, `word2 = "ros"`. The filled table (`dp[i][j]`, rows = prefixes of `"horse"`, columns = prefixes of `"ros"`):

|       | "" | r | o | s |
|-------|----|---|---|---|
| **""**| 0  | 1 | 2 | 3 |
| **h** | 1  | 1 | 2 | 3 |
| **o** | 2  | 2 | 1 | 2 |
| **r** | 3  | 2 | 2 | 2 |
| **s** | 4  | 3 | 3 | 2 |
| **e** | 5  | 4 | 4 | 3 |

Answer = `dp[5][3] = 3`. The path: replace `h→r`, replace `o→o` (free), delete `r`, keep `s`, delete `e` — three operations (`horse → rorse → rose → ros`).

---

## 2. Wildcard Matching (LC 44)

Given an input string `s` and a pattern `p` containing `?` and `*`, return whether `p` matches the **entire** string `s`:

- `?` matches any single character.
- `*` matches any sequence of characters (including the empty sequence).

### State

`dp[i][j]` = whether the first `i` characters of `s` match the first `j` characters of `p`.

### Recurrence

Inspect `p[j - 1]`:

- If it is a normal char or `?`: it must consume one char of `s` —
  `dp[i][j] = dp[i - 1][j - 1] && (p[j-1] == '?' || p[j-1] == s[i-1])`.
- If it is `*`: it matches **empty** (`dp[i][j - 1]`) **or** absorbs one more char of `s` (`dp[i - 1][j]`):
  `dp[i][j] = dp[i][j - 1] || dp[i - 1][j]`.

### Base Cases

- `dp[0][0] = true` — empty pattern matches empty string.
- `dp[i][0] = false` for `i > 0` — non-empty string cannot match empty pattern.
- **Leading `*`s:** `dp[0][j] = dp[0][j - 1] && p[j - 1] == '*'`. An empty string matches a prefix of pattern only if that prefix is all `*`s (each matching the empty sequence).

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(m · n) | O(m · n) |

(where `m = s.len()`, `n = p.len()`)

### Rust — Tabulation

```rust
struct WildcardMatching;

impl WildcardMatching {
    fn is_match(s: String, p: String) -> bool {
        let s: Vec<char> = s.chars().collect();
        let p: Vec<char> = p.chars().collect();
        let m = s.len();
        let n = p.len();
        let mut dp = vec![vec![false; n + 1]; m + 1];

        dp[0][0] = true; // empty matches empty
        for j in 1..=n {
            // leading '*' can match empty prefix of s
            dp[0][j] = dp[0][j - 1] && p[j - 1] == '*';
        }

        for i in 1..=m {
            for j in 1..=n {
                let pc = p[j - 1];
                if pc == '*' {
                    // '*' = empty (dp[i][j-1]) OR absorb s[i-1] (dp[i-1][j])
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
                } else if pc == '?' || pc == s[i - 1] {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = false;
                }
            }
        }
        dp[m][n]
    }
}
```

### Dry Run

`s = "adceb"`, `p = "*a*b"`. The leading `*` makes `dp[0][1] = true`. Walking the table, `dp[5][4]` resolves to `true`: the first `*` matches `""`, `a` matches `a`, the second `*` absorbs `dce`, and `b` matches `b`.

---

## 3. Regular Expression Matching (LC 10)

Given an input string `s` and a pattern `p` with `.` and `*`, return whether `p` matches the **entire** string `s`:

- `.` matches any single character.
- `*` matches **zero or more** of the **preceding** element.

The key difference from wildcard `*`: here `*` modifies the character immediately before it, rather than standing alone.

### State

`dp[i][j]` = whether the first `i` characters of `s` match the first `j` characters of `p`.

### Recurrence

Inspect `p[j - 1]`:

- Normal char or `.`: `dp[i][j] = dp[i - 1][j - 1] && matches(s[i-1], p[j-1])` where `matches` is true when the pattern char is `.` or equals the string char.
- `*` (with preceding char `p[j - 2]`):
  - **Zero occurrences:** drop the pair `x*` → `dp[i][j - 2]`.
  - **One or more occurrences:** if the preceding pattern char matches `s[i - 1]`, consume that char → `dp[i - 1][j]`.
  - Combine: `dp[i][j] = dp[i][j - 2] || (matches(s[i-1], p[j-2]) && dp[i - 1][j])`.

### Base Cases

- `dp[0][0] = true`.
- `dp[i][0] = false` for `i > 0`.
- **Patterns like `a*`, `a*b*`** can match the empty string: for `j >= 2`, if `p[j - 1] == '*'` then `dp[0][j] = dp[0][j - 2]`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Memoization | O(m · n) | O(m · n) + stack |
| Tabulation | O(m · n) | O(m · n) |

### Rust — Memoization

```rust
struct RegexMatchingMemo;

impl RegexMatchingMemo {
    fn is_match(s: String, p: String) -> bool {
        let s_chars: Vec<char> = s.chars().collect();
        let p_chars: Vec<char> = p.chars().collect();
        let m = s_chars.len();
        let n = p_chars.len();
        // 0 = unknown, 1 = true, -1 = false (since bool cannot hold "unset")
        let mut memo = vec![vec![0i32; n + 1]; m + 1];
        Self::solve(&s_chars, &p_chars, 0, 0, &mut memo)
    }

    fn solve(s: &[char], p: &[char], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> bool {
        if j == p.len() { return i == s.len(); }
        if memo[i][j] != 0 { return memo[i][j] == 1; }

        let first_match = i < s.len() && (p[j] == '.' || p[j] == s[i]);

        let result = if j + 1 < p.len() && p[j + 1] == '*' {
            // zero occurrences (skip "x*") OR one+ (consume s[i])
            let zero_occ = Self::solve(s, p, i, j + 2, memo);
            let one_plus = first_match && Self::solve(s, p, i + 1, j, memo);
            zero_occ || one_plus
        } else {
            first_match && Self::solve(s, p, i + 1, j + 1, memo)
        };

        memo[i][j] = if result { 1 } else { -1 };
        result
    }
}
```

### Rust — Tabulation

```rust
struct RegexMatchingTabulation;

impl RegexMatchingTabulation {
    fn is_match(s: String, p: String) -> bool {
        let s: Vec<char> = s.chars().collect();
        let p: Vec<char> = p.chars().collect();
        let m = s.len();
        let n = p.len();
        let mut dp = vec![vec![false; n + 1]; m + 1];

        dp[0][0] = true;
        // patterns like a*, a*b* may match empty string
        for j in 2..=n {
            if p[j - 1] == '*' {
                dp[0][j] = dp[0][j - 2];
            }
        }

        for i in 1..=m {
            for j in 1..=n {
                let pc = p[j - 1];
                if pc == '*' {
                    let prev = p[j - 2];
                    // zero occurrences of the preceding element
                    dp[i][j] = dp[i][j - 2];
                    // one or more occurrences
                    if prev == '.' || prev == s[i - 1] {
                        dp[i][j] = dp[i][j] || dp[i - 1][j];
                    }
                } else if pc == '.' || pc == s[i - 1] {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = false;
                }
            }
        }
        dp[m][n]
    }
}
```

### Dry Run

`s = "aab"`, `p = "c*a*b"`. `c*` matches zero `c`s (`dp[0][2] = true`), `a*` matches `aa`, `b` matches `b`. The chain of true cells leads to `dp[3][5] = true`.

---

## 4. Word Break (LC 139)

Given a string `s` and a dictionary `wordDict`, return whether `s` can be segmented into a space-separated sequence of one or more dictionary words.

### State

`dp[i]` = whether the prefix `&s[0..i]` (length `i`) can be segmented entirely using dictionary words.

### Recurrence

For each end position `i`, try every split point `j < i`:

`dp[i] = OR over j in [0, i) of ( dp[j] && dictionary.contains(&s[j..i]) )`.

In words: the prefix of length `i` is breakable if some earlier prefix of length `j` is breakable AND the remaining segment `s[j..i]` is a dictionary word.

### Base Case

- `dp[0] = true` — the empty prefix is trivially segmentable.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation (HashSet lookup) | O(n^2 · L) | O(n + dict) |

`n = s.len()`, `L` = average word length (cost of substring + hashing).

### Rust — Tabulation

```rust
use std::collections::HashSet;

struct WordBreak;

impl WordBreak {
    fn word_break(s: String, word_dict: Vec<String>) -> bool {
        let dictionary: HashSet<String> = word_dict.into_iter().collect();
        let n = s.len();
        let mut dp = vec![false; n + 1];
        dp[0] = true; // empty prefix

        for i in 1..=n {
            for j in 0..i {
                if dp[j] && dictionary.contains(&s[j..i]) {
                    dp[i] = true;
                    break; // one valid split is enough
                }
            }
        }
        dp[n]
    }
}
```

### Dry Run

`s = "leetcode"`, `wordDict = ["leet", "code"]`.

- `dp[0] = true`.
- `dp[4]`: `dp[0] && "leet" in dict` → `true`.
- `dp[8]`: `dp[4] && "code" in dict` → `true`.

Answer = `dp[8] = true`.

---

## 5. Decode Ways (LC 91)

A message of digits is encoded with `A → 1`, ..., `Z → 26`. Given a digit string `s`, return the number of ways to decode it. A leading `'0'` decodes to nothing valid, so it must be handled carefully.

### State

`dp[i]` = number of ways to decode the prefix `&s[0..i]` (length `i`).

### Recurrence

For position `i` (1-indexed over characters):

- **One-digit decode:** if `s[i - 1]` is `'1'..'9'`, add `dp[i - 1]`.
- **Two-digit decode:** if the substring `s[i-2..i)` (i.e. chars at `i-2` and `i-1`) forms a value in `[10, 26]`, add `dp[i - 2]`.

`dp[i] = (validOne ? dp[i-1] : 0) + (validTwo ? dp[i-2] : 0)`.

A character `'0'` can never decode as a single digit and is only valid as the second digit of `10` or `20`.

### Base Cases

- `dp[0] = 1` — empty string has one (empty) decoding.
- `dp[1] = if s[0] == b'0' { 0 } else { 1 }`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(n) | O(n) |
| O(1) space variant | O(n) | O(1) |

### Rust — Tabulation

```rust
struct DecodeWays;

impl DecodeWays {
    fn num_decodings(s: String) -> i32 {
        let s: Vec<u8> = s.bytes().collect();
        let n = s.len();
        if n == 0 || s[0] == b'0' { return 0; }

        let mut dp = vec![0i32; n + 1];
        dp[0] = 1;
        dp[1] = 1; // s[0] is guaranteed != '0' here

        for i in 2..=n {
            let one = s[i - 1];
            let tens = s[i - 2];

            if one != b'0' {                 // single-digit decode 1..9
                dp[i] += dp[i - 1];
            }
            let two_digit = (tens - b'0') as i32 * 10 + (one - b'0') as i32;
            if two_digit >= 10 && two_digit <= 26 { // two-digit decode 10..26
                dp[i] += dp[i - 2];
            }
        }
        dp[n]
    }
}
```

### Rust — O(1) Space

```rust
struct DecodeWaysSpaceOptimized;

impl DecodeWaysSpaceOptimized {
    fn num_decodings(s: String) -> i32 {
        let s: Vec<u8> = s.bytes().collect();
        let n = s.len();
        if n == 0 || s[0] == b'0' { return 0; }

        let mut prev2 = 1i32;        // dp[i-2]
        let mut prev1 = 1i32;        // dp[i-1], i.e. dp[1]

        for i in 2..=n {
            let mut curr = 0i32;
            let one = s[i - 1];
            let tens = s[i - 2];

            if one != b'0' {
                curr += prev1;
            }
            let two_digit = (tens - b'0') as i32 * 10 + (one - b'0') as i32;
            if two_digit >= 10 && two_digit <= 26 {
                curr += prev2;
            }
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Dry Run

`s = "226"`.

- `dp[0] = 1`, `dp[1] = 1` (`"2"`).
- `dp[2]`: `"2"` valid (+`dp[1]`=1) and `"22"` in [10,26] (+`dp[0]`=1) → `2`.
- `dp[3]`: `"6"` valid (+`dp[2]`=2) and `"26"` in [10,26] (+`dp[1]`=1) → `3`.

Answer = `dp[3] = 3` (`"BBF"`, `"BZ"`, `"VF"`).

---

## 6. Palindromic Substrings (LC 647, count)

Given a string `s`, return the **number** of palindromic substrings (substrings that read the same forwards and backwards; different positions count separately).

### Approach A — Expand Around Center

Every palindrome has a center: either a single character (odd length, `n` centers) or a gap between two characters (even length, `n - 1` centers). For each of the `2n - 1` centers, expand outward while the boundary characters match, counting one palindrome per successful expansion.

### Approach B — DP Boolean Table

`dp[i][j]` = whether `s[i..=j]` is a palindrome.

Recurrence: `dp[i][j] = (s[i] == s[j]) && (j - i < 2 || dp[i + 1][j - 1])`.

The condition `j - i < 2` covers length-1 (`i == j`) and length-2 (`j == i + 1`) substrings, which need no inner check. Iterate `i` from high to low so that `dp[i + 1][...]` is already computed.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Expand around center | O(n^2) | O(1) |
| DP boolean table | O(n^2) | O(n^2) |

### Rust — Expand Around Center

```rust
struct PalindromicSubstringsExpand;

impl PalindromicSubstringsExpand {
    fn count_substrings(s: String) -> i32 {
        let s: Vec<char> = s.chars().collect();
        let mut count = 0;
        for center in 0..s.len() {
            count += Self::expand(&s, center as i32, center as i32);     // odd length
            count += Self::expand(&s, center as i32, center as i32 + 1);  // even length
        }
        count
    }

    fn expand(s: &[char], mut left: i32, mut right: i32) -> i32 {
        let mut count = 0;
        while left >= 0 && right < s.len() as i32
                && s[left as usize] == s[right as usize] {
            count += 1;
            left -= 1;
            right += 1;
        }
        count
    }
}
```

### Rust — DP Boolean Table

```rust
struct PalindromicSubstringsDP;

impl PalindromicSubstringsDP {
    fn count_substrings(s: String) -> i32 {
        let s: Vec<char> = s.chars().collect();
        let n = s.len();
        let mut dp = vec![vec![false; n]; n];
        let mut count = 0;

        for i in (0..n).rev() {
            for j in i..n {
                if s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1]) {
                    dp[i][j] = true;
                    count += 1;
                }
            }
        }
        count
    }
}
```

### Dry Run

`s = "aaa"`. Palindromes: `"a"`(x3), `"aa"`(x2), `"aaa"`(x1) → total `6`. Expand-around-center: center 0 odd → 1; center 0/1 even → 1; center 1 odd → 2 (`"a"`, `"aaa"`); center 1/2 even → 1; center 2 odd → 1. Sum = `1+1+2+1+1 = 6`.

---

## 7. Longest Palindromic Substring (LC 5)

Given a string `s`, return the longest palindromic substring.

### Approach A — Expand Around Center (preferred: O(1) space)

Same `2n - 1` centers as above, but instead of counting, track the longest matched `[start, end]` window and return that substring at the end.

### Approach B — DP Table

`dp[i][j]` = whether `s[i..=j]` is a palindrome (same recurrence as Section 6). While filling, record the longest `(i, j)` span where `dp[i][j]` is true.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Expand around center | O(n^2) | O(1) |
| DP table | O(n^2) | O(n^2) |

### Rust — Expand Around Center

```rust
struct LongestPalindromeExpand {
    start: usize,
    max_len: usize,
}

impl LongestPalindromeExpand {
    fn new() -> Self {
        LongestPalindromeExpand { start: 0, max_len: 0 }
    }

    fn longest_palindrome(&mut self, s: String) -> String {
        let s_chars: Vec<char> = s.chars().collect();
        if s_chars.is_empty() { return String::new(); }
        for center in 0..s_chars.len() {
            self.expand(&s_chars, center as i32, center as i32);      // odd length
            self.expand(&s_chars, center as i32, center as i32 + 1);  // even length
        }
        s_chars[self.start..self.start + self.max_len].iter().collect()
    }

    fn expand(&mut self, s: &[char], mut left: i32, mut right: i32) {
        while left >= 0 && right < s.len() as i32
                && s[left as usize] == s[right as usize] {
            left -= 1;
            right += 1;
        }
        let len = (right - left - 1) as usize; // window after the failing step
        if len > self.max_len {
            self.max_len = len;
            self.start = (left + 1) as usize;
        }
    }
}
```

### Rust — DP Table

```rust
struct LongestPalindromeDP;

impl LongestPalindromeDP {
    fn longest_palindrome(s: String) -> String {
        let s_chars: Vec<char> = s.chars().collect();
        let n = s_chars.len();
        if n < 2 { return s; }

        let mut dp = vec![vec![false; n]; n];
        let mut best_start = 0usize;
        let mut best_len = 1usize;

        for i in (0..n).rev() {
            for j in i..n {
                if s_chars[i] == s_chars[j] && (j - i < 2 || dp[i + 1][j - 1]) {
                    dp[i][j] = true;
                    if j - i + 1 > best_len {
                        best_len = j - i + 1;
                        best_start = i;
                    }
                }
            }
        }
        s_chars[best_start..best_start + best_len].iter().collect()
    }
}
```

### Dry Run

`s = "babad"`. Expanding at center index 1 (`'a'`) odd gives `"bab"` (len 3). Center index 2 (`'b'`) odd gives `"aba"` (len 3). Either is a valid answer; the algorithm keeps the first longest found, `"bab"`.

---

## 8. Interleaving String (LC 97)

Given `s1`, `s2`, and `s3`, return whether `s3` is formed by an interleaving of `s1` and `s2` — i.e. `s3` can be split into pieces drawn alternately (in order) from `s1` and `s2`.

### Length Check First

An interleaving must use every character of both inputs, so `s1.len() + s2.len() == s3.len()` is a necessary precondition. If it fails, return `false` immediately.

### State

`dp[i][j]` = whether the first `i` characters of `s1` and the first `j` characters of `s2` can interleave to form the first `i + j` characters of `s3`.

### Recurrence

The character `s3[i + j - 1]` came from either `s1` or `s2`:

- From `s1`: `dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]`.
- From `s2`: `dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]`.

`dp[i][j] = (fromS1) || (fromS2)`.

### Base Cases

- `dp[0][0] = true`.
- `dp[i][0]`: only `s1`'s prefix used — `dp[i - 1][0] && s1[i - 1] == s3[i - 1]`.
- `dp[0][j]`: only `s2`'s prefix used — `dp[0][j - 1] && s2[j - 1] == s3[j - 1]`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(m · n) | O(m · n) |

### Rust — Tabulation

```rust
struct InterleavingString;

impl InterleavingString {
    fn is_interleave(s1: String, s2: String, s3: String) -> bool {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let s3: Vec<char> = s3.chars().collect();
        let m = s1.len();
        let n = s2.len();
        if m + n != s3.len() { return false; } // length precondition

        let mut dp = vec![vec![false; n + 1]; m + 1];
        dp[0][0] = true;

        for i in 1..=m { // first column: only s1
            dp[i][0] = dp[i - 1][0] && s1[i - 1] == s3[i - 1];
        }
        for j in 1..=n { // first row: only s2
            dp[0][j] = dp[0][j - 1] && s2[j - 1] == s3[j - 1];
        }

        for i in 1..=m {
            for j in 1..=n {
                let target = s3[i + j - 1];
                let from_s1 = dp[i - 1][j] && s1[i - 1] == target;
                let from_s2 = dp[i][j - 1] && s2[j - 1] == target;
                dp[i][j] = from_s1 || from_s2;
            }
        }
        dp[m][n]
    }
}
```

### Dry Run

`s1 = "aab"`, `s2 = "axy"`, `s3 = "aaxaby"`. Lengths: `3 + 3 = 6` ✓. Tracing the table, `dp[3][3]` evaluates to `true`: one valid interleaving is `a`(s1) `a`(s1) `x`(s2) `a`(s2... no) — the DP confirms a consistent path exists yielding `"aaxaby"`.

---

## Recognition Signals

| Signal in the problem statement | Likely problem / sub-pattern |
|---|---|
| "minimum operations to convert / transform one string to another", insert/delete/replace | Edit Distance (LC 72) |
| Pattern with `?` (one char) and `*` (any sequence) | Wildcard Matching (LC 44) |
| Pattern with `.` (one char) and `*` (zero-or-more of preceding) | Regular Expression Matching (LC 10) |
| "can the string be segmented into dictionary words" | Word Break (LC 139) |
| Digits encoded as letters, "count the number of ways to decode" | Decode Ways (LC 91) |
| "count substrings that are palindromes" | Palindromic Substrings (LC 647) |
| "return the longest palindromic substring" | Longest Palindromic Substring (LC 5) |
| "is s3 an interleaving of s1 and s2", merge two strings in order | Interleaving String (LC 97) |
| Two strings, compare prefixes, `dp[i][j]` over lengths | Two-string prefix DP (Edit/Wildcard/Regex/Interleave) |
| One string, property of prefix length `i` | 1D prefix DP (Word Break, Decode Ways) |
| One string, property of span `[i, j]` | 2D substring DP (palindrome problems) |

---

## Summary

| # | Problem | LC | State | Time | Space (opt.) |
|---|---|---|---|---|---|
| 48 | Edit Distance | 72 | `dp[i][j]` = min edits for prefixes | O(m·n) | O(n) |
| 49 | Wildcard Matching | 44 | `dp[i][j]` = prefixes match | O(m·n) | O(m·n) |
| 50 | Regular Expression Matching | 10 | `dp[i][j]` = prefixes match | O(m·n) | O(m·n) |
| 51 | Word Break | 139 | `dp[i]` = prefix segmentable | O(n²·L) | O(n) |
| 52 | Decode Ways | 91 | `dp[i]` = ways to decode prefix | O(n) | O(1) |
| 53 | Palindromic Substrings | 647 | center / `dp[i][j]` palindrome | O(n²) | O(1) |
| 54 | Longest Palindromic Substring | 5 | center / `dp[i][j]` palindrome | O(n²) | O(1) |
| 55 | Interleaving String | 97 | `dp[i][j]` = interleave prefixes | O(m·n) | O(m·n) |

**Key takeaways:**

- **Two strings → `dp[i][j]` over prefix lengths.** Edit Distance, Wildcard, Regex, and Interleaving all share this skeleton; the recurrence differs only in how transitions branch on the current characters and pattern symbols.
- **`*` semantics matter.** In wildcard matching `*` stands alone and matches any sequence; in regex `*` modifies the *preceding* element and means zero-or-more — leading to the `dp[i][j-2]` "skip the pair" transition.
- **Single string → `dp[i]` (prefix) or `dp[i][j]` (span).** Word Break and Decode Ways use prefix states; palindrome problems use either expand-around-center (O(1) space) or a substring boolean table filled bottom-up.
- **Space optimization.** Any `dp[i][j]` depending only on the previous row collapses to two 1D arrays (Edit Distance), and prefix recurrences depending on a fixed window collapse to a few scalars (Decode Ways).
- **Watch the edge cases:** leading `'0'` (Decode Ways), leading `*` (Wildcard/Regex base row), and the length precondition (Interleaving String).

> **Last Updated:** 2026-06-26
