> **Topic:** [Dynamic Programming](../README.md) · **Pattern 1 of 8**

# 1D DP (Fibonacci Style)

The **1D DP** pattern (often called *Fibonacci-style* DP) is the entry point into dynamic programming. It covers problems where the answer for a position `i` depends on a **constant number of previous positions** (usually `i-1` and `i-2`, sometimes up to `i-k`). Because the recurrence only looks "back" a fixed distance, the state collapses into a single index, and the space can frequently be reduced to a few rolling variables.

The mental model is always the same:

1. **Identify the state** — what does `dp[i]` *mean*? It is almost always "the answer considering elements up to (and including / ending at) index `i`."
2. **Write the recurrence** — express `dp[i]` in terms of earlier `dp[...]` values.
3. **Nail the base cases** — the smallest inputs that have no sub-problem.
4. **Optimize space** — if `dp[i]` only needs `dp[i-1]` and `dp[i-2]`, replace the array with two variables.

We walk through the canonical evolution **recursion → memoization → tabulation → space-optimized** on the first problem, then apply the same skeleton to the rest.

---

## 1. Climbing Stairs (LC 70)

> You are climbing a staircase with `n` steps. Each time you can climb **1** or **2** steps. In how many distinct ways can you reach the top?

### State, recurrence, base cases

| Element | Definition |
|---|---|
| **State** | `dp[i]` = number of distinct ways to reach step `i` |
| **Recurrence** | `dp[i] = dp[i-1] + dp[i-2]` (last move was either a 1-step or a 2-step) |
| **Base cases** | `dp[0] = 1` (one way: stand still), `dp[1] = 1` |

This is literally the Fibonacci sequence shifted by one — hence the pattern name.

### Step 1 — Pure recursion (exponential, for intuition only)

```rust
struct Solution;

impl Solution {
    pub fn climb_stairs(n: i32) -> i32 {
        if n <= 1 {
            return 1;
        }
        Solution::climb_stairs(n - 1) + Solution::climb_stairs(n - 2)
    }
}
```

Time `O(2^n)`, because the same sub-problems are recomputed repeatedly.

### Step 2 — Memoization (top-down)

We cache results in `Vec<i32> memo` initialized with `-1` (a sentinel meaning "not yet computed").

```rust
struct Solution;

impl Solution {
    pub fn climb_stairs(n: i32) -> i32 {
        let mut memo = vec![-1; (n + 1) as usize];
        Self::solve(n, &mut memo)
    }

    fn solve(n: i32, memo: &mut Vec<i32>) -> i32 {
        if n <= 1 {
            return 1;
        }
        let idx = n as usize;
        if memo[idx] != -1 {
            return memo[idx];
        }
        memo[idx] = Self::solve(n - 1, memo) + Self::solve(n - 2, memo);
        memo[idx]
    }
}
```

Time `O(n)`, Space `O(n)` for the memo plus `O(n)` recursion stack.

### Step 3 — Tabulation (bottom-up)

```rust
struct Solution;

impl Solution {
    pub fn climb_stairs(n: i32) -> i32 {
        if n <= 1 {
            return 1;
        }
        let n = n as usize;
        let mut dp = vec![0; n + 1];
        dp[0] = 1;
        dp[1] = 1;
        for i in 2..=n {
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        dp[n]
    }
}
```

### Step 4 — Space-optimized O(1)

Since `dp[i]` only depends on the previous two values, keep two rolling variables.

```rust
struct Solution;

impl Solution {
    pub fn climb_stairs(n: i32) -> i32 {
        if n <= 1 {
            return 1;
        }
        let mut prev2 = 1; // dp[i-2]
        let mut prev1 = 1; // dp[i-1]
        for _ in 2..=n {
            let curr = prev1 + prev2;
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Dry run (`n = 5`)

| i | prev2 (dp[i-2]) | prev1 (dp[i-1]) | curr = prev1 + prev2 |
|---|---|---|---|
| 2 | 1 | 1 | 2 |
| 3 | 1 | 2 | 3 |
| 4 | 2 | 3 | 5 |
| 5 | 3 | 5 | 8 |

Result: **8** distinct ways.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Recursion | `O(2^n)` | `O(n)` stack |
| Memoization | `O(n)` | `O(n)` + stack |
| Tabulation | `O(n)` | `O(n)` |
| Space-optimized | `O(n)` | `O(1)` |

---

## 2. Frog Jump (Minimum Cost)

> A frog starts at stone `0` and wants to reach stone `n-1`. From stone `i` it can jump to `i+1` or `i+2`. The cost of a jump is `abs(height[i] - height[j])`. Minimize total cost to reach the last stone.

### State, recurrence, base cases

| Element | Definition |
|---|---|
| **State** | `dp[i]` = minimum cost to reach stone `i` from stone `0` |
| **Recurrence** | `dp[i] = min(dp[i-1] + |h[i]-h[i-1]|, dp[i-2] + |h[i]-h[i-2]|)` |
| **Base cases** | `dp[0] = 0`; `dp[1] = |h[1]-h[0]|` |

### Tabulation

```rust
struct Solution;

impl Solution {
    pub fn frog_jump(height: &[i32]) -> i32 {
        let n = height.len();
        if n == 1 {
            return 0;
        }
        let mut dp = vec![0; n];
        dp[0] = 0;
        dp[1] = (height[1] - height[0]).abs();
        for i in 2..n {
            let one_step = dp[i - 1] + (height[i] - height[i - 1]).abs();
            let two_step = dp[i - 2] + (height[i] - height[i - 2]).abs();
            dp[i] = one_step.min(two_step);
        }
        dp[n - 1]
    }
}
```

### Space-optimized O(1)

```rust
struct Solution;

impl Solution {
    pub fn frog_jump(height: &[i32]) -> i32 {
        let n = height.len();
        if n == 1 {
            return 0;
        }
        let mut prev2 = 0; // dp[i-2]
        let mut prev1 = (height[1] - height[0]).abs(); // dp[i-1]
        for i in 2..n {
            let one_step = prev1 + (height[i] - height[i - 1]).abs();
            let two_step = prev2 + (height[i] - height[i - 2]).abs();
            let curr = one_step.min(two_step);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Dry run (`height = [10, 30, 40, 20]`)

| i | from i-1 (cost) | from i-2 (cost) | dp[i] |
|---|---|---|---|
| 0 | — | — | 0 |
| 1 | dp[0] + \|30-10\| = 20 | — | 20 |
| 2 | dp[1] + \|40-30\| = 30 | dp[0] + \|40-10\| = 30 | 30 |
| 3 | dp[2] + \|20-40\| = 50 | dp[1] + \|20-30\| = 30 | **30** |

Minimum cost to reach the last stone: **30**.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | `O(n)` | `O(n)` |
| Space-optimized | `O(n)` | `O(1)` |

---

## 3. Frog Jump with K Distances

> Same as above, but from stone `i` the frog can jump to any of `i+1, i+2, ..., i+k`. Minimize the total cost.

The recurrence now has an **inner loop over the `k` possible jumps**, so space optimization to `O(1)` is no longer possible — we genuinely need to look back up to `k` positions.

### State, recurrence, base cases

| Element | Definition |
|---|---|
| **State** | `dp[i]` = minimum cost to reach stone `i` |
| **Recurrence** | `dp[i] = min over j in [1..k] of (dp[i-j] + |h[i]-h[i-j]|)` for valid `i-j >= 0` |
| **Base case** | `dp[0] = 0` |

### Tabulation

```rust
struct Solution;

impl Solution {
    pub fn frog_jump_k(height: &[i32], k: usize) -> i32 {
        let n = height.len();
        let mut dp = vec![0; n];
        dp[0] = 0;
        for i in 1..n {
            let mut best = i32::MAX;
            for j in 1..=k {
                if i >= j {
                    let cost = dp[i - j] + (height[i] - height[i - j]).abs();
                    best = best.min(cost);
                }
            }
            dp[i] = best;
        }
        dp[n - 1]
    }
}
```

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | `O(n * k)` | `O(n)` |

> **Note:** A naive `O(1)` two-variable trick fails here because the recurrence reaches back up to `k` slots. You could keep a sliding window of the last `k` `dp` values (a `VecDeque`), but the `O(n)` array is the clean, correct baseline.

---

## 4. House Robber (LC 198)

> Houses are in a row, each holding `nums[i]` money. You cannot rob two **adjacent** houses. Maximize the money robbed.

This is the classic **pick / not-pick** recurrence.

### State, recurrence, base cases

| Element | Definition |
|---|---|
| **State** | `dp[i]` = maximum money robbing from houses `0..i` |
| **Recurrence** | `dp[i] = max(nums[i] + dp[i-2], dp[i-1])` (pick `i` and skip `i-1`, or skip `i`) |
| **Base cases** | `dp[0] = nums[0]`; `dp[1] = max(nums[0], nums[1])` |

### Memoization (top-down)

```rust
struct Solution;

impl Solution {
    pub fn rob(nums: &[i32]) -> i32 {
        let n = nums.len();
        let mut memo = vec![-1; n];
        Self::solve(nums, n as i32 - 1, &mut memo)
    }

    fn solve(nums: &[i32], i: i32, memo: &mut Vec<i32>) -> i32 {
        if i < 0 {
            return 0;
        }
        let idx = i as usize;
        if i == 0 {
            return nums[0];
        }
        if memo[idx] != -1 {
            return memo[idx];
        }
        let pick = nums[idx] + Self::solve(nums, i - 2, memo);
        let not_pick = Self::solve(nums, i - 1, memo);
        memo[idx] = pick.max(not_pick);
        memo[idx]
    }
}
```

### Tabulation

```rust
struct Solution;

impl Solution {
    pub fn rob(nums: &[i32]) -> i32 {
        let n = nums.len();
        if n == 1 {
            return nums[0];
        }
        let mut dp = vec![0; n];
        dp[0] = nums[0];
        dp[1] = nums[0].max(nums[1]);
        for i in 2..n {
            let pick = nums[i] + dp[i - 2];
            let not_pick = dp[i - 1];
            dp[i] = pick.max(not_pick);
        }
        dp[n - 1]
    }
}
```

### Space-optimized O(1)

```rust
struct Solution;

impl Solution {
    pub fn rob(nums: &[i32]) -> i32 {
        let mut prev2 = 0; // dp[i-2]
        let mut prev1 = 0; // dp[i-1]
        for &num in nums {
            let pick = num + prev2;
            let curr = pick.max(prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Dry run (`nums = [2, 7, 9, 3, 1]`)

| i | nums[i] | prev2 | prev1 | pick = nums[i]+prev2 | curr = max(pick, prev1) |
|---|---|---|---|---|---|
| 0 | 2 | 0 | 0 | 2 | 2 |
| 1 | 7 | 0 | 2 | 7 | 7 |
| 2 | 9 | 2 | 7 | 11 | 11 |
| 3 | 3 | 7 | 11 | 10 | 11 |
| 4 | 1 | 11 | 11 | 12 | **12** |

Maximum robbed: **12** (rob houses 0, 2, 4 → 2 + 9 + 1 = 12).

### Complexity

| Approach | Time | Space |
|---|---|---|
| Memoization | `O(n)` | `O(n)` + stack |
| Tabulation | `O(n)` | `O(n)` |
| Space-optimized | `O(n)` | `O(1)` |

---

## 5. House Robber II (LC 213, Circular)

> Same as House Robber, but the houses are arranged in a **circle**: the first and last houses are adjacent, so they cannot both be robbed.

### Key insight

If we cannot rob both the first and last house, then any optimal solution falls into one of two cases:

- **Exclude the last house** → run linear robber on `nums[0 .. n-2]`.
- **Exclude the first house** → run linear robber on `nums[1 .. n-1]`.

The answer is the maximum of the two. (Handle `n == 1` separately.)

```rust
struct Solution;

impl Solution {
    pub fn rob(nums: &[i32]) -> i32 {
        let n = nums.len();
        if n == 1 {
            return nums[0];
        }
        let exclude_last = Self::rob_linear(&nums[..n - 1]);
        let exclude_first = Self::rob_linear(&nums[1..]);
        exclude_last.max(exclude_first)
    }

    // Linear House Robber over the inclusive range [start, end].
    fn rob_linear(nums: &[i32]) -> i32 {
        let mut prev2 = 0;
        let mut prev1 = 0;
        for &num in nums {
            let pick = num + prev2;
            let curr = pick.max(prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Complexity

| Approach | Time | Space |
|---|---|---|
| Two linear passes | `O(n)` | `O(1)` |

---

## 6. Maximum Sum of Non-Adjacent Elements

> Given an array, pick a subset of elements such that **no two are adjacent**, maximizing their sum.

This is **exactly House Robber** — the "no two adjacent houses" constraint is identical to "no two adjacent elements." The recurrence, base cases, and space optimization are word-for-word the same.

| Element | Definition |
|---|---|
| **State** | `dp[i]` = maximum non-adjacent sum using elements `0..i` |
| **Recurrence** | `dp[i] = max(nums[i] + dp[i-2], dp[i-1])` |
| **Base cases** | `dp[0] = nums[0]`; `dp[1] = max(nums[0], nums[1])` |

```rust
struct Solution;

impl Solution {
    pub fn max_non_adjacent_sum(nums: &[i32]) -> i32 {
        let mut prev2 = 0; // best sum ending at or before i-2
        let mut prev1 = 0; // best sum ending at or before i-1
        for &num in nums {
            let pick = num + prev2;
            let curr = pick.max(prev1);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

> **Equivalence note:** If you can solve House Robber, you have already solved this problem. Recognizing such equivalences is the single biggest accelerator in DP — many problems are the same recurrence wearing a different costume. (If negative values are allowed and the empty subset is permitted, this code already handles it correctly, since `prev1`/`prev2` start at 0 and `pick` competes against `prev1`.)

---

## 7. Min/Max Running Cost (Ninja's Training — 1D Variant)

The full **Ninja's Training** problem (LC-style DP) is 2D: each day the ninja picks one of several activities, with a constraint that the same activity cannot be chosen on two consecutive days. That belongs to the [2D DP (Grids)](./2D%20DP%20(Grids).md) pattern, where the state carries both the day and the *last activity chosen*.

Here we look at the **1D precursor**: a single sequence of costs where, on each step, you simply accumulate a **running minimum (or maximum) cost** — the cost of reaching step `i` depends only on the best cost of reaching the immediately preceding step plus the current step's cost. There is no per-state second dimension yet.

### State, recurrence, base cases

| Element | Definition |
|---|---|
| **State** | `dp[i]` = minimum running cost to complete steps `0..i` |
| **Recurrence (min)** | `dp[i] = dp[i-1] + cost[i]` (with `dp[i] = min(dp[i-1], dp[i-2]) + cost[i]` if step skipping is allowed) |
| **Base case** | `dp[0] = cost[0]` |

### Simple 1D running min cost (skipping one step allowed)

This mirrors the structure of **Min Cost Climbing Stairs**: at each step you may have arrived from one of the previous one or two steps, and you want the cheapest arrival.

```rust
struct Solution;

impl Solution {
    // Minimum running cost; from step i you may arrive from i-1 or i-2.
    pub fn min_running_cost(cost: &[i32]) -> i32 {
        let n = cost.len();
        if n == 0 {
            return 0;
        }
        if n == 1 {
            return cost[0];
        }
        let mut prev2 = cost[0];
        let mut prev1 = cost[1];
        for i in 2..n {
            let curr = cost[i] + prev1.min(prev2);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Max variant

The maximum-cost (best reward) version is identical with `max` swapped in:

```rust
struct Solution;

impl Solution {
    pub fn max_running_cost(cost: &[i32]) -> i32 {
        let n = cost.len();
        if n == 0 {
            return 0;
        }
        if n == 1 {
            return cost[0];
        }
        let mut prev2 = cost[0];
        let mut prev1 = cost[1];
        for i in 2..n {
            let curr = cost[i] + prev1.max(prev2);
            prev2 = prev1;
            prev1 = curr;
        }
        prev1
    }
}
```

### Complexity

| Approach | Time | Space |
|---|---|---|
| Space-optimized | `O(n)` | `O(1)` |

> **Bridge to 2D:** The moment a constraint forces you to remember *which choice you made on the previous step* (e.g., "cannot repeat yesterday's activity"), the state grows a second dimension and the problem migrates to [2D DP (Grids)](./2D%20DP%20(Grids).md).

---

## 8. Recognition Signals

| Signal phrase in the problem | What it suggests |
|---|---|
| "climb 1 or 2 steps", "ways to reach" | Fibonacci-style count: `dp[i] = dp[i-1] + dp[i-2]` |
| "jump 1 or 2 / up to k steps", "cost of a jump" | Frog-jump min-cost DP (single-step → `O(1)`; k-step → inner loop, `O(n*k)`) |
| "cannot pick two adjacent", "no two consecutive" | House Robber / max non-adjacent sum: `dp[i] = max(nums[i]+dp[i-2], dp[i-1])` |
| "houses in a circle", "first and last are adjacent" | Run the linear version twice (exclude first OR exclude last), take the max |
| "minimum / maximum cost to reach the end" | Running min/max cost DP along a single sequence |
| "answer at i depends only on i-1 (and i-2)" | Collapse the array into 2 rolling variables → `O(1)` space |
| "depends on the previous *choice*, not just position" | You have left 1D — move to [2D DP (Grids)](./2D%20DP%20(Grids).md) |

---

## 9. Summary

- **1D DP** problems have a state indexed by a single position; the recurrence reaches back a **fixed, small** number of indices.
- Always progress mentally **recursion → memoization → tabulation → space optimization**, but in interviews you can often jump straight to tabulation or the `O(1)` two-variable form once you trust the recurrence.
- **Memoization** uses `Vec<i32> memo` initialized with `-1` as the "not computed" sentinel.
- When `dp[i]` depends only on `dp[i-1]` and `dp[i-2]`, replace the array with `prev1` / `prev2` for **`O(1)` space**.
- The **k-distance frog** breaks the `O(1)` trick because it reaches back up to `k` slots, giving `O(n*k)` time.
- **House Robber**, **Max Non-Adjacent Sum**, and **House Robber II** are the same pick/not-pick recurrence; the circular variant just runs the linear solver twice.
- The 1D running-cost problem is a **precursor to the 2D Ninja's Training**: once a constraint forces remembering the previous *choice*, add a dimension and move to [2D DP (Grids)](./2D%20DP%20(Grids).md).

> **Last Updated:** 2026-06-26
