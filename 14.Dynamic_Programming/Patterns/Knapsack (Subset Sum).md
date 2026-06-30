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

```java
public class SubsetSum {

    public boolean isSubsetSum(int[] arr, int target) {
        return solve(arr.length - 1, target, arr);
    }

    private boolean solve(int idx, int target, int[] arr) {
        if (target == 0) return true;          // empty subset works
        if (idx == 0) return arr[0] == target; // only first element left
        boolean notTake = solve(idx - 1, target, arr);
        boolean take = false;
        if (arr[idx] <= target) {
            take = solve(idx - 1, target - arr[idx], arr);
        }
        return take || notTake;
    }
}
```

Time `O(2^n)`, exponential — every element branches twice.

### 2b. Memoization (`int[][]` with `-1`)

We cache booleans as `int`: `-1` = unvisited, `0` = false, `1` = true. (You could also use `Boolean[][]` and test for `null`.)

```java
import java.util.Arrays;

public class SubsetSum {

    public boolean isSubsetSum(int[] arr, int target) {
        int n = arr.length;
        int[][] dp = new int[n][target + 1];
        for (int[] row : dp) Arrays.fill(row, -1);
        return solve(n - 1, target, arr, dp);
    }

    private boolean solve(int idx, int target, int[] arr, int[][] dp) {
        if (target == 0) return true;
        if (idx == 0) return arr[0] == target;
        if (dp[idx][target] != -1) return dp[idx][target] == 1;

        boolean notTake = solve(idx - 1, target, arr, dp);
        boolean take = false;
        if (arr[idx] <= target) {
            take = solve(idx - 1, target - arr[idx], arr, dp);
        }
        boolean result = take || notTake;
        dp[idx][target] = result ? 1 : 0;
        return result;
    }
}
```

Time `O(n * target)`, Space `O(n * target)` + recursion stack `O(n)`.

### 2c. Tabulation

```java
public class SubsetSum {

    public boolean isSubsetSum(int[] arr, int target) {
        int n = arr.length;
        boolean[][] dp = new boolean[n][target + 1];

        for (int i = 0; i < n; i++) dp[i][0] = true; // sum 0 always possible
        if (arr[0] <= target) dp[0][arr[0]] = true;

        for (int i = 1; i < n; i++) {
            for (int t = 1; t <= target; t++) {
                boolean notTake = dp[i - 1][t];
                boolean take = false;
                if (arr[i] <= t) {
                    take = dp[i - 1][t - arr[i]];
                }
                dp[i][t] = take || notTake;
            }
        }
        return dp[n - 1][target];
    }
}
```

### 2d. Space optimization to 1D

Each row depends only on the previous row, so we keep one array. Because this is **0/1** (each item once), we iterate capacity **downward** to avoid reusing `arr[i]` within the same pass.

```java
public class SubsetSum {

    public boolean isSubsetSum(int[] arr, int target) {
        int n = arr.length;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true; // empty subset

        for (int i = 0; i < n; i++) {
            for (int t = target; t >= arr[i]; t--) { // DOWNWARD: 0/1 semantics
                dp[t] = dp[t] || dp[t - arr[i]];
            }
        }
        return dp[target];
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

```java
public class PartitionEqualSubsetSum {

    public boolean canPartition(int[] nums) {
        int total = 0;
        for (int x : nums) total += x;
        if ((total & 1) == 1) return false; // odd total can't split evenly

        int target = total / 2;
        boolean[] dp = new boolean[target + 1];
        dp[0] = true;

        for (int num : nums) {
            for (int t = target; t >= num; t--) { // 0/1 → downward
                dp[t] = dp[t] || dp[t - num];
            }
        }
        return dp[target];
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

- `dp[0][0] = (arr[0] == 0) ? 2 : 1`
- `if (arr[0] != 0 && arr[0] <= k) dp[0][arr[0]] = 1`

```java
public class CountSubsetsWithSumK {

    private static final int MOD = 1_000_000_007;

    public int perfectSum(int[] arr, int k) {
        int n = arr.length;
        int[][] dp = new int[n][k + 1];

        // Base row handling zeros explicitly.
        if (arr[0] == 0) {
            dp[0][0] = 2;            // skip the zero OR take the zero
        } else {
            dp[0][0] = 1;            // only the empty subset
            if (arr[0] <= k) dp[0][arr[0]] = 1;
        }

        for (int i = 1; i < n; i++) {
            for (int t = 0; t <= k; t++) {
                int notTake = dp[i - 1][t];
                int take = 0;
                if (arr[i] <= t) {
                    take = dp[i - 1][t - arr[i]];
                }
                dp[i][t] = (take + notTake) % MOD;
            }
        }
        return dp[n - 1][k];
    }
}
```

1D version (downward, 0/1). When zeros are *not* present the simpler `dp[0] = 1` base works; with zeros prefer the 2D base above, or pre-handle the count of zeros separately and multiply by `2^(#zeros)`.

```java
public class CountSubsetsWithSumK {

    private static final int MOD = 1_000_000_007;

    public int perfectSum(int[] arr, int k) {
        int[] dp = new int[k + 1];
        dp[0] = 1; // assumes the explicit-zero handling above for arrays containing 0
        for (int num : arr) {
            for (int t = k; t >= num; t--) { // 0/1 → downward
                dp[t] = (dp[t] + dp[t - num]) % MOD;
            }
        }
        return dp[k];
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
- **Base case:** `f(0, cap) = (weight[0] <= cap) ? value[0] : 0`.

### 5a. Pure recursion

```java
public class Knapsack01 {

    public int knapsack(int W, int[] weight, int[] value) {
        return solve(weight.length - 1, W, weight, value);
    }

    private int solve(int idx, int cap, int[] weight, int[] value) {
        if (idx == 0) {
            return weight[0] <= cap ? value[0] : 0;
        }
        int notTake = solve(idx - 1, cap, weight, value);
        int take = Integer.MIN_VALUE;
        if (weight[idx] <= cap) {
            take = value[idx] + solve(idx - 1, cap - weight[idx], weight, value);
        }
        return Math.max(take, notTake);
    }
}
```

### 5b. Memoization (`int[][]`, `Arrays.fill(-1)`)

```java
import java.util.Arrays;

public class Knapsack01 {

    public int knapsack(int W, int[] weight, int[] value) {
        int n = weight.length;
        int[][] dp = new int[n][W + 1];
        for (int[] row : dp) Arrays.fill(row, -1);
        return solve(n - 1, W, weight, value, dp);
    }

    private int solve(int idx, int cap, int[] weight, int[] value, int[][] dp) {
        if (idx == 0) {
            return weight[0] <= cap ? value[0] : 0;
        }
        if (dp[idx][cap] != -1) return dp[idx][cap];

        int notTake = solve(idx - 1, cap, weight, value, dp);
        int take = Integer.MIN_VALUE;
        if (weight[idx] <= cap) {
            take = value[idx] + solve(idx - 1, cap - weight[idx], weight, value, dp);
        }
        return dp[idx][cap] = Math.max(take, notTake);
    }
}
```

### 5c. Tabulation

```java
public class Knapsack01 {

    public int knapsack(int W, int[] weight, int[] value) {
        int n = weight.length;
        int[][] dp = new int[n][W + 1];

        for (int cap = weight[0]; cap <= W; cap++) {
            dp[0][cap] = value[0];
        }

        for (int i = 1; i < n; i++) {
            for (int cap = 0; cap <= W; cap++) {
                int notTake = dp[i - 1][cap];
                int take = Integer.MIN_VALUE;
                if (weight[i] <= cap) {
                    take = value[i] + dp[i - 1][cap - weight[i]];
                }
                dp[i][cap] = Math.max(take, notTake);
            }
        }
        return dp[n - 1][W];
    }
}
```

### 5d. Space optimization to 1D — iterate capacity DOWNWARD

```java
public class Knapsack01 {

    public int knapsack(int W, int[] weight, int[] value) {
        int n = weight.length;
        int[] dp = new int[W + 1];

        for (int cap = weight[0]; cap <= W; cap++) {
            dp[cap] = value[0];
        }

        for (int i = 1; i < n; i++) {
            for (int cap = W; cap >= weight[i]; cap--) { // DOWNWARD
                int take = value[i] + dp[cap - weight[i]];
                dp[cap] = Math.max(dp[cap], take);
            }
        }
        return dp[W];
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
- **Base case:** `f(0, rem) = (rem % coins[0] == 0) ? rem / coins[0] : INF`.

### 6a. Memoization

```java
import java.util.Arrays;

public class CoinChangeMin {

    private static final int INF = (int) 1e9;

    public int coinChange(int[] coins, int amount) {
        int n = coins.length;
        int[][] dp = new int[n][amount + 1];
        for (int[] row : dp) Arrays.fill(row, -1);
        int ans = solve(n - 1, amount, coins, dp);
        return ans >= INF ? -1 : ans;
    }

    private int solve(int idx, int rem, int[] coins, int[][] dp) {
        if (idx == 0) {
            return (rem % coins[0] == 0) ? rem / coins[0] : INF;
        }
        if (dp[idx][rem] != -1) return dp[idx][rem];

        int notTake = solve(idx - 1, rem, coins, dp);
        int take = INF;
        if (coins[idx] <= rem) {
            take = 1 + solve(idx, rem - coins[idx], coins, dp); // STAY on idx
        }
        return dp[idx][rem] = Math.min(take, notTake);
    }
}
```

### 6b. Tabulation, 1D — iterate capacity UPWARD

```java
import java.util.Arrays;

public class CoinChangeMin {

    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1); // sentinel "infinity" (can't exceed amount coins)
        dp[0] = 0;

        for (int coin : coins) {
            for (int rem = coin; rem <= amount; rem++) { // UPWARD: unbounded reuse
                dp[rem] = Math.min(dp[rem], 1 + dp[rem - coin]);
            }
        }
        return dp[amount] > amount ? -1 : dp[amount];
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
- **Base case:** `f(0, rem) = (rem % coins[0] == 0) ? 1 : 0`.

To count **combinations** (not permutations), the 1D form puts **coins on the outer loop** and **amount on the inner loop**. This ordering ensures each combination is counted once because coins are introduced in a fixed order.

### 7a. Memoization

```java
import java.util.Arrays;

public class CoinChangeII {

    public int change(int amount, int[] coins) {
        int n = coins.length;
        long[][] dp = new long[n][amount + 1];
        for (long[] row : dp) Arrays.fill(row, -1);
        return (int) solve(n - 1, amount, coins, dp);
    }

    private long solve(int idx, int rem, int[] coins, long[][] dp) {
        if (idx == 0) {
            return (rem % coins[0] == 0) ? 1 : 0;
        }
        if (dp[idx][rem] != -1) return dp[idx][rem];

        long notTake = solve(idx - 1, rem, coins, dp);
        long take = 0;
        if (coins[idx] <= rem) {
            take = solve(idx, rem - coins[idx], coins, dp); // STAY on idx
        }
        return dp[idx][rem] = take + notTake;
    }
}
```

### 7b. Tabulation, 1D — outer loop coins, inner loop amount (UPWARD)

```java
public class CoinChangeII {

    public int change(int amount, int[] coins) {
        long[] dp = new long[amount + 1];
        dp[0] = 1; // one way to make 0: pick nothing

        for (int coin : coins) {              // OUTER: coins
            for (int rem = coin; rem <= amount; rem++) { // INNER: amount, upward
                dp[rem] += dp[rem - coin];
            }
        }
        return (int) dp[amount];
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
- If `Math.abs(target) > sum` → impossible → answer `0`.
- The derived subset target must be non-negative (it is, given the guards above, since `(sum + target) >= 0`).
- Zeros are still handled by the count-subset base case (each zero doubles the count).

```java
public class TargetSum {

    public int findTargetSumWays(int[] nums, int target) {
        int sum = 0;
        for (int x : nums) sum += x;

        // S1 = (sum + target) / 2 must be a non-negative integer.
        if (Math.abs(target) > sum) return 0;
        if (((sum + target) & 1) == 1) return 0;
        int s1 = (sum + target) / 2;

        return countSubsets(nums, s1);
    }

    private int countSubsets(int[] arr, int k) {
        int n = arr.length;
        int[][] dp = new int[n][k + 1];

        if (arr[0] == 0) {
            dp[0][0] = 2;                 // take or skip the zero
        } else {
            dp[0][0] = 1;
            if (arr[0] <= k) dp[0][arr[0]] = 1;
        }

        for (int i = 1; i < n; i++) {
            for (int t = 0; t <= k; t++) {
                int notTake = dp[i - 1][t];
                int take = (arr[i] <= t) ? dp[i - 1][t - arr[i]] : 0;
                dp[i][t] = take + notTake;
            }
        }
        return dp[n - 1][k];
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

```java
public class MinSubsetSumDifference {

    public int minDifference(int[] arr) {
        int total = 0;
        for (int x : arr) total += x;

        // dp[s] = is sum s reachable by some subset?
        boolean[] dp = new boolean[total + 1];
        dp[0] = true;
        for (int num : arr) {
            for (int s = total; s >= num; s--) { // 0/1 → downward
                dp[s] = dp[s] || dp[s - num];
            }
        }

        int best = Integer.MAX_VALUE;
        for (int s = 0; s <= total / 2; s++) {
            if (dp[s]) {
                int diff = total - 2 * s; // = |s - (total - s)|, s <= total/2
                best = Math.min(best, diff);
            }
        }
        return best;
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
- **Problem 18 (0/1 Knapsack)** is the template: master recursion → memo (`Arrays.fill(dp, -1)`) → tabulation → 1D. The other 0/1 problems are transformations:
  - **Subset Sum** → boolean `OR`.
  - **Partition Equal (416)** → subset sum with `target = total/2`, return false on odd total.
  - **Count Subsets / Target Sum (494)** → `+` counting; Target Sum reduces via `S1 = (sum+target)/2`.
  - **Min Subset Diff** → reachable sums + sweep `min(total - 2s)`.
- **Unbounded** problems (Coin Change 322 / II 518) keep the index on **take** and iterate the 1D capacity loop **upward**; Coin Change II additionally loops **coins outer** to count combinations.
- The **loop-direction rule** (down for 0/1, up for unbounded) is the highest-yield thing to memorize — it is the difference between a correct and a subtly broken solution.
- Use `Math.min` / `Math.max` for optimization, `Integer.compare` rather than subtraction for comparators, and `long` accumulators when counts can overflow `int` (Coin Change II).

> **Last Updated:** 2026-06-26
