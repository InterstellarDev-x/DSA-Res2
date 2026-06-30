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

```java
public class LCSRecursion {

    public int longestCommonSubsequence(String s1, String s2) {
        return solve(s1, s2, s1.length(), s2.length());
    }

    // f(i, j) = LCS length of s1[0..i-1] and s2[0..j-1]
    private int solve(String s1, String s2, int i, int j) {
        if (i == 0 || j == 0) {
            return 0; // empty prefix
        }
        if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
            return 1 + solve(s1, s2, i - 1, j - 1);
        }
        return Math.max(solve(s1, s2, i - 1, j), solve(s1, s2, i, j - 1));
    }
}
```

**Time:** `O(2^(m+n))` in the worst case — exponential due to overlapping subproblems. **Space:** `O(m + n)` recursion depth.

---

## 3. Step 2 — Memoization (Top-Down)

There are only `(m + 1) * (n + 1)` distinct states, so we cache them. We initialize the memo with `-1` using `Arrays.fill` to distinguish "uncomputed" from a legitimate `0`.

```java
import java.util.Arrays;

public class LCSMemo {

    public int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int[] row : dp) {
            Arrays.fill(row, -1);
        }
        return solve(s1, s2, m, n, dp);
    }

    private int solve(String s1, String s2, int i, int j, int[][] dp) {
        if (i == 0 || j == 0) {
            return 0;
        }
        if (dp[i][j] != -1) {
            return dp[i][j];
        }
        if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
            dp[i][j] = 1 + solve(s1, s2, i - 1, j - 1, dp);
        } else {
            dp[i][j] = Math.max(solve(s1, s2, i - 1, j, dp),
                                solve(s1, s2, i, j - 1, dp));
        }
        return dp[i][j];
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)` table + `O(m + n)` recursion stack.

---

## 4. Step 3 — Tabulation (Bottom-Up)

Convert the recursion into iteration. Because `dp[i][j]` depends on `dp[i-1][j-1]`, `dp[i-1][j]`, and `dp[i][j-1]`, we fill the table in increasing order of `i` and `j`. Row `0` and column `0` are already `0` (default Java init).

```java
public class LCSTabulation {

    public int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`.

---

## 5. Step 4 — Space Optimization (Two Rows)

Each row depends only on the previous row, so we keep just two 1D arrays of size `n + 1`.

```java
public class LCSSpaceOptimized {

    public int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[] prev = new int[n + 1];
        int[] curr = new int[n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    curr[j] = 1 + prev[j - 1];
                } else {
                    curr[j] = Math.max(prev[j], curr[j - 1]);
                }
            }
            int[] temp = prev;
            prev = curr;
            curr = temp; // reuse the array; values get overwritten next pass
        }
        return prev[n];
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

```java
class Solution {
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length();
        int n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

---

## 8. Problem 24 — Print LCS (GFG)

Beyond the length, reconstruct the actual subsequence string by **backtracking** the filled DP table from `dp[m][n]` to `dp[0][0]`.

**Backtracking rule:** start at `(m, n)`.
- If `s1[i-1] == s2[j-1]`, this character is part of the LCS — prepend it and move diagonally to `(i-1, j-1)`.
- Otherwise move toward the larger neighbor: if `dp[i-1][j] >= dp[i][j-1]` go up, else go left.

```java
public class PrintLCS {

    public String printLCS(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];

        // Build the standard LCS table.
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        // Backtrack to reconstruct the subsequence.
        StringBuilder sb = new StringBuilder();
        int i = m;
        int j = n;
        while (i > 0 && j > 0) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                sb.append(s1.charAt(i - 1)); // characters are collected in reverse
                i--;
                j--;
            } else if (dp[i - 1][j] >= dp[i][j - 1]) {
                i--;
            } else {
                j--;
            }
        }
        return sb.reverse().toString();
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

```java
public class LongestCommonSubstring {

    public int longestCommonSubstr(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];
        int best = 0;

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                    best = Math.max(best, dp[i][j]);
                } else {
                    dp[i][j] = 0; // reset on mismatch
                }
            }
        }
        return best;
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`, reducible to `O(n)` with two rows.

---

## 10. Problem 26 — Longest Palindromic Subsequence (LC 516)

**Key reduction:** the longest palindromic subsequence of `s` is exactly the LCS of `s` and `reverse(s)`. A palindrome reads the same forwards and backwards, so the longest subsequence common to `s` and its reverse must itself be a palindrome.

```java
class Solution {
    public int longestPalindromeSubseq(String s) {
        String rev = new StringBuilder(s).reverse().toString();
        return lcs(s, rev);
    }

    private int lcs(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
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

```java
class Solution {
    public int minInsertions(String s) {
        int n = s.length();
        return n - longestPalindromeSubseq(s);
    }

    private int longestPalindromeSubseq(String s) {
        String rev = new StringBuilder(s).reverse().toString();
        int n = s.length();
        int[][] dp = new int[n + 1][n + 1];
        for (int i = 1; i <= n; i++) {
            for (int j = 1; j <= n; j++) {
                if (s.charAt(i - 1) == rev.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[n][n];
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

```java
public class MinInsertDeleteAtoB {

    public int minOperations(String a, String b) {
        int m = a.length();
        int n = b.length();
        int lcs = longestCommonSubsequence(a, b);
        int deletions = m - lcs;
        int insertions = n - lcs;
        return deletions + insertions; // == m + n - 2 * lcs
    }

    private int longestCommonSubsequence(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }
        return dp[m][n];
    }
}
```

**Time:** `O(m * n)`. **Space:** `O(m * n)`.

---

## 13. Problem 29 — Shortest Common Supersequence (LC 1092)

The shortest common supersequence (SCS) is the shortest string that has both `s1` and `s2` as subsequences. Its **length** is `m + n - LCS`, because the LCS characters are shared and counted once instead of twice.

To **reconstruct** the SCS, build the full LCS table and backtrack. When characters match, emit that single character and step diagonally. When they differ, emit the character from whichever direction the LCS table grew from, stepping in that direction. Finally, flush whatever remains of either string.

```java
class Solution {
    public String shortestCommonSupersequence(String s1, String s2) {
        int m = s1.length();
        int n = s2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                    dp[i][j] = 1 + dp[i - 1][j - 1];
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        StringBuilder sb = new StringBuilder();
        int i = m;
        int j = n;
        while (i > 0 && j > 0) {
            if (s1.charAt(i - 1) == s2.charAt(j - 1)) {
                sb.append(s1.charAt(i - 1)); // shared character, write once
                i--;
                j--;
            } else if (dp[i - 1][j] >= dp[i][j - 1]) {
                sb.append(s1.charAt(i - 1)); // came from s1
                i--;
            } else {
                sb.append(s2.charAt(j - 1)); // came from s2
                j--;
            }
        }
        // Flush leftover prefix of whichever string remains.
        while (i > 0) {
            sb.append(s1.charAt(i - 1));
            i--;
        }
        while (j > 0) {
            sb.append(s2.charAt(j - 1));
            j--;
        }
        return sb.reverse().toString();
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

> **Overflow note:** counts can exceed `Integer.MAX_VALUE`. LeetCode guarantees the answer fits in a 32-bit signed int, but **intermediate** sums can overflow. Using `long` for the table (or being careful) is the safe choice; below we use `int[][]` matching the LeetCode guarantee and note where `long` would be required for unconstrained inputs.

```java
class Solution {
    public int numDistinct(String s, String t) {
        int m = s.length();
        int n = t.length();
        // Use long internally to avoid intermediate overflow for general inputs.
        long[][] dp = new long[m + 1][n + 1];

        for (int i = 0; i <= m; i++) {
            dp[i][0] = 1; // empty t matched exactly one way
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (s.charAt(i - 1) == t.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
                } else {
                    dp[i][j] = dp[i - 1][j];
                }
            }
        }
        return (int) dp[m][n];
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
- **Counting variants** (Distinct Subsequences) keep the same state but **sum** the branches instead of maximizing, and demand care with overflow — prefer `long` for the table.

Master the four-step ladder (recursion → memoization → tabulation → space optimization) on the core once, and every other problem becomes "set up the right two strings, then add a one-line transformation."

> **Last Updated:** 2026-06-26
