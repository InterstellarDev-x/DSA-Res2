> **Topic:** [Dynamic Programming](../README.md) · **Pattern 3 of 8**

# Knapsack (Subset Sum)

The **Knapsack** family is the workhorse of dynamic programming interviews. Every problem here reduces to the same question: *"Given a collection of items, can I pick a subset (or a multiset) that satisfies some constraint on a running total?"* The constraint is usually a target sum, a capacity, or a count of ways.

There are two flavors:

- **0/1 Knapsack** — each item may be picked **at most once**. After deciding on item `i`, you move on to item `i-1` (or `i+1`). This covers Subset Sum, Partition, Target Sum, and the classic weight/value knapsack.
- **Unbounded Knapsack** — each item may be picked **unlimited times**. After picking item `i`, you may pick it *again*, so you stay on the same index. This covers Coin Change and Coin Change II.

The crucial mechanical difference shows up in the **1D space-optimized loop direction**: 0/1 iterates capacity **downward**, unbounded iterates **upward**. The whole second half of this note is dedicated to getting that right.

This pattern is sibling to [1D DP (Fibonacci Style)](./1D%20DP%20(Fibonacci%20Style).md) and [Grids & Unique Paths](./Grids%20&%20Unique%20Paths.md). Master the 0/1 Knapsack template in problem 18 and everything else is a transformation onto it.

---

## 1. The Core State Machine

Every problem below is built from one decision per item: **take it** or **skip it**.

| Concept | 0/1 Knapsack | Unbounded Knapsack |
| --- | --- | --- |
| State | `(index, remaining)` | `(index, remaining)` |
| On **skip** | `f(index-1, remaining)` | `f(index-1, remaining)` |
| On **take** | `f(index-1, remaining - w[index])` | `f(index, remaining - w[index])` (stay!) |
| Item reuse | at most once | unlimited |
| 1D capacity loop | **downward** (`cap → 0`) | **upward** (`0 → cap`) |

The recurrence template:

```text
f(i, rem) = combine( f(i-1, rem),                 // skip item i
                     g(item i) + f(next, rem - w_i) )   // take item i, if w_i <= rem
```

where `combine` is `OR` (feasibility), `+` (counting), `min`/`max` (optimization), and `next` is `i-1` (0/1) or `i` (unbounded).

---

## 2. Subset Sum Equal to Target (GFG)

**Problem.** Given an array `arr` of `n` non-negative integers and a value `target`, return `true` if **some subset** of `arr` sums to exactly `target`.

- **State:** `f(i, t)` = can we form sum `t` using elements from index `0..i`?
- **Recurrence:** `f(i, t) = f(i-1, t) OR f(i-1, t - arr[i])` (second term only if `arr[i] <= t`)
- **Base cases:** `f(i, 0) = true` (empty subset sums to 0); `f(0, t) = (arr[0] == t)`.

### 2a. Pure recursion

```rust
struct SubsetSum;

impl SubsetSum {
    fn is_subset_sum(arr: &[i32], target: i32) -> bool {
        Self::solve(arr.len() as i32 - 1, target, arr)
    }

    fn solve(idx: i32, target: i32, arr: &[i32]) -> bool {
        if target == 0 { return true; }          // empty subset works
        if idx == 0 { return arr[0] == target; } // only first element left
        let not_take = Self::solve(idx - 1, target, arr);
        let mut take = false;
        if arr[idx as usize] <= target {
            take = Self::solve(idx - 1, target - arr[idx as usize], arr);
        }
        take || not_take
    }
}
```

Time `O(2^n)`, exponential — every element branches twice.

### 2b. Memoization (`Vec<Vec<i32>>` initialized to `-1`)

We cache booleans as `i32`: `-1` = unvisited, `0` = false, `1` = true. (You could also use a separate `visited` array.)

```rust
struct SubsetSum;

impl SubsetSum {
    fn is_subset_sum(arr: &[i32], target: usize) -> bool {
        let n = arr.len();
        let mut dp = vec![vec![-1i32; target + 1]; n];
        Self::solve(n as i32 - 1, target as i32, arr, &mut dp)
    }

    fn solve(idx: i32, target: i32, arr: &[i32], dp: &mut Vec<Vec<i32>>) -> bool {
        if target == 0 { return true; }
        if idx == 0 { return arr[0] == target; }
        let i = idx as usize;
        let t = target as usize;
        if dp[i][t] != -1 { return dp[i][t] == 1; }

        let not_take = Self::solve(idx - 1, target, arr, dp);
        let mut take = false;
        if arr[i] <= target {
            take = Self::solve(idx - 1, target - arr[i], arr, dp);
        }
        let result = take || not_take;
        dp[i][t] = if result { 1 } else { 0 };
        result
    }
}
```

Time `O(n * target)`, Space `O(n * target)` + recursion stack `O(n)`.

### 2c. Tabulation

```rust
struct SubsetSum;

impl SubsetSum {
    fn is_subset_sum(arr: &[i32], target: usize) -> bool {
        let n = arr.len();
        let mut dp = vec![vec![false; target + 1]; n];

        for i in 0..n { dp[i][0] = true; } // sum 0 always possible
        if arr[0] as usize <= target { dp[0][arr[0] as usize] = true; }

        for i in 1..n {
            for t in 1..=target {
                let not_take = dp[i - 1][t];
                let take = if arr[i] as usize <= t {
                    dp[i - 1][t - arr[i] as usize]
                } else {
                    false
                };
                dp[i][t] = take || not_take;
            }
        }
        dp[n - 1][target]
    }
}
```

### 2d. Space optimization to 1D

Each row depends only on the previous row, so we keep one array. Because this is **0/1** (each item once), we iterate capacity **downward** to avoid reusing `arr[i]` within the same pass.

```rust
struct SubsetSum;

impl SubsetSum {
    fn is_subset_sum(arr: &[i32], target: usize) -> bool {
        let n = arr.len();
        let mut dp = vec![false; target + 1];
        dp[0] = true; // empty subset

        for i in 0..n {
            let w = arr[i] as usize;
            for t in (w..=target).rev() { // DOWNWARD: 0/1 semantics
                dp[t] = dp[t] || dp[t - w];
            }
        }
        dp[target]
    }
}
```

Time `O(n * target)`, Space `O(target)`.

### 2e. Dry-run

`arr = [3, 1, 2]`, `target = 4`. Track the 1D `dp` over capacities `0..4`.

| Step | Processed item | dp[0] | dp[1] | dp[2] | dp[3] | dp[4] |
| --- | --- | --- | --- | --- | --- | --- |
| init | — | T | F | F | F | F |
| i=0 | 3 | T | F | F | T | F |
| i=1 | 1 | T | T | F | T | T |
| i=2 | 2 | T | T | T | T | T |

- After item `3`: only sums `{0, 3}` reachable.
- After item `1` (down `4→1`): `dp[4] = dp[4] \|\| dp[3] = T` (3+1), `dp[1] = dp[1] \|\| dp[0] = T`.
- After item `2`: `dp[2]` becomes T (just the `2`), and the others stay.
- `dp[4] == true` → subset `{3, 1}` sums to 4. ✔

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * target)` |
| Space (2D) | `O(n * target)` |
| Space (1D) | `O(target)` |

---

## 3. Partition Equal Subset Sum (LC 416)

**Problem.** Given `nums`, return `true` if it can be split into two subsets with **equal sum**.

**Key insight.** Two equal halves means each must equal `totalSum / 2`. If `totalSum` is **odd**, it is impossible immediately. Otherwise this is exactly *Subset Sum* with `target = totalSum / 2` — reuse the 1D routine verbatim.

```rust
struct PartitionEqualSubsetSum;

impl PartitionEqualSubsetSum {
    fn can_partition(nums: &[i32]) -> bool {
        let total: i32 = nums.iter().sum();
        if (total & 1) == 1 { return false; } // odd total can't split evenly

        let target = (total / 2) as usize;
        let mut dp = vec![false; target + 1];
        dp[0] = true;

        for &num in nums {
            let w = num as usize;
            for t in (w..=target).rev() { // 0/1 → downward
                dp[t] = dp[t] || dp[t - w];
            }
        }
        dp[target]
    }
}
```

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * total/2)` |
| Space | `O(total/2)` |

---

## 4. Count Subsets with Sum K (GFG)

**Problem.** Count the number of subsets whose sum equals `k`.

- **State:** `f(i, t)` = number of subsets of `arr[0..i]` summing to `t`.
- **Recurrence:** `f(i, t) = f(i-1, t) + f(i-1, t - arr[i])`
- **Combine is `+`** instead of `OR`.

### Edge case: zeros

If `arr[i] == 0`, you may either include or exclude it without changing the sum, so each zero **doubles** the count. This must be reflected in the base cases. With the index-0 base case below, an element equal to `0` contributes both "take" and "skip" paths automatically because `dp[0][0]` should count *two* ways (take the zero, or skip it). The clean way to handle this:

- `dp[0][0] = if arr[0] == 0 { 2 } else { 1 }`
- `if arr[0] != 0 && arr[0] as usize <= k { dp[0][arr[0] as usize] = 1; }`

```rust
struct CountSubsetsWithSumK;

impl CountSubsetsWithSumK {
    const MOD: i64 = 1_000_000_007;

    fn perfect_sum(arr: &[i32], k: usize) -> i32 {
        let n = arr.len();
        let mut dp = vec![vec![0i64; k + 1]; n];

        // Base row handling zeros explicitly.
        if arr[0] == 0 {
            dp[0][0] = 2;            // skip the zero OR take the zero
        } else {
            dp[0][0] = 1;            // only the empty subset
            if arr[0] as usize <= k { dp[0][arr[0] as usize] = 1; }
        }

        for i in 1..n {
            for t in 0..=k {
                let not_take = dp[i - 1][t];
                let take = if arr[i] as usize <= t {
                    dp[i - 1][t - arr[i] as usize]
                } else {
                    0
                };
                dp[i][t] = (take + not_take) % Self::MOD;
            }
        }
        dp[n - 1][k] as i32
    }
}
```

1D version (downward, 0/1). When zeros are *not* present the simpler `dp[0] = 1` base works; with zeros prefer the 2D base above, or pre-handle the count of zeros separately and multiply by `2^(#zeros)`.

```rust
struct CountSubsetsWithSumK;

impl CountSubsetsWithSumK {
    const MOD: i64 = 1_000_000_007;

    fn perfect_sum(arr: &[i32], k: usize) -> i32 {
        let mut dp = vec![0i64; k + 1];
        dp[0] = 1; // assumes the explicit-zero handling above for arrays containing 0
        for &num in arr {
            let w = num as usize;
            for t in (w..=k).rev() { // 0/1 → downward
                dp[t] = (dp[t] + dp[t - w]) % Self::MOD;
            }
        }
        dp[k] as i32
    }
}
```

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * k)` |
| Space (2D) | `O(n * k)` |
| Space (1D) | `O(k)` |

---

## 5. 0/1 Knapsack (GFG) — CORE TEMPLATE

**Problem.** Given `n` items with `weight[i]` and `value[i]` and a knapsack of capacity `W`, maximize total value such that total weight `<= W`. Each item used **at most once**.

This is the canonical template. Internalize all four stages.

- **State:** `f(i, cap)` = max value using items `0..i` with capacity `cap`.
- **Recurrence:** `f(i, cap) = max( f(i-1, cap), value[i] + f(i-1, cap - weight[i]) )` (take only if `weight[i] <= cap`)
- **Base case:** `f(0, cap) = if weight[0] <= cap { value[0] } else { 0 }`.

### 5a. Pure recursion

```rust
struct Knapsack01;

impl Knapsack01 {
    fn knapsack(w: usize, weight: &[i32], value: &[i32]) -> i32 {
        Self::solve(weight.len() as i32 - 1, w as i32, weight, value)
    }

    fn solve(idx: i32, cap: i32, weight: &[i32], value: &[i32]) -> i32 {
        if idx == 0 {
            return if weight[0] <= cap { value[0] } else { 0 };
        }
        let i = idx as usize;
        let not_take = Self::solve(idx - 1, cap, weight, value);
        let mut take = i32::MIN;
        if weight[i] <= cap {
            take = value[i] + Self::solve(idx - 1, cap - weight[i], weight, value);
        }
        take.max(not_take)
    }
}
```

### 5b. Memoization (`Vec<Vec<i32>>` initialized to `-1`)

```rust
struct Knapsack01;

impl Knapsack01 {
    fn knapsack(w: usize, weight: &[i32], value: &[i32]) -> i32 {
        let n = weight.len();
        let mut dp = vec![vec![-1i32; w + 1]; n];
        Self::solve(n as i32 - 1, w as i32, weight, value, &mut dp)
    }

    fn solve(idx: i32, cap: i32, weight: &[i32], value: &[i32], dp: &mut Vec<Vec<i32>>) -> i32 {
        if idx == 0 {
            return if weight[0] <= cap { value[0] } else { 0 };
        }
        let i = idx as usize;
        let c = cap as usize;
        if dp[i][c] != -1 { return dp[i][c]; }

        let not_take = Self::solve(idx - 1, cap, weight, value, dp);
        let mut take = i32::MIN;
        if weight[i] <= cap {
            take = value[i] + Self::solve(idx - 1, cap - weight[i], weight, value, dp);
        }
        dp[i][c] = take.max(not_take);
        dp[i][c]
    }
}
```

### 5c. Tabulation

```rust
struct Knapsack01;

impl Knapsack01 {
    fn knapsack(w: usize, weight: &[i32], value: &[i32]) -> i32 {
        let n = weight.len();
        let mut dp = vec![vec![0i32; w + 1]; n];

        for cap in (weight[0] as usize)..=w {
            dp[0][cap] = value[0];
        }

        for i in 1..n {
            for cap in 0..=w {
                let not_take = dp[i - 1][cap];
                let take = if weight[i] as usize <= cap {
                    value[i] + dp[i - 1][cap - weight[i] as usize]
                } else {
                    i32::MIN
                };
                dp[i][cap] = take.max(not_take);
            }
        }
        dp[n - 1][w]
    }
}
```

### 5d. Space optimization to 1D — iterate capacity DOWNWARD

```rust
struct Knapsack01;

impl Knapsack01 {
    fn knapsack(w: usize, weight: &[i32], value: &[i32]) -> i32 {
        let n = weight.len();
        let mut dp = vec![0i32; w + 1];

        for cap in (weight[0] as usize)..=w {
            dp[cap] = value[0];
        }

        for i in 1..n {
            for cap in (weight[i] as usize..=w).rev() { // DOWNWARD
                let take = value[i] + dp[cap - weight[i] as usize];
                dp[cap] = dp[cap].max(take);
            }
        }
        dp[w]
    }
}
```

**Why downward?** In the 1D array, `dp[cap]` is being updated in place. The recurrence for 0/1 reads from the **previous row** at `dp[cap - weight[i]]`. If we iterate **upward**, by the time we reach `cap` the slot `dp[cap - weight[i]]` has *already been updated in this same pass* — meaning item `i` could be counted twice (that is the *unbounded* behavior). Iterating **downward** guarantees `dp[cap - weight[i]]` still holds the previous row's value, preserving the "each item once" semantics.

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * W)` |
| Space (2D) | `O(n * W)` |
| Space (1D) | `O(W)` |

---

## 6. Coin Change (LC 322) — Minimum Coins, UNBOUNDED

**Problem.** Given coin denominations `coins` and an `amount`, return the **fewest** coins needed to make `amount`, or `-1` if impossible. Each coin may be used **unlimited** times.

- **State:** `f(i, rem)` = min coins to form `rem` using coins `0..i`.
- **Recurrence:** `f(i, rem) = min( f(i-1, rem), 1 + f(i, rem - coins[i]) )` — note **`f(i, ...)` on take, NOT `f(i-1, ...)`**, because the coin can be reused.
- **Base case:** `f(0, rem) = if rem % coins[0] == 0 { rem / coins[0] } else { INF }`.

### 6a. Memoization

```rust
struct CoinChangeMin;

impl CoinChangeMin {
    const INF: i32 = 1_000_000_000;

    fn coin_change(coins: &[i32], amount: usize) -> i32 {
        let n = coins.len();
        let mut dp = vec![vec![-1i32; amount + 1]; n];
        let ans = Self::solve(n as i32 - 1, amount as i32, coins, &mut dp);
        if ans >= Self::INF { -1 } else { ans }
    }

    fn solve(idx: i32, rem: i32, coins: &[i32], dp: &mut Vec<Vec<i32>>) -> i32 {
        if idx == 0 {
            return if rem % coins[0] == 0 { rem / coins[0] } else { Self::INF };
        }
        let i = idx as usize;
        let r = rem as usize;
        if dp[i][r] != -1 { return dp[i][r]; }

        let not_take = Self::solve(idx - 1, rem, coins, dp);
        let take = if coins[i] <= rem {
            1 + Self::solve(idx, rem - coins[i], coins, dp) // STAY on idx
        } else {
            Self::INF
        };
        dp[i][r] = take.min(not_take);
        dp[i][r]
    }
}
```

### 6b. Tabulation, 1D — iterate capacity UPWARD

```rust
struct CoinChangeMin;

impl CoinChangeMin {
    fn coin_change(coins: &[i32], amount: usize) -> i32 {
        let mut dp = vec![amount as i32 + 1; amount + 1]; // sentinel "infinity" (can't exceed amount coins)
        dp[0] = 0;

        for &coin in coins {
            let w = coin as usize;
            for rem in w..=amount { // UPWARD: unbounded reuse
                dp[rem] = dp[rem].min(1 + dp[rem - w]);
            }
        }
        if dp[amount] > amount as i32 { -1 } else { dp[amount] }
    }
}
```

**Why upward?** Because the coin is **unbounded**, the take branch reads from the *current* row (`dp[rem - coin]` updated in this same pass). Iterating **upward** is exactly what makes `dp[rem - coin]` already reflect "this coin used one or more times," allowing multiple uses of the same coin. (This is the mirror image of the 0/1 downward rule from problem 18.)

`-1` is returned when `dp[amount]` never improved past the infinity sentinel.

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * amount)` |
| Space (1D) | `O(amount)` |

---

## 7. Coin Change II (LC 518) — Count Ways, UNBOUNDED

**Problem.** Count the number of **combinations** of coins that make `amount`. Unlimited coins; `{1,2}` and `{2,1}` are the **same combination** (order irrelevant).

- **State:** `f(i, rem)` = number of combinations using coins `0..i`.
- **Recurrence:** `f(i, rem) = f(i-1, rem) + f(i, rem - coins[i])` (stay on `i` for take).
- **Base case:** `f(0, rem) = if rem % coins[0] == 0 { 1 } else { 0 }`.

To count **combinations** (not permutations), the 1D form puts **coins on the outer loop** and **amount on the inner loop**. This ordering ensures each combination is counted once because coins are introduced in a fixed order.

### 7a. Memoization

```rust
struct CoinChangeII;

impl CoinChangeII {
    fn change(amount: usize, coins: &[i32]) -> i32 {
        let n = coins.len();
        let mut dp = vec![vec![-1i64; amount + 1]; n];
        Self::solve(n as i32 - 1, amount as i32, coins, &mut dp) as i32
    }

    fn solve(idx: i32, rem: i32, coins: &[i32], dp: &mut Vec<Vec<i64>>) -> i64 {
        if idx == 0 {
            return if rem % coins[0] == 0 { 1 } else { 0 };
        }
        let i = idx as usize;
        let r = rem as usize;
        if dp[i][r] != -1 { return dp[i][r]; }

        let not_take = Self::solve(idx - 1, rem, coins, dp);
        let take = if coins[i] <= rem {
            Self::solve(idx, rem - coins[i], coins, dp) // STAY on idx
        } else {
            0
        };
        dp[i][r] = take + not_take;
        dp[i][r]
    }
}
```

### 7b. Tabulation, 1D — outer loop coins, inner loop amount (UPWARD)

```rust
struct CoinChangeII;

impl CoinChangeII {
    fn change(amount: usize, coins: &[i32]) -> i32 {
        let mut dp = vec![0i64; amount + 1];
        dp[0] = 1; // one way to make 0: pick nothing

        for &coin in coins {              // OUTER: coins
            let w = coin as usize;
            for rem in w..=amount { // INNER: amount, upward
                dp[rem] += dp[rem - w];
            }
        }
        dp[amount] as i32
    }
}
```

If you swapped the loops (amount outer, coins inner) you would count **permutations** instead of combinations.

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * amount)` |
| Space (1D) | `O(amount)` |

---

## 8. Target Sum (LC 494)

**Problem.** Assign `+` or `-` to each element of `nums` so the resulting expression equals `target`. Count the number of assignments.

**Transformation.** Let `S1` = sum of elements assigned `+`, and `S2` = sum of elements assigned `-`. Then:

```text
S1 - S2 = target
S1 + S2 = sum         (every element is in exactly one group)
-----------------------------
2 * S1 = sum + target
S1     = (sum + target) / 2
```

So the problem reduces to **Count Subsets with Sum = `(sum + target) / 2`** (problem 17).

**Guards.**
- If `(sum + target)` is **odd**, `S1` is not an integer → answer `0`.
- If `abs(target) > sum` → impossible → answer `0`.
- The derived subset target must be non-negative (it is, given the guards above, since `(sum + target) >= 0`).
- Zeros are still handled by the count-subset base case (each zero doubles the count).

```rust
struct TargetSum;

impl TargetSum {
    fn find_target_sum_ways(nums: &[i32], target: i32) -> i32 {
        let sum: i32 = nums.iter().sum();

        // S1 = (sum + target) / 2 must be a non-negative integer.
        if target.abs() > sum { return 0; }
        if ((sum + target) & 1) == 1 { return 0; }
        let s1 = ((sum + target) / 2) as usize;

        Self::count_subsets(nums, s1)
    }

    fn count_subsets(arr: &[i32], k: usize) -> i32 {
        let n = arr.len();
        let mut dp = vec![vec![0i32; k + 1]; n];

        if arr[0] == 0 {
            dp[0][0] = 2;                 // take or skip the zero
        } else {
            dp[0][0] = 1;
            if arr[0] as usize <= k { dp[0][arr[0] as usize] = 1; }
        }

        for i in 1..n {
            for t in 0..=k {
                let not_take = dp[i - 1][t];
                let take = if arr[i] as usize <= t { dp[i - 1][t - arr[i] as usize] } else { 0 };
                dp[i][t] = take + not_take;
            }
        }
        dp[n - 1][k]
    }
}
```

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * (sum+target)/2)` |
| Space | `O(n * k)` (reducible to `O(k)`) |

---

## 9. Partition Set Into Two Subsets With Minimum Difference (GFG)

**Problem.** Partition `arr` into two subsets `S1`, `S2` to minimize `|sum(S1) - sum(S2)|`. Return that minimum difference.

**Insight.** If one subset has sum `s`, the other has `total - s`, so the difference is `|total - 2s|`. We want to minimize this over all **reachable** subset sums `s`. Use the boolean Subset-Sum table to find every reachable `s` in `0..total`, then sweep.

By symmetry it suffices to scan `s` from `0` to `total/2` (each reachable `s` and its complement give the same difference), and minimize `total - 2*s`.

```rust
struct MinSubsetSumDifference;

impl MinSubsetSumDifference {
    fn min_difference(arr: &[i32]) -> i32 {
        let total: i32 = arr.iter().sum();
        let total_usize = total as usize;

        // dp[s] = is sum s reachable by some subset?
        let mut dp = vec![false; total_usize + 1];
        dp[0] = true;
        for &num in arr {
            let w = num as usize;
            for s in (w..=total_usize).rev() { // 0/1 → downward
                dp[s] = dp[s] || dp[s - w];
            }
        }

        let mut best = i32::MAX;
        for s in 0..=(total_usize / 2) {
            if dp[s] {
                let diff = total - 2 * s as i32; // = |s - (total - s)|, s <= total/2
                best = best.min(diff);
            }
        }
        best
    }
}
```

A worked check on `arr = [1, 6, 11, 5]`, `total = 23`: reachable sums include `11` (= 6+5) and `12` (= 1+11). At `s = 11`, `diff = 23 - 22 = 1`. That is the minimum. ✔

| Aspect | Complexity |
| --- | --- |
| Time | `O(n * total)` |
| Space | `O(total)` |

---

## 10. 0/1 vs Unbounded Knapsack — Loop Order

This is the single most error-prone detail in the entire pattern. Get it wrong and feasibility/counting silently produces wrong answers.

| Dimension | 0/1 Knapsack | Unbounded Knapsack |
| --- | --- | --- |
| Item usage | at most **once** | **unlimited** |
| Recurrence on take | `f(i-1, rem - w)` → **move to `i-1`** | `f(i, rem - w)` → **stay on `i`** |
| 2D row read on take | previous row (`dp[i-1][...]`) | current row (`dp[i][...]`) |
| **1D capacity loop** | **DOWNWARD** (`cap → w`) | **UPWARD** (`w → cap`) |
| Why | downward keeps `dp[cap-w]` as the *old* (previous-row) value, so item used once | upward lets `dp[cap-w]` be *already updated* this pass, so item reused |
| Members | Subset Sum, Partition (416, MinDiff), Count Subsets, Target Sum (494), classic knapsack (GFG) | Coin Change (322), Coin Change II (518), rod cutting, unbounded knapsack |

**The mantra:**

```text
0/1       → take moves to index-1  → 1D loop goes DOWNWARD
Unbounded → take stays on index    → 1D loop goes UPWARD
```

A second subtlety, specific to **counting combinations** (Coin Change II): put the **items on the outer loop** so combinations are counted once (order ignored). Swapping to amount-outer counts permutations.

---

## 11. Recognition Signals

| If the problem says... | Reach for... | Combine op | Loop dir (1D) |
| --- | --- | --- | --- |
| "subset that sums to target?" (yes/no) | Subset Sum | `OR` | down |
| "split into two equal halves" | Partition (target = sum/2) | `OR` | down |
| "how many subsets sum to K" | Count Subsets | `+` | down |
| "+/- signs to reach target" | Target Sum → Count Subsets | `+` | down |
| "minimize difference of two groups" | Min Subset Diff | `OR` then sweep | down |
| "max value, capacity W, each item once" | 0/1 Knapsack | `max` | down |
| "fewest coins, reuse allowed" | Coin Change | `min` | up |
| "count ways to make amount, reuse allowed" | Coin Change II | `+` | up (coins outer) |
| "unlimited copies of each item" | Unbounded Knapsack | `max`/`min`/`+` | up |

Quick triage:

- **Each item once?** → 0/1, capacity loop **down**.
- **Unlimited copies?** → Unbounded, capacity loop **up**.
- **Yes/No** → boolean `OR`. **Count** → `+`. **Best** → `min`/`max`.
- **Negative or transformed target?** → algebra it back to a subset-sum (Target Sum).

---

## 12. Summary

- Everything in this pattern is the same `(index, remaining)` state machine with a **take/skip** decision.
- **Problem 18 (0/1 Knapsack)** is the template: master recursion → memo (initialized to `-1`) → tabulation → 1D. The other 0/1 problems are transformations:
  - **Subset Sum** → boolean `OR`.
  - **Partition Equal (416)** → subset sum with `target = total/2`, return false on odd total.
  - **Count Subsets / Target Sum (494)** → `+` counting; Target Sum reduces via `S1 = (sum+target)/2`.
  - **Min Subset Diff** → reachable sums + sweep `min(total - 2s)`.
- **Unbounded** problems (Coin Change 322 / II 518) keep the index on **take** and iterate the 1D capacity loop **upward**; Coin Change II additionally loops **coins outer** to count combinations.
- The **loop-direction rule** (down for 0/1, up for unbounded) is the highest-yield thing to memorize — it is the difference between a correct and a subtly broken solution.
- Use `min` / `max` for optimization, a comparison lambda rather than subtraction for custom comparators, and `i64` accumulators when counts can overflow `i32` (Coin Change II).

> **Last Updated:** 2026-06-26
