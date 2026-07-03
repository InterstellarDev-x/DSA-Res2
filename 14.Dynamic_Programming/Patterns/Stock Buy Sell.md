> **Topic:** [Dynamic Programming](../README.md) · **Pattern 5 of 8**

# Stock Buy Sell

The "Stock Buy Sell" family is a collection of problems that look superficially different — one transaction, infinite transactions, at most two, at most K, with a cooldown, with a fee — but every one of them is a specialization of a **single underlying state machine**. Once you internalize that machine, you stop memorizing six tricks and instead derive each variant by *constraining* one general recurrence. This page builds that unified model first, then solves all six problems with full, compilable Rust.

The whole family shares one rule: on any day you either **hold** a share or you don't, and you transition between those two states by buying (pay `prices[i]`) or selling (receive `prices[i]`). The variations only change *how many times* you may transition and *what penalties* apply.

Related patterns: [Knapsack (Subset Sum)](./Knapsack%20(Subset%20Sum).md) · [Longest Common Subsequence](./Longest%20Common%20Subsequence.md) · [Kadane (Max Subarray)](./Kadane%20(Max%20Subarray).md)

---

## 1. The Unified State Machine

Model the decision at each day with three coordinates:

```
dp[day][transactionsLeft][holding]
```

- `day` — index `i` into `prices`, ranging `0 .. n-1` (plus a terminal `n`).
- `transactionsLeft` — how many *complete* buy→sell pairs we are still allowed to do, `0 .. K`.
- `holding` — `0` means cash (no share), `1` means we currently own a share.

`dp[i][t][h]` = the **maximum profit obtainable from day `i` onward**, given we may still complete `t` transactions and we are currently in holding state `h`.

### State transitions

From any state we always have the option to **do nothing** (move to the next day, same holding). Beyond that:

- If `holding == 0` (we have cash), we may **buy**: pay `prices[i]`, flip to holding, transactions unchanged (a transaction is only "consumed" when *completed*, by convention at the sell).
- If `holding == 1` (we own a share), we may **sell**: receive `prices[i]`, flip to cash, and consume one transaction (`t-1`).

### General recurrence

We adopt the convention that a transaction is **counted at the buy** (decrement `t` when buying). Either convention works as long as you are consistent; counting at the buy keeps the `holding == 1` branch from needing `t`:

```
dp[i][t][0] = max( dp[i+1][t][0],                       // rest (stay in cash)
                   dp[i+1][t][1] + prices[i] )          // sell

dp[i][t][1] = max( dp[i+1][t][1],                       // rest (keep holding)
                   dp[i+1][t-1][0] - prices[i] )        // buy (consumes a transaction)
```

### Base cases

- `dp[n][t][h] = 0` for all `t, h` — no days left, no further profit.
- `dp[i][0][1]` is invalid/`-infinity` in spirit: you cannot be holding with zero transactions left if buying consumes the budget. In practice we guard with `t > 0` before the buy branch, so `dp[i][0][0] = 0`.

### Complexity of the general model

| Quantity | Value |
| --- | --- |
| States | `O(n · K · 2)` |
| Transitions per state | `O(1)` |
| Time | `O(n · K)` |
| Space (full table) | `O(n · K)` |
| Space (rolling day) | `O(K)` |

### How each variant specializes

| Problem | Constraint on the machine |
| --- | --- |
| LC 121 (one txn) | `K = 1` |
| LC 122 (infinite) | `K = ∞` → the `t` dimension collapses (it never binds) |
| LC 123 (at most 2) | `K = 2` |
| LC 188 (at most K) | the general machine itself |
| LC 309 (cooldown) | add a "must-rest" edge after selling |
| LC 714 (fee) | subtract `fee` on the sell edge |

The rest of this page walks each one and shows the specialization explicitly.

---

## 2. Problem 31 — Best Time to Buy and Sell Stock (LC 121, one transaction)

You may complete **at most one** transaction. This is the `K = 1` specialization. Because there is only one buy and one sell, the DP collapses to a single linear scan: track the **minimum price seen so far** (the best day to have bought) and the **best profit** achievable by selling today.

### State / recurrence (collapsed form)

- `minPrice` = lowest price in `prices[0..i]`.
- `maxProfit` = `max over i of (prices[i] - minPrice)`.

### Rust

```rust
struct StockI;

impl StockI {
    // O(n) time, O(1) space — the K = 1 specialization.
    fn max_profit(prices: &[i32]) -> i32 {
        let mut min_price = i32::MAX;
        let mut best = 0;
        for &price in prices {
            if price < min_price {
                min_price = price;
            } else if price - min_price > best {
                best = price - min_price;
            }
        }
        best
    }

    // The same problem via the unified holding-state DP with K = 1, O(1) space.
    // cash0 = best profit holding nothing, having used 0 buys.
    // hold1 = best profit holding a share (the one allowed buy).
    // cash1 = best profit holding nothing after the one allowed sell.
    fn max_profit_dp(prices: &[i32]) -> i32 {
        let mut hold1 = i32::MIN; // bought once, not yet sold
        let mut cash1 = 0;        // sold the one share
        for &price in prices {
            hold1 = hold1.max(-price);         // buy today (from initial cash 0)
            cash1 = cash1.max(hold1 + price);  // sell today
        }
        cash1
    }
}
```

### Dry run

`prices = [7, 1, 5, 3, 6, 4]`

| i | price | minPrice (before) | price - minPrice | best |
| --- | --- | --- | --- | --- |
| 0 | 7 | ∞ → 7 | — | 0 |
| 1 | 1 | 7 → 1 | — | 0 |
| 2 | 5 | 1 | 4 | 4 |
| 3 | 3 | 1 | 2 | 4 |
| 4 | 6 | 1 | 5 | **5** |
| 5 | 4 | 1 | 3 | 5 |

Answer: **5** (buy at 1, sell at 6).

### Complexity

| | Time | Space |
| --- | --- | --- |
| Running min | `O(n)` | `O(1)` |
| Holding DP | `O(n)` | `O(1)` |

---

## 3. Problem 32 — Best Time to Buy and Sell Stock II (LC 122, infinite transactions)

You may complete **as many transactions as you like** (buy then sell, repeatedly, but not simultaneously). This is `K = ∞`: the transaction-count dimension never binds, so it disappears. Two clean solutions exist.

### Greedy: sum every positive delta

Any multi-day upswing `a < b < c` yields the same profit whether you take it as one transaction `(c - a)` or as a chain `(b - a) + (c - b)`. So just capture **every** day-over-day increase.

### Holding-state DP

With unbounded transactions the state reduces to two values: best profit while in cash, best profit while holding.

### Rust

```rust
struct StockII;

impl StockII {
    // Greedy: O(n) time, O(1) space.
    fn max_profit(prices: &[i32]) -> i32 {
        let mut profit = 0;
        for i in 1..prices.len() {
            let delta = prices[i] - prices[i - 1];
            if delta > 0 {
                profit += delta;
            }
        }
        profit
    }

    // Holding-state DP (K = infinite => no transaction dimension), O(1) space.
    fn max_profit_dp(prices: &[i32]) -> i32 {
        let mut cash = 0;         // not holding
        let mut hold = i32::MIN;  // holding a share
        for &price in prices {
            let prev_cash = cash;
            cash = cash.max(hold + price);       // sell today
            hold = hold.max(prev_cash - price);  // buy today (reuse prior cash)
        }
        cash
    }
}
```

### Recurrence

```
cash[i] = max( cash[i-1], hold[i-1] + prices[i] )   // sell or rest
hold[i] = max( hold[i-1], cash[i-1] - prices[i] )   // buy or rest
```

Base: `cash = 0`, `hold = -prices[0]` (or `-∞` before day 0).

### Complexity

| | Time | Space |
| --- | --- | --- |
| Greedy | `O(n)` | `O(1)` |
| Holding DP | `O(n)` | `O(1)` |

---

## 4. Problem 33 — Best Time to Buy and Sell Stock III (LC 123, at most 2)

Now `K = 2`. We use the full three-dimensional machine but with the transaction axis fixed to size 2, then progressively optimize space.

### 4.1 Top-down memoization (3D)

```rust
struct StockIII_Memo;

impl StockIII_Memo {
    fn solve(day: usize, cap: usize, holding: usize, prices: &[i32], memo: &mut Vec<Vec<Vec<i32>>>) -> i32 {
        let n = prices.len();
        if day == n || cap == 0 {
            return 0;
        }
        if memo[day][cap][holding] != -1 {
            return memo[day][cap][holding];
        }

        let rest = Self::solve(day + 1, cap, holding, prices, memo);
        let action = if holding == 0 {
            // buy: pay price, now holding, cap unchanged (we count txn at sell)
            Self::solve(day + 1, cap, 1, prices, memo) - prices[day]
        } else {
            // sell: receive price, now in cash, consume one transaction
            Self::solve(day + 1, cap - 1, 0, prices, memo) + prices[day]
        };

        let result = rest.max(action);
        memo[day][cap][holding] = result;
        result
    }

    fn max_profit(prices: &[i32]) -> i32 {
        let n = prices.len();
        // dimensions: day (n), transactionsLeft (3: 0,1,2), holding (2)
        let mut memo = vec![vec![vec![-1i32; 2]; 3]; n];
        Self::solve(0, 2, 0, prices, &mut memo)
    }
}
```

### 4.2 Bottom-up tabulation (3D)

```rust
struct StockIII_Tab;

impl StockIII_Tab {
    fn max_profit(prices: &[i32]) -> i32 {
        let n = prices.len();
        if n == 0 {
            return 0;
        }
        // dp[day][cap][holding]; day goes n down to 0
        let mut dp = vec![vec![vec![0i32; 2]; 3]; n + 1];
        // base: dp[n][*][*] = 0 (already zero); cap == 0 also yields 0.

        for day in (0..n).rev() {
            for cap in 1..=2usize {
                // holding == 0 (can buy)
                dp[day][cap][0] = dp[day + 1][cap][0].max(
                    dp[day + 1][cap][1] - prices[day]);  // buy
                // holding == 1 (can sell)
                dp[day][cap][1] = dp[day + 1][cap][1].max(
                    dp[day + 1][cap - 1][0] + prices[day]);  // sell, consume txn
            }
        }
        dp[0][2][0]
    }
}
```

### 4.3 Space-optimized (four scalars)

With `K = 2` the entire table collapses to four running values: cost/profit after the 1st buy, after the 1st sell, after the 2nd buy, after the 2nd sell.

```rust
struct StockIII_Opt;

impl StockIII_Opt {
    // O(n) time, O(1) space.
    fn max_profit(prices: &[i32]) -> i32 {
        let mut buy1 = i32::MIN;  // after first buy
        let mut sell1 = 0;        // after first sell
        let mut buy2 = i32::MIN;  // after second buy
        let mut sell2 = 0;        // after second sell
        for &price in prices {
            buy1 = buy1.max(-price);
            sell1 = sell1.max(buy1 + price);
            buy2 = buy2.max(sell1 - price);
            sell2 = sell2.max(buy2 + price);
        }
        sell2
    }
}
```

### Complexity

| Approach | Time | Space |
| --- | --- | --- |
| 3D memo | `O(n · 2 · 2)` = `O(n)` | `O(n · 2 · 2)` = `O(n)` |
| 3D tabulation | `O(n)` | `O(n)` |
| Four scalars | `O(n)` | `O(1)` |

---

## 5. Problem 34 — Best Time to Buy and Sell Stock IV (LC 188, at most K)

The general machine, no constraint removed. Below are the memo, the tabulation, and a 2D / 1D space optimization. A standard micro-optimization: if `K >= n / 2`, transactions are effectively unlimited and we fall back to the LC 122 greedy.

### 5.1 Top-down memoization (3D)

```rust
struct StockIV_Memo;

impl StockIV_Memo {
    fn solve(day: usize, cap: usize, holding: usize, prices: &[i32], memo: &mut Vec<Vec<Vec<i32>>>) -> i32 {
        let n = prices.len();
        if day == n || cap == 0 {
            return 0;
        }
        if memo[day][cap][holding] != -1 {
            return memo[day][cap][holding];
        }

        let rest = Self::solve(day + 1, cap, holding, prices, memo);
        let action = if holding == 0 {
            Self::solve(day + 1, cap, 1, prices, memo) - prices[day]        // buy
        } else {
            Self::solve(day + 1, cap - 1, 0, prices, memo) + prices[day]   // sell, consume txn
        };

        let result = rest.max(action);
        memo[day][cap][holding] = result;
        result
    }

    fn max_profit(k: usize, prices: &[i32]) -> i32 {
        let n = prices.len();
        if n == 0 || k == 0 {
            return 0;
        }
        // dimensions: day (n), transactionsLeft (k+1), holding (2)
        let mut memo = vec![vec![vec![-1i32; 2]; k + 1]; n];
        Self::solve(0, k, 0, prices, &mut memo)
    }
}
```

### 5.2 Bottom-up tabulation (3D)

```rust
struct StockIV_Tab;

impl StockIV_Tab {
    fn max_profit(k: usize, prices: &[i32]) -> i32 {
        let n = prices.len();
        if n == 0 || k == 0 {
            return 0;
        }
        let mut dp = vec![vec![vec![0i32; 2]; k + 1]; n + 1]; // base dp[n][*][*] = 0

        for day in (0..n).rev() {
            for cap in 1..=k {
                dp[day][cap][0] = dp[day + 1][cap][0].max(
                    dp[day + 1][cap][1] - prices[day]);
                dp[day][cap][1] = dp[day + 1][cap][1].max(
                    dp[day + 1][cap - 1][0] + prices[day]);
            }
        }
        dp[0][k][0]
    }
}
```

### 5.3 Space-optimized to 2D (rolling day) and 1D (buy/sell pairs)

```rust
struct StockIV_Opt;

impl StockIV_Opt {
    // 2D: keep only the next day's plane -> O(k) space.
    fn max_profit_2d(k: usize, prices: &[i32]) -> i32 {
        let n = prices.len();
        if n == 0 || k == 0 {
            return 0;
        }
        let mut next = vec![vec![0i32; 2]; k + 1];
        for day in (0..n).rev() {
            let mut cur = vec![vec![0i32; 2]; k + 1];
            for cap in 1..=k {
                cur[cap][0] = next[cap][0].max(next[cap][1] - prices[day]);
                cur[cap][1] = next[cap][1].max(next[cap - 1][0] + prices[day]);
            }
            next = cur;
        }
        next[k][0]
    }

    // 1D: buy[t]/sell[t] arrays scanned over days, like the four-scalar StockIII
    // generalized to k pairs. O(k) space.
    fn max_profit_1d(k: usize, prices: &[i32]) -> i32 {
        if prices.is_empty() || k == 0 {
            return 0;
        }
        let mut buy = vec![i32::MIN; k + 1];
        let mut sell = vec![0i32; k + 1];
        // sell[0] stays 0; buy[0] unused for transactions.
        for &price in prices {
            for t in 1..=k {
                buy[t] = buy[t].max(sell[t - 1] - price);
                sell[t] = sell[t].max(buy[t] + price);
            }
        }
        sell[k]
    }
}
```

### Complexity

| Approach | Time | Space |
| --- | --- | --- |
| 3D memo | `O(n · k)` | `O(n · k)` |
| 3D tabulation | `O(n · k)` | `O(n · k)` |
| 2D rolling | `O(n · k)` | `O(k)` |
| 1D buy/sell | `O(n · k)` | `O(k)` |

---

## 6. Problem 35 — Best Time to Buy and Sell With Cooldown (LC 309)

Unlimited transactions, but after **selling** you must **skip the next day** (cooldown) before buying again. We add a third holding-ish state, or equivalently track a "just sold" state. Below is a clean 3-state formulation: `hold`, `sold` (sold today, in cooldown), `rest` (in cash, free to buy).

### States

- `hold` — own a share.
- `sold` — sold today; tomorrow is forced rest.
- `rest` — in cash and free to act (was resting or out of cooldown).

### Recurrence

```
hold[i] = max( hold[i-1], rest[i-1] - prices[i] )   // keep holding, or buy (only from rest)
sold[i] = hold[i-1] + prices[i]                      // must have been holding to sell
rest[i] = max( rest[i-1], sold[i-1] )                // stay in cash, or come off cooldown
```

Answer = `max(sold[n-1], rest[n-1])` (never end holding).

### Rust

```rust
struct StockCooldown;

impl StockCooldown {
    // O(n) time, O(1) space.
    fn max_profit(prices: &[i32]) -> i32 {
        if prices.is_empty() {
            return 0;
        }
        let mut hold = i32::MIN; // own a share
        let mut sold = 0;        // sold today (cooldown begins)
        let mut rest = 0;        // in cash, free to buy
        for &price in prices {
            let prev_hold = hold;
            let prev_sold = sold;
            let prev_rest = rest;

            hold = prev_hold.max(prev_rest - price); // buy only from rest
            sold = prev_hold + price;                // sell
            rest = prev_rest.max(prev_sold);         // stay or exit cooldown
        }
        sold.max(rest)
    }
}
```

### Complexity

| | Time | Space |
| --- | --- | --- |
| 3-state DP | `O(n)` | `O(1)` |

---

## 7. Problem 36 — Best Time to Buy and Sell With Transaction Fee (LC 714)

Unlimited transactions, but each **completed** transaction costs a fixed `fee`. Charge the fee once per transaction — by convention, subtract it at the **sell** edge. This is the LC 122 holding DP with a fee on the sell transition.

### Recurrence

```
cash[i] = max( cash[i-1], hold[i-1] + prices[i] - fee )  // sell, pay fee
hold[i] = max( hold[i-1], cash[i-1] - prices[i] )        // buy
```

### Rust

```rust
struct StockWithFee;

impl StockWithFee {
    // O(n) time, O(1) space.
    fn max_profit(prices: &[i32], fee: i32) -> i32 {
        let mut cash = 0;         // not holding
        let mut hold = i32::MIN;  // holding a share
        for &price in prices {
            let prev_cash = cash;
            cash = cash.max(hold + price - fee); // sell, charge fee
            hold = hold.max(prev_cash - price);  // buy
        }
        cash
    }
}
```

### Complexity

| | Time | Space |
| --- | --- | --- |
| Holding DP | `O(n)` | `O(1)` |

---

## 8. The General K-Transaction Template (and the specializations)

Everything above is the one template from Section 1:

```rust
// dp[day][cap][holding]
dp[day][cap][0] = dp[day + 1][cap][0].max(
    dp[day + 1][cap][1] - prices[day]);    // rest / buy
dp[day][cap][1] = dp[day + 1][cap][1].max(
    dp[day + 1][cap - 1][0] + prices[day]); // rest / sell
```

Explicit specializations:

- **Problem 31 (K = 1).** Fix `cap` at 1. The `cap` axis has one useful slice, the recurrence collapses to the four-scalar / running-min form. `buy1`/`sell1` only.
- **Problem 32 (K = ∞).** The `cap - 1` term never falls below a binding limit, so the `cap` dimension is irrelevant — drop it entirely. Left with `cash`/`hold`, which is precisely the greedy "sum positive deltas" in DP clothing.
- **Problem 33 (K = 2).** Fix `cap` at 2. Two buy/sell pairs → the four scalars `buy1, sell1, buy2, sell2`.
- **Problem 34 (K = k).** The template verbatim, with the `K >= n/2` shortcut delegating to LC 122.
- **Problem 35 (cooldown).** `K = ∞` plus an extra edge: buying is only allowed from a `rest` state reached one day after a `sold` state.
- **Problem 36 (fee).** `K = ∞` with `- fee` added on the sell edge.

---

## 9. Recognition Signals

| If the problem says... | It is... | Go-to approach |
| --- | --- | --- |
| "at most one transaction" | LC 121, `K = 1` | running min + max profit, `O(1)` |
| "as many transactions as you like" | LC 122, `K = ∞` | greedy positive deltas, `O(1)` |
| "at most two transactions" | LC 123, `K = 2` | four scalars `buy1/sell1/buy2/sell2` |
| "at most k transactions" | LC 188, general | 3D DP → 1D buy/sell arrays; `K ≥ n/2` shortcut |
| "cannot buy the day after selling" / "cooldown" | LC 309 | 3-state DP (`hold/sold/rest`) |
| "fee per transaction" | LC 714 | holding DP, subtract fee on sell |
| "can hold at most one share at a time" | any of the above | confirms the two-state `holding ∈ {0,1}` machine |
| "buy then sell, not simultaneously" | any of the above | the buy→sell edge ordering |

---

## 10. Summary

- Every stock problem is `dp[day][transactionsLeft][holding]` with `holding ∈ {0,1}`. Learn the machine, derive the variant.
- **Two universal edges:** buy (`cash → hold`, pay price) and sell (`hold → cash`, receive price, consume a transaction). Resting is always available.
- **K = 1 / K = 2** collapse to a fixed number of scalars (`buy/sell` pairs) → `O(1)` space.
- **K = ∞** drops the transaction dimension entirely → the greedy positive-delta sum.
- **General K** is the full 3D table; optimize to `O(k)` space, and short-circuit to greedy when `K ≥ n/2`.
- **Cooldown** adds a forced-rest edge after selling; **fee** subtracts a constant on the sell edge. Neither changes the core machine.
- Rust idioms used throughout: `Vec<Vec<Vec<i32>>>` / `Vec<Vec<i32>>` tables, `vec![]` macro with `-1` for memo init, `.max()` for transitions, guarding holding state with `i32::MIN` as the "impossible" sentinel.

| Problem | LC | Constraint | Best space |
| --- | --- | --- | --- |
| Stock I | 121 | at most 1 | `O(1)` |
| Stock II | 122 | unlimited | `O(1)` |
| Stock III | 123 | at most 2 | `O(1)` |
| Stock IV | 188 | at most K | `O(k)` |
| Cooldown | 309 | unlimited + cooldown | `O(1)` |
| Fee | 714 | unlimited + fee | `O(1)` |

---

> **Last Updated:** 2026-06-26
