> **Topic:** [Dynamic Programming](../README.md) · **Pattern 7 of 8**

# Partition DP (MCM)

Partition DP (also called *interval DP* or *Matrix Chain Multiplication DP*) solves problems where an interval `[i, j]` must be split into two (or more) sub-intervals, and the cost of the whole depends on a **partition point** `k` plus the recursively-computed costs of the pieces. The defining trait is that **you cannot decide the answer for `[i, j]` greedily** — you must *try every place to cut* and take the best. This pattern powers classic interview problems like Matrix Chain Multiplication, Burst Balloons, and Palindrome Partitioning.

Two flavours appear in this pattern:

- **Middle-partition DP** — cut the interval at some interior `k` and combine `dp(i, k)` and `dp(k+1, j)` (MCM, Cut a Stick, Burst Balloons, Boolean Evaluation).
- **Front-partition DP** — anchor at the front index `i`, try every prefix `[i..k]` as the *first* piece, then recurse on the suffix `[k+1..n]` (Palindrome Partitioning II, Partition Array for Maximum Sum). This is really a 1-D partition DP and is closely related to the [DP on Strings](./DP%20on%20Strings.md) family.

---

## 1. The Partition DP Template

The core idea: an interval `[i, j]` is solved by considering **all partition points `k`** inside it. For each `k` we split the interval into two parts, solve each part recursively, and add the *cost of merging* the two parts at that split.

```
dp(i, j) = best over k of  dp(i, k) + dp(k + 1, j) + cost(i, k, j)
```

- `k` ranges over `[i, j - 1]` (split between `k` and `k + 1`), or over `[i, j]` for problems where `k` itself is the merge point (Burst Balloons, Cut a Stick).
- `best` is `min` or `max` depending on whether we minimize cost or maximize value.
- `cost(i, k, j)` is the price of combining the two solved halves — the part that is *unique to each problem*.

### Base case

When the interval is empty or a single element (`i >= j`, or `i + 1 == j`, etc.), there is **nothing to partition** so the cost is `0` (or a problem-specific trivial value).

### Memoized (top-down) skeleton

```rust
struct PartitionTemplate {
    memo: Vec<Vec<i32>>,
}

impl PartitionTemplate {
    fn solve(&mut self, arr: &[i32]) -> i32 {
        let n = arr.len();
        self.memo = vec![vec![-1; n]; n];
        Self::dp(arr, 0, n - 1, &mut self.memo)
    }

    fn dp(arr: &[i32], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
        if i >= j {
            return 0;                       // empty / single-element interval
        }
        if memo[i][j] != -1 {
            return memo[i][j];
        }
        let mut best = i32::MAX;       // use i32::MIN for maximization
        for k in i..j {                // try every partition point
            let cost = Self::dp(arr, i, k, memo)
                     + Self::dp(arr, k + 1, j, memo)
                     + Self::merge_cost(arr, i, k, j);
            best = best.min(cost);
        }
        memo[i][j] = best;
        best
    }

    fn merge_cost(_arr: &[i32], _i: usize, _k: usize, _j: usize) -> i32 {
        0                               // problem-specific
    }
}
```

### Tabulated (bottom-up) skeleton

Because `dp(i, j)` depends on **smaller intervals** (`[i, k]` and `[k+1, j]`, both shorter than `[i, j]`), we fill the table by **increasing interval length**. Equivalently, iterate `i` from high to low and `j` from low to high — this guarantees the sub-intervals are already computed.

```rust
struct PartitionTemplateTab;

impl PartitionTemplateTab {
    fn solve(arr: &[i32]) -> i32 {
        let n = arr.len();
        let mut dp = vec![vec![0i32; n]; n];  // dp[i][i] == 0 already (empty interval)

        for i in (0..n).rev() {             // decreasing i
            for j in (i + 1)..n {           // increasing j  (j > i)
                let mut best = i32::MAX;
                for k in i..j {
                    let cost = dp[i][k] + dp[k + 1][j] + Self::merge_cost(arr, i, k, j);
                    best = best.min(cost);
                }
                dp[i][j] = best;
            }
        }
        dp[0][n - 1]
    }

    fn merge_cost(_arr: &[i32], _i: usize, _k: usize, _j: usize) -> i32 {
        0
    }
}
```

> **Why decreasing `i`, increasing `j`?** `dp[i][k]` uses the same row `i` but a column `k < j` (already done in this row). `dp[k+1][j]` uses a *lower* row `k + 1 > i` (already done because we go from the bottom up). Filling by length `len = j - i` from `1` to `n - 1` is an equivalent, more explicit ordering.

**Complexity (generic):** `O(n^2)` states × `O(n)` partition points = **`O(n^3)` time**, **`O(n^2)` space**.

---

## 2. Matrix Chain Multiplication (GFG)

Given dimensions `arr[]` where matrix `i` (1-indexed) has shape `arr[i-1] x arr[i]`, find the minimum number of scalar multiplications to compute the product `M1 · M2 · … · M(n-1)`.

**Mapping to the template:** Interval `[i, j]` represents the product of matrices `i..j`. We pick a split `k` (the matrix at which we place the last multiplication), pay `dp(i, k) + dp(k+1, j)` for the two halves, then **`cost = arr[i-1] * arr[k] * arr[j]`** to multiply the two resulting matrices (shapes `arr[i-1] x arr[k]` and `arr[k] x arr[j]`).

- **State:** `dp[i][j]` = min cost to multiply matrices `i..j` (using 1-based matrix indices, `1 <= i <= j <= n-1`).
- **Recurrence:** `dp[i][j] = min over k in [i, j-1] of dp[i][k] + dp[k+1][j] + arr[i-1]*arr[k]*arr[j]`
- **Base case:** `dp[i][i] = 0` (a single matrix needs no multiplication).
- **Answer:** `dp[1][n-1]`.

### Recursion + Memoization

```rust
struct MatrixChainMemo {
    memo: Vec<Vec<i32>>,
}

impl MatrixChainMemo {
    fn matrix_multiplication(&mut self, arr: &[i32]) -> i32 {
        let n = arr.len();                 // n-1 matrices
        self.memo = vec![vec![-1; n]; n];
        Self::dp(arr, 1, n - 1, &mut self.memo)
    }

    fn dp(arr: &[i32], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
        if i >= j {
            return 0;                       // single matrix (or empty)
        }
        if memo[i][j] != -1 {
            return memo[i][j];
        }
        let mut best = i32::MAX;
        for k in i..j {
            let cost = Self::dp(arr, i, k, memo)
                     + Self::dp(arr, k + 1, j, memo)
                     + arr[i - 1] * arr[k] * arr[j];
            best = best.min(cost);
        }
        memo[i][j] = best;
        best
    }
}
```

### Tabulation

```rust
struct MatrixChainTab;

impl MatrixChainTab {
    fn matrix_multiplication(arr: &[i32]) -> i32 {
        let n = arr.len();
        let mut dp = vec![vec![0i32; n]; n];  // dp[i][i] = 0 by default

        for i in (1..n).rev() {
            for j in (i + 1)..n {
                let mut best = i32::MAX;
                for k in i..j {
                    let cost = dp[i][k]
                             + dp[k + 1][j]
                             + arr[i - 1] * arr[k] * arr[j];
                    best = best.min(cost);
                }
                dp[i][j] = best;
            }
        }
        dp[1][n - 1]
    }
}
```

### Dry run

Input: `arr = [10, 20, 30, 40]` → matrices `M1(10x20)`, `M2(20x30)`, `M3(30x40)`.

We need `dp[1][3]`. First the length-1 intervals are all `0`. Then length-2:

- `dp[1][2]`: only `k = 1` → `arr[0]*arr[1]*arr[2] = 10*20*30 = 6000`.
- `dp[2][3]`: only `k = 2` → `arr[1]*arr[2]*arr[3] = 20*30*40 = 24000`.

Now `dp[1][3]`, trying both splits:

| `k` | `dp[1][k]` | `dp[k+1][3]` | merge `arr[0]*arr[k]*arr[3]` | total |
|-----|-----------|-------------|----------------------------|-------|
| 1   | `dp[1][1]=0` | `dp[2][3]=24000` | `10*20*40 = 8000` | 32000 |
| 2   | `dp[1][2]=6000` | `dp[3][3]=0` | `10*30*40 = 12000` | 18000 |

`dp[1][3] = min(32000, 18000) = 18000`. So `(M1·M2)·M3` is optimal.

### Complexity

| Aspect | Value |
|--------|-------|
| States | `O(n^2)` intervals |
| Transition | `O(n)` partition points |
| **Time** | **`O(n^3)`** |
| **Space** | **`O(n^2)`** (+ `O(n)` recursion stack for memo) |

---

## 3. Minimum Cost to Cut a Stick (LC 1547)

A stick of length `n` must be cut at every position in `cuts[]`. The cost of one cut equals the current length of the stick being cut; the total cost depends on the order. Minimize total cost.

**Mapping to the template:** Add the **boundary sentinels** `0` and `n` to `cuts`, then **sort**. Now treat the sorted array as the indices defining intervals. `dp(i, j)` = min cost to make all cuts strictly between sentinel `i` and sentinel `j`. We choose which cut `k` (with `i < k < j`) to perform *first* in this segment; its cost is the current segment length `arr[j] - arr[i]`, and the two resulting sub-segments are solved recursively. (Choosing the *first* cut is equivalent to the partition point.)

- **State:** `dp[i][j]` = min cost to perform all cuts strictly between `arr[i]` and `arr[j]` in the padded, sorted array.
- **Recurrence:** `dp[i][j] = (arr[j] - arr[i]) + min over k in (i, j) of dp[i][k] + dp[k][j]` — only when `j > i + 1` (there is at least one cut inside).
- **Base case:** `dp[i][i+1] = 0` (no cut between adjacent sentinels).
- **Answer:** `dp[0][m-1]` where `m = cuts.length + 2`.

### Memoization

```rust
struct CutStickMemo {
    memo: Vec<Vec<i32>>,
}

impl CutStickMemo {
    fn min_cost(&mut self, n: i32, cuts: &[i32]) -> i32 {
        let c = cuts.len();
        let mut arr = vec![0i32; c + 2];
        arr[0] = 0;
        arr[c + 1] = n;
        for i in 0..c {
            arr[i + 1] = cuts[i];
        }
        arr.sort();

        let m = arr.len();
        self.memo = vec![vec![-1; m]; m];
        Self::dp(&arr, 0, m - 1, &mut self.memo)
    }

    fn dp(arr: &[i32], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
        if j <= i + 1 {
            return 0;                       // no cut lies strictly inside
        }
        if memo[i][j] != -1 {
            return memo[i][j];
        }
        let mut best = i32::MAX;
        for k in (i + 1)..j {
            let cost = Self::dp(arr, i, k, memo) + Self::dp(arr, k, j, memo) + (arr[j] - arr[i]);
            best = best.min(cost);
        }
        memo[i][j] = best;
        best
    }
}
```

### Tabulation

```rust
struct CutStickTab;

impl CutStickTab {
    fn min_cost(n: i32, cuts: &[i32]) -> i32 {
        let c = cuts.len();
        let mut arr = vec![0i32; c + 2];
        arr[0] = 0;
        arr[c + 1] = n;
        for i in 0..c {
            arr[i + 1] = cuts[i];
        }
        arr.sort();

        let m = arr.len();
        let mut dp = vec![vec![0i32; m]; m];

        for i in (0..m).rev() {
            for j in (i + 2)..m {              // need at least one cut between
                let mut best = i32::MAX;
                for k in (i + 1)..j {
                    let cost = dp[i][k] + dp[k][j] + (arr[j] - arr[i]);
                    best = best.min(cost);
                }
                dp[i][j] = best;
            }
        }
        dp[0][m - 1]
    }
}
```

### Complexity

| Aspect | Value |
|--------|-------|
| Let `m = cuts.length + 2` | — |
| **Time** | **`O(m^3)`** |
| **Space** | **`O(m^2)`** |

---

## 4. Burst Balloons (LC 312)

Given `nums[]`, bursting balloon `i` yields `nums[left] * nums[i] * nums[right]` where `left`/`right` are the *currently adjacent* balloons. Maximize total coins.

**Mapping to the template — the key trick:** thinking about which balloon to burst *first* is hard (its neighbors keep changing). Instead, think about the **last balloon `k` to burst** within an interval. When `k` is last in `(i, j)`, its neighbors are exactly the fixed boundaries `i` and `j`, so the gain is `padded[i] * padded[k] * padded[j]`, **independent of the order** the rest were burst.

We pad `nums` with `1` on both ends so boundary multiplications are well-defined.

- **State:** `dp[i][j]` = max coins from bursting all balloons strictly between padded indices `i` and `j` (i.e. open interval `(i, j)`).
- **Recurrence:** `dp[i][j] = max over k in (i, j) of dp[i][k] + dp[k][j] + padded[i]*padded[k]*padded[j]`.
- **Base case:** `dp[i][j] = 0` when `j - i <= 1` (no balloon inside).
- **Answer:** `dp[0][m-1]` with `m = nums.length + 2`.

### Memoization

```rust
struct BurstBalloonsMemo {
    memo: Vec<Vec<i32>>,
}

impl BurstBalloonsMemo {
    fn max_coins(&mut self, nums: &[i32]) -> i32 {
        let n = nums.len();
        let mut padded = vec![0i32; n + 2];
        padded[0] = 1;
        padded[n + 1] = 1;
        for i in 0..n {
            padded[i + 1] = nums[i];
        }

        let m = padded.len();
        self.memo = vec![vec![-1; m]; m];
        Self::dp(&padded, 0, m - 1, &mut self.memo)
    }

    fn dp(padded: &[i32], i: usize, j: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
        if j <= i + 1 {
            return 0;                       // no balloon between i and j
        }
        if memo[i][j] != -1 {
            return memo[i][j];
        }
        let mut best = i32::MIN;
        for k in (i + 1)..j {              // k = last balloon to burst in (i, j)
            let coins = Self::dp(padded, i, k, memo)
                      + Self::dp(padded, k, j, memo)
                      + padded[i] * padded[k] * padded[j];
            best = best.max(coins);
        }
        memo[i][j] = best;
        best
    }
}
```

### Tabulation

```rust
struct BurstBalloonsTab;

impl BurstBalloonsTab {
    fn max_coins(nums: &[i32]) -> i32 {
        let n = nums.len();
        let mut padded = vec![0i32; n + 2];
        padded[0] = 1;
        padded[n + 1] = 1;
        for i in 0..n {
            padded[i + 1] = nums[i];
        }

        let m = padded.len();
        let mut dp = vec![vec![0i32; m]; m];

        for i in (0..m).rev() {
            for j in (i + 2)..m {
                let mut best = i32::MIN;
                for k in (i + 1)..j {
                    let coins = dp[i][k]
                              + dp[k][j]
                              + padded[i] * padded[k] * padded[j];
                    best = best.max(coins);
                }
                dp[i][j] = best;
            }
        }
        dp[0][m - 1]
    }
}
```

### Complexity

| Aspect | Value |
|--------|-------|
| Let `m = nums.length + 2` | — |
| **Time** | **`O(m^3)` = `O(n^3)`** |
| **Space** | **`O(n^2)`** |

---

## 5. Boolean Evaluation / Evaluate Expression to True (GFG)

Given a boolean expression string with operands `T`/`F` and operators `&`, `|`, `^`, count the number of ways to **parenthesize** it so it evaluates to `true`. Answer modulo `1000` (per GFG).

**Mapping to the template:** Interval `[i, j]` is a sub-expression. We **partition on every operator position `k`** (operators sit at odd indices). For each operator we recursively obtain `(trueCount, falseCount)` of the left part `[i, k-1]` and right part `[k+1, j]`, then **combine** based on the operator, carrying **both** counts because computing "ways to be true" needs "ways to be false" of the children (e.g. for `^`).

- **State:** `dp[i][j][isTrue]` = number of ways the sub-expression `[i, j]` evaluates to `isTrue` (`1` = true, `0` = false). We encode the boolean as a third dimension.
- **Recurrence:** for each operator at `k`, with left counts `(lt, lf)` and right `(rt, rf)` and `total = (lt+lf)*(rt+rf)`:
  - `&` : `true += lt*rt`,  `false += total - lt*rt`
  - `|` : `false += lf*rf`, `true += total - lf*rf`
  - `^` : `true += lt*rf + lf*rt`, `false += lt*rt + lf*rf`
- **Base case:** single character — `T` contributes `(true=1, false=0)`, `F` contributes `(true=0, false=1)`.
- **Answer:** `dp[0][n-1][true]`.

### Memoization (encoded state)

We encode `(i, j, isTrue)` into one integer key and memoize in a `HashMap`. Counts are kept modulo `1000` to match the GFG variant.

```rust
use std::collections::HashMap;

struct BooleanEvalMemo {
    memo: HashMap<String, i32>,
}

impl BooleanEvalMemo {
    const MOD: i32 = 1000;

    fn count_ways(&mut self, s: &str) -> i32 {
        self.memo.clear();
        let n = s.len();
        Self::solve_dp(s, 0, n - 1, true, &mut self.memo)
    }

    // returns number of ways s[i..j] evaluates to is_true
    fn solve_dp(s: &str, i: usize, j: usize, is_true: bool, memo: &mut HashMap<String, i32>) -> i32 {
        if i > j {
            return 0;
        }
        if i == j {
            let c = s.as_bytes()[i] as char;
            return if is_true { if c == 'T' { 1 } else { 0 } }
                   else       { if c == 'F' { 1 } else { 0 } };
        }
        let key = format!("{}_{}_{}", i, j, if is_true { 1 } else { 0 });  // encoded state
        if let Some(&val) = memo.get(&key) {
            return val;
        }

        let mut ways = 0i32;
        let mut k = i + 1;
        while k < j {                                    // operators at odd offsets
            let op = s.as_bytes()[k] as char;
            let lt = Self::solve_dp(s, i, k - 1, true, memo);
            let lf = Self::solve_dp(s, i, k - 1, false, memo);
            let rt = Self::solve_dp(s, k + 1, j, true, memo);
            let rf = Self::solve_dp(s, k + 1, j, false, memo);

            if op == '&' {
                ways += if is_true { lt * rt }
                        else       { lt * rf + lf * rt + lf * rf };
            } else if op == '|' {
                ways += if is_true { lt * rt + lt * rf + lf * rt }
                        else       { lf * rf };
            } else { // '^'
                ways += if is_true { lt * rf + lf * rt }
                        else       { lt * rt + lf * rf };
            }
            ways %= Self::MOD;
            k += 2;
        }

        memo.insert(key, ways);
        ways
    }
}
```

> **Note on the merge cost:** unlike pure MCM, here the "cost" combines *both* truth counts of the children, which is why we recurse for `true` and `false` separately. This is still a partition DP — `k` is the operator we evaluate **last**.

### Complexity

| Aspect | Value |
|--------|-------|
| States | `O(n^2)` intervals × `2` truth values |
| Transition | `O(n)` operator positions |
| **Time** | **`O(n^3)`** |
| **Space** | **`O(n^2)`** (memo + recursion) |

---

## 6. Palindrome Partitioning II (LC 132)

Partition string `s` so every substring is a palindrome; return the **minimum number of cuts**. This is a **front-partition** variant: we fix the front index `i` and try every prefix `[i..k]` as the first palindromic piece.

**Mapping to the template:** Instead of splitting an interval into two and recursing on both halves, we anchor the left end. `dp(i)` = min cuts for the suffix `s[i..n-1]`. For each `k >= i` where `s[i..k]` is a palindrome, we make a cut after `k` (cost `1`) and recurse on `dp(k+1)`. The recurrence is 1-D, but the *idea* — try every partition point — is identical. We **precompute** an `isPalindrome[i][j]` table so each check is `O(1)`.

- **State:** `dp[i]` = min cuts needed for `s[i..n-1]`.
- **Recurrence:** `dp[i] = min over k in [i, n-1] with isPal[i][k] of (1 + dp[k+1])`; if `isPal[i][n-1]` then `dp[i] = 0` (no cut).
- **Base case:** `dp[n] = -1` so that the final whole-string palindrome adds `1 + (-1) = 0` cuts cleanly.
- **Answer:** `dp[0]`.

### Precompute isPalindrome table

```rust
struct PalindromePartitionII;

impl PalindromePartitionII {
    fn build_is_palindrome(s: &[u8]) -> Vec<Vec<bool>> {
        let n = s.len();
        let mut is_pal = vec![vec![false; n]; n];
        for i in (0..n).rev() {
            for j in i..n {
                if s[i] == s[j] && (j - i < 2 || is_pal[i + 1][j - 1]) {
                    is_pal[i][j] = true;
                }
            }
        }
        is_pal
    }

    // Tabulated front-partition DP
    fn min_cut(s: &str) -> i32 {
        let n = s.len();
        let bytes = s.as_bytes();
        let is_pal = Self::build_is_palindrome(bytes);

        let mut dp = vec![0i32; n + 1];
        dp[n] = -1;                         // sentinel so last palindrome costs 0 cuts

        for i in (0..n).rev() {
            let mut best = i32::MAX;
            for k in i..n {                  // s[i..k] is the first piece
                if is_pal[i][k] {
                    best = best.min(1 + dp[k + 1]);
                }
            }
            dp[i] = best;
        }
        dp[0]
    }
}
```

### Memoized variant

```rust
struct PalindromePartitionIIMemo {
    is_pal: Vec<Vec<bool>>,
    memo: Vec<i32>,
}

impl PalindromePartitionIIMemo {
    fn min_cut(&mut self, s: &str) -> i32 {
        let n = s.len();
        let bytes = s.as_bytes();
        self.is_pal = vec![vec![false; n]; n];
        for i in (0..n).rev() {
            for j in i..n {
                if bytes[i] == bytes[j] && (j - i < 2 || self.is_pal[i + 1][j - 1]) {
                    self.is_pal[i][j] = true;
                }
            }
        }
        self.memo = vec![-1; n];
        let is_pal = &self.is_pal;
        let memo = &mut self.memo;
        Self::dp(bytes, 0, is_pal, memo)
    }

    fn dp(s: &[u8], i: usize, is_pal: &Vec<Vec<bool>>, memo: &mut Vec<i32>) -> i32 {
        let n = s.len();
        if i == n {
            return -1;                      // sentinel
        }
        if memo[i] != -1 {
            return memo[i];
        }
        let mut best = i32::MAX;
        for k in i..n {
            if is_pal[i][k] {
                best = best.min(1 + Self::dp(s, k + 1, is_pal, memo));
            }
        }
        memo[i] = best;
        best
    }
}
```

### Complexity

| Aspect | Value |
|--------|-------|
| Palindrome table | `O(n^2)` time/space |
| DP states | `O(n)` |
| DP transition | `O(n)` |
| **Time** | **`O(n^2)`** |
| **Space** | **`O(n^2)`** (palindrome table) |

---

## 7. Partition Array for Maximum Sum (LC 1043)

Partition `arr[]` into contiguous subarrays of length **at most `k`**. After partitioning, every element in a subarray becomes the subarray's **maximum**. Return the maximum possible sum of the resulting array.

**Mapping to the template:** Another **front-partition** variant. `dp(i)` = max sum obtainable from `arr[i..n-1]`. From index `i` we try each subarray length `len` in `[1, k]` (the first piece is `arr[i..i+len-1]`), track the running max of that window, and recurse on `dp(i + len)`.

- **State:** `dp[i]` = max sum achievable starting at index `i`.
- **Recurrence:** `dp[i] = max over len in [1, min(k, n-i)] of (curMax * len) + dp[i+len]`, where `curMax = max(arr[i..i+len-1])`.
- **Base case:** `dp[n] = 0` (empty suffix).
- **Answer:** `dp[0]`.

### Tabulation (front partition)

```rust
struct PartitionForMaxSum;

impl PartitionForMaxSum {
    fn max_sum_after_partitioning(arr: &[i32], k: usize) -> i32 {
        let n = arr.len();
        let mut dp = vec![0i32; n + 1];          // dp[n] = 0 by default

        for i in (0..n).rev() {
            let mut cur_max = i32::MIN;
            let mut best = i32::MIN;
            for len in 1..=(k.min(n - i)) {
                cur_max = cur_max.max(arr[i + len - 1]);
                let candidate = cur_max * len as i32 + dp[i + len];
                best = best.max(candidate);
            }
            dp[i] = best;
        }
        dp[0]
    }
}
```

### Memoized variant

```rust
struct PartitionForMaxSumMemo {
    memo: Vec<i32>,
}

impl PartitionForMaxSumMemo {
    fn max_sum_after_partitioning(&mut self, arr: &[i32], k: usize) -> i32 {
        let n = arr.len();
        self.memo = vec![-1; n];
        Self::dp(arr, k, 0, &mut self.memo)
    }

    fn dp(arr: &[i32], k: usize, i: usize, memo: &mut Vec<i32>) -> i32 {
        let n = arr.len();
        if i == n {
            return 0;
        }
        if memo[i] != -1 {
            return memo[i];
        }
        let mut cur_max = i32::MIN;
        let mut best = i32::MIN;
        for len in 1..=(k.min(n - i)) {
            cur_max = cur_max.max(arr[i + len - 1]);
            let candidate = cur_max * len as i32 + Self::dp(arr, k, i + len, memo);
            best = best.max(candidate);
        }
        memo[i] = best;
        best
    }
}
```

### Complexity

| Aspect | Value |
|--------|-------|
| States | `O(n)` |
| Transition | `O(k)` lengths |
| **Time** | **`O(n * k)`** |
| **Space** | **`O(n)`** |

---

## 8. Recognition Signals

| Signal in the problem statement | Likely Partition DP? | Variant |
|----------------------------------|----------------------|---------|
| "Find the optimal way to **parenthesize / group / multiply**" | Yes | Middle (MCM) |
| Cost of combining two parts depends on **boundary values** (`arr[i-1]`, `arr[j]`) | Yes | Middle (MCM) |
| "Order of operations matters" and neighbors change after each step | Yes — think **last** operation | Middle (Burst Balloons, Cut a Stick) |
| Expression with operators to be parenthesized; count/optimize results | Yes — carry **multiple** child states | Middle (Boolean Eval) |
| "Cut the string/array into pieces, each piece must satisfy a property, minimize cuts / maximize value" | Yes | **Front** partition |
| "Subarrays of length at most `k`, replace by max/min" | Yes | **Front** partition |
| You must try **every split point** and combine — no greedy/single-pass works | Yes | Either |
| Answer naturally indexed by an **interval `[i, j]`** | Yes | Middle |
| Answer naturally indexed by a **single start index `i`** with a per-piece rule | Yes | Front |

---

## 9. Summary

| # | Problem | State | Partition point | Best | Time | Space |
|---|---------|-------|-----------------|------|------|-------|
| 42 | Matrix Chain Multiplication | `dp[i][j]` over matrices | `k` = last multiply | min | `O(n^3)` | `O(n^2)` |
| 43 | Min Cost to Cut a Stick | `dp[i][j]` over sentinels | `k` = first cut | min | `O(m^3)` | `O(m^2)` |
| 44 | Burst Balloons | `dp[i][j]` open interval | `k` = **last** balloon | max | `O(n^3)` | `O(n^2)` |
| 45 | Boolean Evaluation | `dp[i][j][bool]` | `k` = operator (last) | sum | `O(n^3)` | `O(n^2)` |
| 46 | Palindrome Partitioning II | `dp[i]` suffix | `k` = end of first piece | min | `O(n^2)` | `O(n^2)` |
| 47 | Partition Array for Max Sum | `dp[i]` suffix | `len` of first piece | max | `O(n*k)` | `O(n)` |

**Key takeaways:**

- The signature move is `dp(i, j) = best over k of dp(i, k) + dp(k+1, j) + cost(i, k, j)` — *try every split*.
- **Middle-partition** problems (42–45) are indexed by an interval `[i, j]` and run in `O(n^3)`; fill the table by increasing length (or decreasing `i`, increasing `j`).
- For Burst Balloons and Cut a Stick, reasoning about the **last** operation (rather than the first) keeps the boundary cost fixed and well-defined — padding with sentinels makes the boundaries always valid.
- Boolean Evaluation generalizes the merge step: you must carry **both** truth counts because the cost of one child's truth depends on the other child's falsity.
- **Front-partition** problems (46–47) collapse to a 1-D `dp[i]` because only the *first* piece's boundary matters; they are cheaper (`O(n^2)` or `O(n*k)`) and overlap with the [DP on Strings](./DP%20on%20Strings.md) pattern.
- Always use `Vec<Vec<i32>>`/`Vec<i32>` for the DP table, `vec![-1; n]` for memo sentinels, and `.min()`/`.max()` for the merge.

> **Last Updated:** 2026-06-26
