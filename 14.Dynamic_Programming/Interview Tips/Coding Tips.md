> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 1 of 4**

# Coding Tips — Dynamic Programming

Ten habits that make DP problems fast to write and hard to get wrong in an interview. Read alongside [Common Mistakes](./Common%20Mistakes.md), [State Design](./State%20Design.md), and [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Use the 5-step DP framework, out loud

Every DP problem yields to the same five questions. Narrate them — interviewers grade the process:

1. **State** — what does `dp[...]` mean? ("min coins to make amount `a`")
2. **Recurrence** — how does a state combine smaller states?
3. **Base case(s)** — the smallest states you can fill directly.
4. **Iteration order** — the order that guarantees dependencies are ready.
5. **Answer** — which cell holds the final result?

If you can answer all five, the code writes itself. If you're stuck, you're missing one of them.

## 2. Start with recursion, then memoize

Write the brute-force recursion first — it makes the state and recurrence obvious. Then add a memo. This top-down path is faster to *derive* than jumping to tabulation, and it lets you show the overlapping subproblems explicitly.

```rust
// 1) brute force
fn solve(i: usize, cap: usize) -> i32 { todo!() }
// 2) add memo, same body
fn solve(i: usize, cap: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
    if memo[i][cap] != -1 { return memo[i][cap]; }
    // ...
    let result: i32 = todo!();
    memo[i][cap] = result;
    result
}
```

## 3. Initialize memo to a sentinel that can't be a real answer

For `Vec<Vec<i32>>`, initialize with `-1` (e.g. `vec![vec![-1; m]; n]`) **only when `-1` is not a valid answer**. When `0` is a legitimate result (counting, boolean), prefer `Vec<Vec<Option<i32>>>` and test for `None`, or pick a clearly out-of-range sentinel. This avoids the "0 collides with not-computed" bug — see [Common Mistakes](./Common%20Mistakes.md).

```rust
let memo: Vec<Vec<i32>> = vec![vec![-1; cap + 1]; n];
```

## 4. Use 1-indexed dp arrays to kill boundary checks

Sizing `dp` as `(n+1)` (or `(m+1)×(n+1)`) and treating index 0 as the empty base case removes `if (i-1 >= 0)` clutter from the inner loop. Edit Distance, LCS, Knapsack, and Maximal Square are all cleanest this way.

```rust
let dp: Vec<Vec<i32>> = vec![vec![0; n + 1]; m + 1]; // row 0 / col 0 are the empty-string base cases
```

## 5. Space-optimize: 2 rows → rolling, then → single array

Most 2D grid/string DP only reads the previous row (and sometimes the diagonal). Collapse to two 1D arrays, then often to one (iterate carefully to not clobber values you still need).

```rust
let mut prev = vec![0i32; n + 1];
let mut cur = vec![0i32; n + 1];
for _i in 1..=m {
    for j in 1..=n { cur[j] = f(prev[j], prev[j - 1], cur[j - 1]); }
    prev = cur.clone(); // or std::mem::swap(&mut prev, &mut cur)
}
```

## 6. Know the knapsack loop-order rule cold

- **0/1 knapsack (each item once):** iterate capacity **descending** in the 1D form, so each item is used at most once.
- **Unbounded (item reusable, e.g. Coin Change II):** iterate capacity **ascending**, so an item can be reused.

```rust
// 0/1: capacity DESCENDING
for &item in &items {
    for c in (item..=cap).rev() { dp[c] = todo!(); }
}
// unbounded: capacity ASCENDING
for &item in &items {
    for c in item..=cap { dp[c] += dp[c - item]; }
}
```

See [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md).

## 7. Encode multi-dimensional state into a HashMap key when the array is sparse/huge

When dimensions are large or unbounded (e.g. remaining sum can be negative, or state is `(i, j, k)`), a dense array wastes memory. Encode the tuple into a single key:

```rust
use std::collections::HashMap;
let mut memo: HashMap<i64, i32> = HashMap::new();
let key: i64 = ((i as i64) << 20) | ((j as i64) << 10) | (k as i64); // pack, mind the bit widths
```

Use an `i64` and verify the shift widths exceed each dimension's max so packing is collision-free.

## 8. Precompute palindrome tables before partition DP

For Palindrome Partitioning II and similar, precompute `is_pal[i][j]` in O(n²) first, then the partition DP reads it in O(1). Doing the palindrome check inside the partition loop turns O(n²) into O(n³).

```rust
let chars: Vec<char> = s.chars().collect();
let n = chars.len();
let mut is_pal: Vec<Vec<bool>> = vec![vec![false; n]; n];
for i in (0..n).rev() {
    for j in i..n {
        is_pal[i][j] = chars[i] == chars[j] && (j - i < 2 || is_pal[i + 1][j - 1]);
    }
}
```

## 9. Reach for binary search to make LIS O(n log n)

The O(n²) `dp[i]` LIS is the default, but the moment the interviewer says "can you do better?", switch to the patience-sorting `tails[]` array with binary search:

```rust
fn length_of_lis(nums: &[i32]) -> usize {
    let mut tails: Vec<i32> = Vec::new();
    let mut size = 0usize;
    for &x in nums {
        let (mut lo, mut hi) = (0usize, size);
        while lo < hi {                 // first index with tails[idx] >= x
            let mid = (lo + hi) >> 1;
            if tails[mid] < x { lo = mid + 1; } else { hi = mid; }
        }
        if lo == tails.len() { tails.push(x); } else { tails[lo] = x; }
        if lo == size { size += 1; }
    }
    size
}
```

See [Longest Increasing Subsequence](../Patterns/Longest%20Increasing%20Subsequence.md).

## 10. Draw the table for any 2D DP

Before coding a grid/string DP, sketch the table, mark the base row/column, draw the arrows showing which neighbors each cell reads, and pick a fill order that satisfies them. Five minutes of drawing prevents the off-by-one and iteration-order bugs that dominate [Common Mistakes](./Common%20Mistakes.md).

---

## Quick reference

| Tip | One-liner |
|-----|-----------|
| 1 | State → recurrence → base → order → answer. |
| 2 | Recursion first, then memo. |
| 3 | Sentinel must not collide with a real answer. |
| 4 | 1-index dp to drop boundary checks. |
| 5 | Two rows, then one. |
| 6 | 0/1 = descending capacity; unbounded = ascending. |
| 7 | Pack `(i,j,k)` into an `i64` key for HashMap memo. |
| 8 | Precompute `is_pal[][]` before partition DP. |
| 9 | LIS → `tails[]` + binary search for O(n log n). |
| 10 | Draw the table, mark base, draw the arrows. |

> **Last Updated:** 2026-06-26
