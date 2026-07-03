> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 2 of 4**

# Common Mistakes — Dynamic Programming

Twelve bugs that fail hidden tests or get flagged in interviews, each as a wrong/correct pair. Pair with [Coding Tips](./Coding%20Tips.md), [State Design](./State%20Design.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Memo sentinel collides with a valid `0`

`0` is a legitimate answer (zero coins, zero ways-as-boolean), so `-1`-init `Vec<i32>` is fine, but `0`-init silently returns "0" for uncomputed states.

```rust
// WRONG: default Vec<i32> is 0; can't tell "not computed" from "answer is 0"
let memo: Vec<i32> = vec![0; n];
fn solve(i: usize, memo: &mut Vec<i32>) -> i32 { if memo[i] != 0 { return memo[i]; } /* ... */ 0 }

// CORRECT: use -1 sentinel (when -1 is impossible) and fill it
let mut memo: Vec<i32> = vec![-1; n];
fn solve(i: usize, memo: &mut Vec<i32>) -> i32 { if memo[i] != -1 { return memo[i]; } /* ... */ 0 }
// or Vec<Option<i32>> with memo[i].is_some() when 0 IS a valid answer
let mut memo: Vec<Option<i32>> = vec![None; n];
```

## 2. Wrong knapsack loop order (0/1 vs unbounded)

```rust
// WRONG: 0/1 knapsack with ASCENDING capacity reuses each item → unbounded behavior
for &item in &items {
    for c in item..=cap { dp[c] = dp[c].max(dp[c - item] + val); }
}

// CORRECT: 0/1 needs DESCENDING capacity so each item is used once
for &item in &items {
    for c in (item..=cap).rev() { dp[c] = dp[c].max(dp[c - item] + val); }
}
```

## 3. Off-by-one in dp size (`n` vs `n+1`)

```rust
// WRONG: size n, then index dp[n] → index out of bounds; no room for empty base
let dp: Vec<i32> = vec![0; n];
// CORRECT: size n+1, index 0 is the empty-prefix base case
let mut dp: Vec<i32> = vec![0; n + 1];
dp[0] = base_value;
```

## 4. House Robber II — forgetting the circular constraint

```rust
// WRONG: treats circular houses as linear → may rob both first and last (adjacent)
return rob_linear(&nums, 0, n - 1);

// CORRECT: first and last are adjacent → take the better of excluding one end
if n == 1 { return nums[0]; }
return rob_linear(&nums, 0, n - 2).max(rob_linear(&nums, 1, n - 1));
```

## 5. Coin Change overflow / missing sentinel

```rust
// WRONG: i32::MAX + 1 overflows and corrupts the min
dp.fill(i32::MAX);
dp[a] = dp[a].min(dp[a - c] + 1);

// CORRECT: amount+1 is a safe "infinity" (no real answer exceeds amount)
dp.fill(amount + 1);
dp[a] = dp[a].min(dp[a - c] + 1);
if dp[amount] > amount { -1 } else { dp[amount] }
```

## 6. LCS base case row/column not zero-initialized conceptually

```rust
// WRONG: starting the loops at 0 and reading dp[i-1]/dp[j-1] at the edge
for i in 0..=m {
    for j in 0..=n { dp[i][j] = dp[i-1][j-1] + 1; }  // i-1 underflows (usize) crash
}

// CORRECT: row 0 and col 0 are LCS=0 (empty string); real loop starts at 1
// let mut dp: Vec<Vec<i32>> = vec![vec![0; n+1]; m+1];  // zero-initialized, row 0 / col 0 = 0
for i in 1..=m {
    for j in 1..=n {
        dp[i][j] = if a[i-1] == b[j-1] {
            dp[i-1][j-1] + 1
        } else {
            dp[i-1][j].max(dp[i][j-1])
        };
    }
}
```

## 7. Edit Distance base case (empty-string costs)

```rust
// WRONG: leaving the first row/column as 0 → claims converting "" to "abc" is free
let mut dp: Vec<Vec<i32>> = vec![vec![0; n+1]; m+1];  // all zeros

// CORRECT: building from/to empty costs the string length
for i in 0..=m { dp[i][0] = i as i32; }  // delete i chars
for j in 0..=n { dp[0][j] = j as i32; }  // insert j chars
```

## 8. Decode Ways — mishandling `'0'`

```rust
// WRONG: always adds both branches, accepts "06", "30", standalone "0"
dp[i] = dp[i-1] + dp[i-2];

// CORRECT: single digit only if != '0'; two digits only if in 10..26
let bytes = s.as_bytes();
if bytes[i-1] != b'0' { dp[i] += dp[i-1]; }
let two = (bytes[i-2] - b'0') as i32 * 10 + (bytes[i-1] - b'0') as i32;
if two >= 10 && two <= 26 { dp[i] += dp[i-2]; }
```

## 9. Palindrome DP wrong iteration order

`dp[i][j]` depends on `dp[i+1][j-1]` (the inner substring), so you must fill **shorter substrings first**.

```rust
// WRONG: top-down i then j reads dp[i+1][j-1] before it's computed
let chars: Vec<char> = s.chars().collect();
for i in 0..n {
    for j in 0..n { dp[i][j] = chars[i] == chars[j] && dp[i+1][j-1]; }
}

// CORRECT: i descending, j ascending (or iterate by increasing length)
let chars: Vec<char> = s.chars().collect();
for i in (0..n).rev() {
    for j in i..n {
        dp[i][j] = chars[i] == chars[j] && (j - i < 2 || dp[i+1][j-1]);
    }
}
```

## 10. Burst Balloons — missing boundary padding

```rust
// WRONG: bursting first / reading nums[i-1] or nums[j+1] off the ends → index errors
// and the subproblems aren't independent.

// CORRECT: pad with virtual 1s and think "which balloon k bursts LAST in (i,j)"
let mut arr = vec![0i32; n + 2];
arr[0] = 1;
arr[n + 1] = 1;
for i in 0..n { arr[i + 1] = nums[i]; }
// dp[i][j] = max over k in (i,j) of dp[i][k] + arr[i]*arr[k]*arr[j] + dp[k][j];
```

## 11. Subset Sum — negative / out-of-range index

```rust
// WRONG: blindly indexing dp[c - num] when num > c → underflow (usize) crash
dp[c] = dp[c] || dp[c - num];

// CORRECT: guard the capacity (or only loop c >= num)
if num <= c { dp[c] = dp[c] || dp[c - num]; }
```

## 12. Stock state machine — wrong transition

```rust
// WRONG: lets you sell without ever having bought, or buy twice in a row
hold = hold.max(cash + price);  // sign / source state mixed up

// CORRECT: hold comes from (stay holding) or (buy today from cash);
//          cash comes from (stay in cash) or (sell today what you hold)
let mut hold = i32::MIN;
let mut cash = 0i32;
for &price in &prices {
    let prev_cash = cash;
    cash = cash.max(hold + price);       // sell
    hold = hold.max(prev_cash - price);  // buy
}
cash
```

See [Stock Buy Sell](../Patterns/Stock%20Buy%20Sell.md) for the full state machine.

---

## Mistake → fix cheat sheet

| # | Mistake | Fix |
|---|---------|-----|
| 1 | `0`-init memo collides with valid 0 | `-1` sentinel or `Option<i32>`/`None` |
| 2 | Ascending cap for 0/1 knapsack | descending capacity |
| 3 | dp sized `n` not `n+1` | size `n+1`, base at index 0 |
| 4 | Linear robber on circular houses | two passes, exclude each end |
| 5 | `i32::MAX + 1` overflow | `amount+1` sentinel |
| 6 | LCS edge crash | loop from 1, row/col 0 = 0 |
| 7 | Edit Distance zero base row | `dp[i][0]=i`, `dp[0][j]=j` |
| 8 | Decode Ways ignores `'0'` | guard single + two-digit ranges |
| 9 | Palindrome wrong order | shorter substrings first |
| 10 | No balloon padding | pad 1s, "last to burst" |
| 11 | Negative subset index | guard `num <= c` |
| 12 | Bad stock transition | hold/cash from correct prior states |

> **Last Updated:** 2026-06-26
