> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Problems

# Microsoft — DP Interview Deep Dives

Microsoft favors classic DP with a production framing — decoding, grid counting, diff tools, and the maximal-square subproblem behind image/region detection. These four deep dives carry the full state/recurrence/base/Rust/complexity treatment plus the edge cases Microsoft hidden tests love.

See also: [OA-Qns → Microsoft](../OA-Qns/Microsoft.md) · [Interview Problems → Amazon](./Amazon.md) · [Interview Problems → Google](./Google.md) · [Topic README](../README.md)

---

## 1. Decode Ways — state transitions and the `'0'` minefield

**Problem.** A message of digits maps to letters by `'1'→A … '26'→Z`. Count the ways to decode it. A leading `'0'` or an isolated `'0'` is invalid.

### State and recurrence

- **State:** `dp[i]` = number of ways to decode the prefix `s[0..i)`.
- **Base:** `dp[0] = 1` (empty string decodes one way), `dp[1] = (s[0] != '0') ? 1 : 0`.
- **Recurrence for `i ≥ 2`:**
  - **Single digit** `s[i-1]`: valid iff it is `'1'..'9'` (not `'0'`) → add `dp[i-1]`.
  - **Two digits** `s[i-2..i-1]`: valid iff the number is in `10..26` → add `dp[i-2]`.

### The `'0'` cases that break naive solutions

- `"0"`, `"06"` → 0 ways (a `'0'` cannot start a code, and `"06"` ≠ `"6"`).
- `"10"`, `"20"` → 1 way (`'0'` only survives as the second digit of `10`/`20`).
- `"30"` → 0 ways (`30` is out of `10..26`, and standalone `'0'` is invalid).
- `"100"` → 0 ways (the trailing `'0'` has no valid partner).

```rust
fn num_decodings(s: &str) -> i32 {
    let s: Vec<char> = s.chars().collect();
    let n = s.len();
    if n == 0 || s[0] == '0' { return 0; }
    let mut dp = vec![0i32; n + 1];
    dp[0] = 1;
    dp[1] = 1;                              // s[0] != '0' guaranteed above
    for i in 2..=n {
        let one = s[i - 1];
        let ten = s[i - 2];
        if one != '0' {                     // valid single digit
            dp[i] += dp[i - 1];
        }
        let two = (ten as i32 - '0' as i32) * 10 + (one as i32 - '0' as i32);
        if two >= 10 && two <= 26 {         // valid two-digit code
            dp[i] += dp[i - 2];
        }
        // if neither branch fired, dp[i] stays 0 → dead end
    }
    dp[n]
}
```

Time `O(n)`, space `O(n)` (trivially reducible to O(1) with two variables). The crucial property: if a `'0'` cannot pair into `10`/`20`, `dp[i]` stays `0` and poisons everything after it — which is exactly correct.

### Common follow-ups
- **Decode Ways II** (LC 639) — `'*'` is a wildcard for `1..9`; multiply by the count of digit possibilities, mod `1e9+7`.
- "Return the decodings themselves" — backtracking instead of counting.

---

## 2. Unique Paths — combinatorial vs DP

**Problem.** A robot at the top-left of an `m × n` grid moves only right or down. Count distinct paths to the bottom-right.

### DP view

- **State:** `dp[i][j]` = number of paths to cell `(i, j)`.
- **Base:** first row and first column are all `1` (only one straight-line way).
- **Recurrence:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.

```rust
fn unique_paths(m: i32, n: i32) -> i32 {
    let m = m as usize;
    let n = n as usize;
    let mut dp = vec![1i32; n];             // first row
    for _i in 1..m {
        for j in 1..n {
            dp[j] += dp[j - 1];            // dp[j] (old=up) + dp[j-1] (left)
        }
    }
    dp[n - 1]
}
```

Time `O(m·n)`, space `O(n)` (rolling row).

### Combinatorial closed form

The robot makes exactly `(m-1)` downs and `(n-1)` rights in some order → choose positions: `C(m+n-2, m-1)`. Compute it incrementally to avoid overflow:

```rust
fn unique_paths(m: i32, n: i32) -> i32 {
    let mut result: i64 = 1;
    for i in 1..m {
        result = result * (n as i64 - 1 + i as i64) / i as i64;  // build C(m+n-2, m-1) step by step
    }
    result as i32
}
```

`O(m)` time. Interviewers like seeing you recognize the combinatorial identity, but the DP generalizes to obstacles (Unique Paths II) and cost (Min Path Sum) — see [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md).

### Common follow-ups
- **Unique Paths II** — obstacles; a blocked cell contributes 0 paths.
- **Minimum Path Sum** — switch from counting to minimizing cost.

---

## 3. Edit Distance — the production diff-tool framing

**Problem.** Same as the classic Edit Distance (LC 72), but Microsoft frames it as the engine behind a **diff / spell-checker / autocorrect**: the minimum-cost transformation between two text versions.

### State and recurrence (recap)

- `dp[i][j]` = min ops to turn `word1[0..i)` into `word2[0..j)`.
- Match → `dp[i-1][j-1]`; else `1 + min(delete dp[i-1][j], insert dp[i][j-1], replace dp[i-1][j-1])`.
- Base: `dp[i][0]=i`, `dp[0][j]=j` (cost of building from / down to empty).

```rust
fn min_distance(word1: String, word2: String) -> i32 {
    let m = word1.len();
    let n = word2.len();
    let w1: Vec<char> = word1.chars().collect();
    let w2: Vec<char> = word2.chars().collect();
    let mut dp = vec![vec![0i32; n + 1]; m + 1];
    for i in 0..=m { dp[i][0] = i as i32; }
    for j in 0..=n { dp[0][j] = j as i32; }
    for i in 1..=m {
        for j in 1..=n {
            if w1[i - 1] == w2[j - 1] {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + dp[i - 1][j - 1]
                    .min(dp[i - 1][j])
                    .min(dp[i][j - 1]);
            }
        }
    }
    dp[m][n]
}
```

### Production framing the interviewer is fishing for

- A real diff tool reports the **operations**, not just the count — so be ready to backtrack through `dp` and emit a script (`KEEP / INSERT / DELETE / REPLACE`).
- Real text is large → discuss the O(n) two-row space optimization (full code in [Interview Problems → Google](./Google.md)).
- Autocorrect weights operations differently (a transposition is cheap) → Damerau-Levenshtein extends the recurrence with an extra transposition term.

### Common follow-ups
- "Output the diff" — reconstruct from the table.
- "Weighted operations" — replace the `1 +` with per-op costs.
- "Transpositions allowed" — Damerau-Levenshtein.

---

## 4. Maximal Square — DP on a grid

**Problem.** Given a binary matrix, find the area of the largest square consisting entirely of `1`s.

### State and recurrence

- **State:** `dp[i][j]` = side length of the largest all-`1` square whose **bottom-right corner** is `(i, j)`.
- **Recurrence (when `matrix[i][j] == '1'`):** `dp[i][j] = 1 + min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1])`.
- **Why `min` of three neighbors:** a square of side `s` ending at `(i,j)` requires squares of side `s-1` ending at the cell above, the cell to the left, and the diagonal — the smallest of those three caps how far you can extend.
- **Base:** first row/column equal the cell value (square of side 1 if it's a `1`).

```rust
fn maximal_square(matrix: Vec<Vec<char>>) -> i32 {
    let m = matrix.len();
    let n = matrix[0].len();
    let mut dp = vec![vec![0i32; n + 1]; m + 1];  // 1-indexed → no boundary checks
    let mut max_side = 0;
    for i in 1..=m {
        for j in 1..=n {
            if matrix[i - 1][j - 1] == '1' {
                dp[i][j] = 1 + dp[i - 1][j - 1]
                    .min(dp[i - 1][j])
                    .min(dp[i][j - 1]);
                max_side = max_side.max(dp[i][j]);
            }
        }
    }
    max_side * max_side                    // area
}
```

Time `O(m·n)`, space `O(m·n)` (reducible to O(n) with a rolling row + a saved diagonal). The 1-indexed `dp` of size `(m+1)×(n+1)` removes all boundary checks — a tip echoed in [Coding Tips](../Interview%20Tips/Coding%20Tips.md).

### Common follow-ups
- "Largest *rectangle*" — different problem; histogram + stack, not this DP.
- "Count all squares" (LC 1277) — sum every `dp[i][j]` instead of taking the max.

---

## Summary table

| Problem | LC | Pattern | State | Time |
|---------|----|---------|-------|------|
| Decode Ways | 91 | [DP on Strings](../Patterns/DP%20on%20Strings.md) | `dp[i]` ways for prefix | O(n) |
| Unique Paths | 62 | [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md) | `dp[i][j]` paths to cell | O(m·n) |
| Edit Distance | 72 | [DP on Strings](../Patterns/DP%20on%20Strings.md) | `dp[i][j]` over two strings | O(m·n) |
| Maximal Square | 221 | [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md) | `dp[i][j]` square side at corner | O(m·n) |

> **Last Updated:** 2026-06-26
