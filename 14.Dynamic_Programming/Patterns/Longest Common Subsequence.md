> **Topic:** [Dynamic Programming](../README.md) · **Pattern 4 of 8**

# Longest Common Subsequence

The Longest Common Subsequence (LCS) family is one of the most reusable patterns in dynamic programming. Once you internalize the single 2D recurrence at its core, an entire constellation of string problems collapses into thin wrappers around it: longest common substring, palindromic subsequences, minimum edits to form a palindrome, conversion costs, shortest common supersequence, and distinct-subsequence counting. This document builds the core from first principles (recursion → memoization → tabulation → space optimization) and then shows, problem by problem, how nearly every variant reduces back to LCS.

A **subsequence** is any sequence derived by deleting zero or more characters without changing the relative order of the remaining characters. It does **not** have to be contiguous. `"ace"` is a subsequence of `"abcde"`. A **substring**, by contrast, must be contiguous.

Related reading: [DP on Strings](./DP%20on%20Strings.md) · [DP on Subsequences](./DP%20on%20Subsequences.md) · [Matrix Chain Multiplication](./Matrix%20Chain%20Multiplication.md)

---

## 1. The LCS Core

Given two strings `s1` (length `m`) and `s2` (length `n`), find the length of their longest common subsequence.

### State

Let `dp[i][j]` = the length of the LCS of the prefixes `s1[0..i-1]` and `s2[0..j-1]` (i.e. the first `i` characters of `s1` and the first `j` characters of `s2`). Using **1-indexed prefix lengths** rather than 0-indexed pointers makes the base case clean.

### Recurrence

```
if s1[i-1] == s2[j-1]:
    dp[i][j] = 1 + dp[i-1][j-1]      // characters match: extend the diagonal
else:
    dp[i][j] = max(dp[i-1][j], dp[i][j-1])   // skip one character from either string
```

### Base case

`dp[0][j] = dp[i][0] = 0` — an empty prefix shares nothing with anything.

### Answer

`dp[m][n]`.

> **Why it works:** Consider the last characters. If they are equal, they can always be paired up as the last element of *some* optimal LCS, so we add 1 and recurse on both shrunken prefixes. If they differ, at least one of them cannot be the final matched character, so we try dropping each and take the better outcome.

---

## 2. Step 1 — Recursion

The recursion mirrors the recurrence exactly. We move two pointers from the **ends** of both strings toward the front.

```rust
struct LCSRecursion;

impl LCSRecursion {
    fn longest_common_subsequence(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        Self::solve(&s1, &s2, s1.len(), s2.len())
    }

    // f(i, j) = LCS length of s1[0..i-1] and s2[0..j-1]
    fn solve(s1: &[char], s2: &[char], i: usize, j: usize) -> i32 {
        if i == 0 || j == 0 {
            return 0; // empty prefix
        }
        if s1[i - 1] == s2[j - 1] {
            return 1 + Self::solve(s1, s2, i - 1, j - 1);
        }
        Self::solve(s1, s2, i - 1, j).max(Self::solve(s1, s2, i, j - 1))
    }
}
```

**Time:** `O(2^(m+n))` in the worst case — exponential due to overlapping subproblems. **Space:** `O(m + n)` recursion depth.

---

## 3. Step 2 — Memoization (Top-Down)

There are only `(m + 1) * (n + 1)` distinct states, so we cache them. We initialize the memo with `-1` (constructor initializes all cells to `-1`) to distinguish "uncomputed" from a legitimate `0`.

```rust
struct LCSMemo;

impl LCSMemo {
    fn longest_common_subsequence(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![-1i32; n + 1]; m + 1];
        Self::solve(&s1, &s2, m, n, &mut dp)
    }

    fn solve(s1: &[char], s2: &[char], i: usize, j: usize, dp: &mut Vec<Vec<i32>>) -> i32 {
        if i == 0 || j == 0 {
            return 0;
        }
        if dp[i][j] != -1 {
            return dp[i][j];
        }
        if s1[i - 1] == s2[j - 1] {
            dp[i][j] = 1 + Self::solve(s1, s2, i - 1, j - 1, dp);
        } else {
            dp[i][j] = Self::solve(s1, s2, i - 1, j, dp)
                .max(Self::solve(s1, s2, i, j - 1, dp));
        }
        dp[i][j]
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)` table + `O(m + n)` recursion stack.

---

## 4. Step 3 — Tabulation (Bottom-Up)

Convert the recursion into iteration. Because `dp[i][j]` depends on `dp[i-1][j-1]`, `dp[i-1][j]`, and `dp[i][j-1]`, we fill the table in increasing order of `i` and `j`. Row `0` and column `0` are already `0` (zero-initialized by default in Rust).

```rust
struct LCSTabulation;

impl LCSTabulation {
    fn longest_common_subsequence(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];

        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }
        dp[m][n]
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`.

---

## 5. Step 4 — Space Optimization (Two Rows)

Each row depends only on the previous row, so we keep just two 1D arrays of size `n + 1`.

```rust
struct LCSSpaceOptimized;

impl LCSSpaceOptimized {
    fn longest_common_subsequence(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut prev = vec![0i32; n + 1];
        let mut curr = vec![0i32; n + 1];

        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    curr[j] = 1 + prev[j - 1];
                } else {
                    curr[j] = prev[j].max(curr[j - 1]);
                }
            }
            std::mem::swap(&mut prev, &mut curr); // reuse the array; values get overwritten next pass
        }
        prev[n]
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(n)`.

> Space optimization works for length-only queries. The moment you need to **reconstruct** the actual subsequence (Problems 24 and 29), you must keep the full 2D table.

---

## 6. Dry Run

`s1 = "abcde"`, `s2 = "ace"`. Expected LCS = `"ace"`, length `3`.

Rows are indexed by `s1` (`abcde`), columns by `s2` (`ace`). Cell `dp[i][j]` follows the recurrence.

|       | "" | a | c | e |
|-------|----|---|---|---|
| **""**| 0  | 0 | 0 | 0 |
| **a** | 0  | 1 | 1 | 1 |
| **b** | 0  | 1 | 1 | 1 |
| **c** | 0  | 1 | 2 | 2 |
| **d** | 0  | 1 | 2 | 2 |
| **e** | 0  | 1 | 2 | 3 |

Walkthrough of a few cells:
- `dp[1][1]`: `s1[0]='a'` == `s2[0]='a'` → `1 + dp[0][0] = 1`.
- `dp[3][2]`: `s1[2]='c'` == `s2[1]='c'` → `1 + dp[2][1] = 1 + 1 = 2`.
- `dp[3][3]`: `s1[2]='c'` != `s2[2]='e'` → `max(dp[2][3], dp[3][2]) = max(1, 2) = 2`.
- `dp[5][3]`: `s1[4]='e'` == `s2[2]='e'` → `1 + dp[4][2] = 1 + 2 = 3`.

The bottom-right cell `dp[5][3] = 3` is the answer.

---

## 7. Problem 23 — Longest Common Subsequence (LC 1143)

This **is** the core, covered in Sections 1–6. Use the space-optimized version in interviews unless reconstruction is required. The canonical signature on LeetCode:

```rust
struct Solution;

impl Solution {
    fn longest_common_subsequence(text1: String, text2: String) -> i32 {
        let t1: Vec<char> = text1.chars().collect();
        let t2: Vec<char> = text2.chars().collect();
        let m = t1.len();
        let n = t2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];

        for i in 1..=m {
            for j in 1..=n {
                if t1[i - 1] == t2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }
        dp[m][n]
    }
}
```

---

## 8. Problem 24 — Print LCS (GFG)

Beyond the length, reconstruct the actual subsequence string by **backtracking** the filled DP table from `dp[m][n]` to `dp[0][0]`.

**Backtracking rule:** start at `(m, n)`.
- If `s1[i-1] == s2[j-1]`, this character is part of the LCS — prepend it and move diagonally to `(i-1, j-1)`.
- Otherwise move toward the larger neighbor: if `dp[i-1][j] >= dp[i][j-1]` go up, else go left.

```rust
struct PrintLCS;

impl PrintLCS {
    fn print_lcs(s1: &str, s2: &str) -> String {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];

        // Build the standard LCS table.
        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }

        // Backtrack to reconstruct the subsequence.
        let mut result = Vec::new();
        let mut i = m;
        let mut j = n;
        while i > 0 && j > 0 {
            if s1[i - 1] == s2[j - 1] {
                result.push(s1[i - 1]); // characters are collected in reverse
                i -= 1;
                j -= 1;
            } else if dp[i - 1][j] >= dp[i][j - 1] {
                i -= 1;
            } else {
                j -= 1;
            }
        }
        result.reverse();
        result.into_iter().collect()
    }
}
```

**Time:** `O(m * n)` to build + `O(m + n)` to backtrack. **Space:** `O(m * n)` (full table needed for reconstruction).

---

## 9. Problem 25 — Longest Common Substring (GFG)

A **substring** is contiguous. The recurrence changes in two ways: on a mismatch we **reset to 0** (a substring cannot have gaps), and the answer is the **global maximum** across the whole table, not `dp[m][n]`.

### Recurrence

```
if s1[i-1] == s2[j-1]:
    dp[i][j] = 1 + dp[i-1][j-1]
else:
    dp[i][j] = 0          // contiguity broken
answer = max over all dp[i][j]
```

```rust
struct LongestCommonSubstring;

impl LongestCommonSubstring {
    fn longest_common_substr(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];
        let mut best = 0;

        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                    best = best.max(dp[i][j]);
                } else {
                    dp[i][j] = 0; // reset on mismatch
                }
            }
        }
        best
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`, reducible to `O(n)` with two rows.

---

## 10. Problem 26 — Longest Palindromic Subsequence (LC 516)

**Key reduction:** the longest palindromic subsequence of `s` is exactly the LCS of `s` and `reverse(s)`. A palindrome reads the same forwards and backwards, so the longest subsequence common to `s` and its reverse must itself be a palindrome.

```rust
struct Solution;

impl Solution {
    fn longest_palindrome_subseq(s: String) -> i32 {
        let rev: String = s.chars().rev().collect();
        Self::lcs(&s, &rev)
    }

    fn lcs(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];
        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }
        dp[m][n]
    }
}
```

**Time:** `O(n^2)`. **Space:** `O(n^2)`.

---

## 11. Problem 27 — Minimum Insertions to Make a String Palindrome (LC 1312)

**Key reduction:** the characters that already form the longest palindromic subsequence (LPS) can stay; every remaining character needs a mirror inserted. Therefore:

```
answer = n - LPS(s)
```

where `LPS(s) = LCS(s, reverse(s))` from Problem 26. The same value also equals the minimum number of *deletions* to make `s` a palindrome.

```rust
struct Solution;

impl Solution {
    fn min_insertions(s: String) -> i32 {
        let n = s.len() as i32;
        n - Self::longest_palindrome_subseq(&s)
    }

    fn longest_palindrome_subseq(s: &str) -> i32 {
        let rev: String = s.chars().rev().collect();
        let s: Vec<char> = s.chars().collect();
        let rev: Vec<char> = rev.chars().collect();
        let n = s.len();
        let mut dp = vec![vec![0i32; n + 1]; n + 1];
        for i in 1..=n {
            for j in 1..=n {
                if s[i - 1] == rev[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }
        dp[n][n]
    }
}
```

**Time:** `O(n^2)`. **Space:** `O(n^2)`.

---

## 12. Problem 28 — Minimum Insertions/Deletions to Convert A to B (GFG)

To turn `A` (length `m`) into `B` (length `n`), keep the LCS untouched, delete the rest of `A`, and insert the rest of `B`.

```
deletions  = m - LCS(A, B)
insertions = n - LCS(A, B)
answer     = deletions + insertions = m + n - 2 * LCS(A, B)
```

```rust
struct MinInsertDeleteAtoB;

impl MinInsertDeleteAtoB {
    fn min_operations(a: &str, b: &str) -> i32 {
        let m = a.len() as i32;
        let n = b.len() as i32;
        let lcs = Self::longest_common_subsequence(a, b);
        let deletions = m - lcs;
        let insertions = n - lcs;
        deletions + insertions // == m + n - 2 * lcs
    }

    fn longest_common_subsequence(s1: &str, s2: &str) -> i32 {
        let s1: Vec<char> = s1.chars().collect();
        let s2: Vec<char> = s2.chars().collect();
        let m = s1.len();
        let n = s2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];
        for i in 1..=m {
            for j in 1..=n {
                if s1[i - 1] == s2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }
        dp[m][n]
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`.

---

## 13. Problem 29 — Shortest Common Supersequence (LC 1092)

The shortest common supersequence (SCS) is the shortest string that has both `s1` and `s2` as subsequences. Its **length** is `m + n - LCS`, because the LCS characters are shared and counted once instead of twice.

To **reconstruct** the SCS, build the full LCS table and backtrack. When characters match, emit that single character and step diagonally. When they differ, emit the character from whichever direction the LCS table grew from, stepping in that direction. Finally, flush whatever remains of either string.

```rust
struct Solution;

impl Solution {
    fn shortest_common_supersequence(s1: String, s2: String) -> String {
        let sc1: Vec<char> = s1.chars().collect();
        let sc2: Vec<char> = s2.chars().collect();
        let m = sc1.len();
        let n = sc2.len();
        let mut dp = vec![vec![0i32; n + 1]; m + 1];

        for i in 1..=m {
            for j in 1..=n {
                if sc1[i - 1] == sc2[j - 1] {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = dp[i - 1][j].max(dp[i][j - 1]);
                }
            }
        }

        let mut result = Vec::new();
        let mut i = m;
        let mut j = n;
        while i > 0 && j > 0 {
            if sc1[i - 1] == sc2[j - 1] {
                result.push(sc1[i - 1]); // shared character, write once
                i -= 1;
                j -= 1;
            } else if dp[i - 1][j] >= dp[i][j - 1] {
                result.push(sc1[i - 1]); // came from s1
                i -= 1;
            } else {
                result.push(sc2[j - 1]); // came from s2
                j -= 1;
            }
        }
        // Flush leftover prefix of whichever string remains.
        while i > 0 {
            result.push(sc1[i - 1]);
            i -= 1;
        }
        while j > 0 {
            result.push(sc2[j - 1]);
            j -= 1;
        }
        result.reverse();
        result.into_iter().collect()
    }
}
```

The SCS **length** alone is simply `m + n - dp[m][n]`.

**Time:** `O(m * n)`. **Space:** `O(m * n)`.

---

## 14. Problem 30 — Distinct Subsequences (LC 115)

Count how many distinct subsequences of `s` equal `t`. This is a counting variant of LCS, so the recurrence sums rather than maximizes.

### State

`dp[i][j]` = number of distinct subsequences of `s[0..i-1]` that equal `t[0..j-1]`.

### Recurrence

```
if s[i-1] == t[j-1]:
    dp[i][j] = dp[i-1][j-1] + dp[i-1][j]   // use this s-char to match, OR skip it
else:
    dp[i][j] = dp[i-1][j]                  // can only skip this s-char
```

### Base case

`dp[i][0] = 1` for all `i` (the empty target is matched exactly one way — by deleting everything). `dp[0][j] = 0` for `j > 0`.

> **Overflow note:** counts can exceed `i32::MAX`. LeetCode guarantees the answer fits in a 32-bit signed int, but **intermediate** sums can overflow. Using `i64` for the table (or being careful) is the safe choice; below we use `Vec<Vec<i64>>` matching the LeetCode guarantee and note where `i64` would be required for unconstrained inputs.

```rust
struct Solution;

impl Solution {
    fn num_distinct(s: String, t: String) -> i32 {
        let s: Vec<char> = s.chars().collect();
        let t: Vec<char> = t.chars().collect();
        let m = s.len();
        let n = t.len();
        // Use i64 internally to avoid intermediate overflow for general inputs.
        let mut dp = vec![vec![0i64; n + 1]; m + 1];

        for i in 0..=m {
            dp[i][0] = 1; // empty t matched exactly one way
        }

        for i in 1..=m {
            for j in 1..=n {
                if s[i - 1] == t[j - 1] {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        dp[m][n] as i32
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`, reducible to `O(n)` (iterate `j` from high to low when using one row).

---

## 15. Complexity Summary

| # | Problem | Reduction | Time | Space |
|---|---------|-----------|------|-------|
| 23 | LCS (LC 1143) | the core | `O(m·n)` | `O(m·n)` → `O(n)` |
| 24 | Print LCS (GFG) | LCS + backtrack | `O(m·n)` | `O(m·n)` |
| 25 | Longest Common Substring (GFG) | LCS with mismatch reset | `O(m·n)` | `O(m·n)` → `O(n)` |
| 26 | Longest Palindromic Subseq (LC 516) | `LCS(s, reverse(s))` | `O(n²)` | `O(n²)` |
| 27 | Min Insertions for Palindrome (LC 1312) | `n - LPS(s)` | `O(n²)` | `O(n²)` |
| 28 | Min Insert/Delete A→B (GFG) | `m + n - 2·LCS` | `O(m·n)` | `O(m·n)` |
| 29 | Shortest Common Supersequence (LC 1092) | `m + n - LCS` + backtrack | `O(m·n)` | `O(m·n)` |
| 30 | Distinct Subsequences (LC 115) | counting LCS variant | `O(m·n)` | `O(m·n)` → `O(n)` |

---

## 16. Recognition Signals

| If the prompt says… | Reach for… | Mechanic |
|---------------------|-----------|----------|
| "common subsequence of two strings" | LCS core | match → `1 + diag`, else `max(up, left)` |
| "print / reconstruct the subsequence" | LCS + backtrack | keep full 2D table, walk from `(m,n)` |
| "common **substring** / contiguous" | LCS variant | reset to `0` on mismatch, track global max |
| "longest **palindromic** subsequence" | LPS | `LCS(s, reverse(s))` |
| "min insertions/deletions to make palindrome" | LPS wrapper | `n - LPS(s)` |
| "convert A to B with insert/delete" | LCS wrapper | `m + n - 2·LCS` |
| "shortest string containing both" | SCS | `m + n - LCS`; backtrack to build it |
| "count the number of ways / distinct subsequences" | counting LCS | sum branches instead of `max` |

---

## 17. Summary

Almost every problem in this pattern is a thin layer over a single 2D recurrence:

- **The core** compares last characters: on a match, extend the diagonal (`1 + dp[i-1][j-1]`); on a mismatch, take the best of dropping one character from either string (`max(dp[i-1][j], dp[i][j-1])`).
- **Reconstruction** (Print LCS, SCS) requires the full 2D table and a backtrack from `(m, n)` — space optimization is off the table here.
- **Palindrome problems** reduce to LCS by pairing a string with its reverse (`LPS = LCS(s, reverse(s))`), and insertion/deletion counts are simple arithmetic on the LPS.
- **Conversion and supersequence** problems are arithmetic identities on the LCS length: `m + n - 2·LCS` for edit distance via insert/delete, and `m + n - LCS` for the shortest common supersequence.
- **Counting variants** (Distinct Subsequences) keep the same state but **sum** the branches instead of maximizing, and demand care with overflow — prefer `i64` for the table.

Master the four-step ladder (recursion → memoization → tabulation → space optimization) on the core once, and every other problem becomes "set up the right two strings, then add a one-line transformation."

> **Last Updated:** 2026-06-26
