> **Topic:** [Dynamic Programming](../README.md) · **Pattern 5 of 8**

# Stock Buy Sell

The "Stock Buy Sell" family is a collection of problems that look superficially different — one transaction, infinite transactions, at most two, at most K, with a cooldown, with a fee — but every one of them is a specialization of a **single underlying state machine**. Once you internalize that machine, you stop memorizing six tricks and instead derive each variant by *constraining* one general recurrence. This page builds that unified model first, then solves all six problems with full, compilable C++.

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

### C++

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockI {
public:
    // O(n) time, O(1) space — the K = 1 specialization.
    int maxProfit(vector<int>& prices) {
        int minPrice = INT_MAX;
        int best = 0;
        for (auto& price : prices) {
            if (price < minPrice) {
                minPrice = price;
            } else if (price - minPrice > best) {
                best = price - minPrice;
            }
        }
        return best;
    }

    // The same problem via the unified holding-state DP with K = 1, O(1) space.
    // cash0 = best profit holding nothing, having used 0 buys.
    // hold1 = best profit holding a share (the one allowed buy).
    // cash1 = best profit holding nothing after the one allowed sell.
    int maxProfitDP(vector<int>& prices) {
        int hold1 = INT_MIN; // bought once, not yet sold
        int cash1 = 0;       // sold the one share
        for (auto& price : prices) {
            hold1 = max(hold1, -price);        // buy today (from initial cash 0)
            cash1 = max(cash1, hold1 + price); // sell today
        }
        return cash1;
    }
};
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

### C++

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockII {
public:
    // Greedy: O(n) time, O(1) space.
    int maxProfit(vector<int>& prices) {
        int profit = 0;
        for (int i = 1; i < (int)prices.size(); i++) {
            int delta = prices[i] - prices[i - 1];
            if (delta > 0) {
                profit += delta;
            }
        }
        return profit;
    }

    // Holding-state DP (K = infinite => no transaction dimension), O(1) space.
    int maxProfitDP(vector<int>& prices) {
        int cash = 0;        // not holding
        int hold = INT_MIN;  // holding a share
        for (auto& price : prices) {
            int prevCash = cash;
            cash = max(cash, hold + price);       // sell today
            hold = max(hold, prevCash - price);   // buy today (reuse prior cash)
        }
        return cash;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIII_Memo {

    vector<vector<vector<int>>> memo;
    vector<int> prices;
    int n;

    int solve(int day, int cap, int holding) {
        if (day == n || cap == 0) {
            return 0;
        }
        if (memo[day][cap][holding] != -1) {
            return memo[day][cap][holding];
        }

        int rest = solve(day + 1, cap, holding);
        int action;
        if (holding == 0) {
            // buy: pay price, now holding, cap unchanged (we count txn at sell)
            action = solve(day + 1, cap, 1) - prices[day];
        } else {
            // sell: receive price, now in cash, consume one transaction
            action = solve(day + 1, cap - 1, 0) + prices[day];
        }

        int result = max(rest, action);
        memo[day][cap][holding] = result;
        return result;
    }

public:
    int maxProfit(vector<int>& prices) {
        this->prices = prices;
        this->n = prices.size();
        // dimensions: day (n), transactionsLeft (3: 0,1,2), holding (2)
        memo = vector<vector<vector<int>>>(n, vector<vector<int>>(3, vector<int>(2, -1)));
        return solve(0, 2, 0);
    }
};
```

### 4.2 Bottom-up tabulation (3D)

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIII_Tab {
public:
    int maxProfit(vector<int>& prices) {
        int n = prices.size();
        if (n == 0) {
            return 0;
        }
        // dp[day][cap][holding]; day goes n down to 0
        vector<vector<vector<int>>> dp(n + 1, vector<vector<int>>(3, vector<int>(2, 0)));
        // base: dp[n][*][*] = 0 (already zero); cap == 0 also yields 0.

        for (int day = n - 1; day >= 0; day--) {
            for (int cap = 1; cap <= 2; cap++) {
                // holding == 0 (can buy)
                dp[day][cap][0] = max(
                        dp[day + 1][cap][0],                 // rest
                        dp[day + 1][cap][1] - prices[day]);  // buy
                // holding == 1 (can sell)
                dp[day][cap][1] = max(
                        dp[day + 1][cap][1],                     // rest
                        dp[day + 1][cap - 1][0] + prices[day]);  // sell, consume txn
            }
        }
        return dp[0][2][0];
    }
};
```

### 4.3 Space-optimized (four scalars)

With `K = 2` the entire table collapses to four running values: cost/profit after the 1st buy, after the 1st sell, after the 2nd buy, after the 2nd sell.

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIII_Opt {
public:
    // O(n) time, O(1) space.
    int maxProfit(vector<int>& prices) {
        int buy1 = INT_MIN;  // after first buy
        int sell1 = 0;       // after first sell
        int buy2 = INT_MIN;  // after second buy
        int sell2 = 0;       // after second sell
        for (auto& price : prices) {
            buy1 = max(buy1, -price);
            sell1 = max(sell1, buy1 + price);
            buy2 = max(buy2, sell1 - price);
            sell2 = max(sell2, buy2 + price);
        }
        return sell2;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIV_Memo {

    vector<vector<vector<int>>> memo;
    vector<int> prices;
    int n;

    int solve(int day, int cap, int holding) {
        if (day == n || cap == 0) {
            return 0;
        }
        if (memo[day][cap][holding] != -1) {
            return memo[day][cap][holding];
        }

        int rest = solve(day + 1, cap, holding);
        int action;
        if (holding == 0) {
            action = solve(day + 1, cap, 1) - prices[day];        // buy
        } else {
            action = solve(day + 1, cap - 1, 0) + prices[day];    // sell, consume txn
        }

        int result = max(rest, action);
        memo[day][cap][holding] = result;
        return result;
    }

public:
    int maxProfit(int k, vector<int>& prices) {
        this->prices = prices;
        this->n = prices.size();
        if (n == 0 || k == 0) {
            return 0;
        }
        // dimensions: day (n), transactionsLeft (k+1), holding (2)
        memo = vector<vector<vector<int>>>(n, vector<vector<int>>(k + 1, vector<int>(2, -1)));
        return solve(0, k, 0);
    }
};
```

### 5.2 Bottom-up tabulation (3D)

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIV_Tab {
public:
    int maxProfit(int k, vector<int>& prices) {
        int n = prices.size();
        if (n == 0 || k == 0) {
            return 0;
        }
        vector<vector<vector<int>>> dp(n + 1, vector<vector<int>>(k + 1, vector<int>(2, 0))); // base dp[n][*][*] = 0

        for (int day = n - 1; day >= 0; day--) {
            for (int cap = 1; cap <= k; cap++) {
                dp[day][cap][0] = max(
                        dp[day + 1][cap][0],
                        dp[day + 1][cap][1] - prices[day]);
                dp[day][cap][1] = max(
                        dp[day + 1][cap][1],
                        dp[day + 1][cap - 1][0] + prices[day]);
            }
        }
        return dp[0][k][0];
    }
};
```

### 5.3 Space-optimized to 2D (rolling day) and 1D (buy/sell pairs)

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockIV_Opt {
public:
    // 2D: keep only the next day's plane -> O(k) space.
    int maxProfit2D(int k, vector<int>& prices) {
        int n = prices.size();
        if (n == 0 || k == 0) {
            return 0;
        }
        vector<vector<int>> next(k + 1, vector<int>(2, 0));
        for (int day = n - 1; day >= 0; day--) {
            vector<vector<int>> cur(k + 1, vector<int>(2, 0));
            for (int cap = 1; cap <= k; cap++) {
                cur[cap][0] = max(next[cap][0], next[cap][1] - prices[day]);
                cur[cap][1] = max(next[cap][1], next[cap - 1][0] + prices[day]);
            }
            next = cur;
        }
        return next[k][0];
    }

    // 1D: buy[t]/sell[t] arrays scanned over days, like the four-scalar StockIII
    // generalized to k pairs. O(k) space.
    int maxProfit1D(int k, vector<int>& prices) {
        if (prices.empty() || k == 0) {
            return 0;
        }
        vector<int> buy(k + 1, 0);
        vector<int> sell(k + 1, 0);
        fill(buy.begin(), buy.end(), INT_MIN);
        // sell[0] stays 0; buy[0] unused for transactions.
        for (auto& price : prices) {
            for (int t = 1; t <= k; t++) {
                buy[t] = max(buy[t], sell[t - 1] - price);
                sell[t] = max(sell[t], buy[t] + price);
            }
        }
        return sell[k];
    }
};
```

(The `#include <bits/stdc++.h>` covers all standard library needs for `StockIV_Opt::maxProfit1D`.)

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

### C++

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockCooldown {
public:
    // O(n) time, O(1) space.
    int maxProfit(vector<int>& prices) {
        if (prices.empty()) {
            return 0;
        }
        int hold = INT_MIN; // own a share
        int sold = 0;       // sold today (cooldown begins)
        int rest = 0;       // in cash, free to buy
        for (auto& price : prices) {
            int prevHold = hold;
            int prevSold = sold;
            int prevRest = rest;

            hold = max(prevHold, prevRest - price); // buy only from rest
            sold = prevHold + price;                // sell
            rest = max(prevRest, prevSold);         // stay or exit cooldown
        }
        return max(sold, rest);
    }
};
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

### C++

```cpp
#include <bits/stdc++.h>
using namespace std;

class StockWithFee {
public:
    // O(n) time, O(1) space.
    int maxProfit(vector<int>& prices, int fee) {
        int cash = 0;        // not holding
        int hold = INT_MIN;  // holding a share
        for (auto& price : prices) {
            int prevCash = cash;
            cash = max(cash, hold + price - fee); // sell, charge fee
            hold = max(hold, prevCash - price);   // buy
        }
        return cash;
    }
};
```

### Complexity

| | Time | Space |
| --- | --- | --- |
| Holding DP | `O(n)` | `O(1)` |

---

## 8. The General K-Transaction Template (and the specializations)

Everything above is the one template from Section 1:

```cpp
// dp[day][cap][holding]
dp[day][cap][0] = max(dp[day + 1][cap][0],
                      dp[day + 1][cap][1] - prices[day]);   // rest / buy
dp[day][cap][1] = max(dp[day + 1][cap][1],
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
- C++ standards used throughout: `vector<vector<vector<int>>>` / `vector<vector<int>>` tables, vector constructor with `-1` for memo init, `max` for transitions, guarding holding state with `INT_MIN` as the "impossible" sentinel.

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
