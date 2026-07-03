> **Topic:** [Dynamic Programming](../README.md) · **Design Data Structure Problems**

# DP Memoization Framework

Dynamic programming is, at its heart, **recursion + a cache**. Every top-down DP is a plain recursive function whose results are stored so that each distinct subproblem is solved exactly once. This file is a reusable toolkit: how to size and initialize memo tables, how to memoize non-integer / sparse states, how to mechanically convert a top-down solution into a bottom-up one, and how to reason about correctness, complexity, and stack depth.

If you internalize the recipes here, you can solve *any* of the 55 problems in this topic by writing the brute-force recursion first and then "switching on" the cache.

---

## 1. The Core Idea — Recursion With a Cache

A subproblem is uniquely identified by its **state** (the arguments that actually vary across recursive calls). Memoization stores the answer for each state the first time it is computed and returns the stored value on every subsequent visit.

```rust
use std::collections::HashMap;

// Skeleton of every top-down DP.
fn solve(s: State, memo: &mut HashMap<State, i32>) -> i32 {
    if is_base_case(s) { return base_value(s); }          // 1. base case
    if let Some(&cached) = memo.get(&s) { return cached; } // 2. cache hit -> return
    let ans = combine(                                      // 3. recurrence / transition
        solve(next1(s), memo),
        solve(next2(s), memo),
        // ...
    );
    memo.insert(s, ans);                                   // 4. cache the result
    ans
}
```

Two properties must hold for DP to apply:

| Property | Meaning |
|----------|---------|
| **Optimal substructure** | The optimal answer to a problem is built from optimal answers to its subproblems. |
| **Overlapping subproblems** | The same subproblem is reached via many different recursion paths — so caching pays off. |

If subproblems are *not* overlapping (e.g., pure divide-and-conquer like merge sort), memoization buys nothing.

---

## 2. Integer-Indexed Memo Tables (the common case)

When the state is a small tuple of bounded integers (`index`, `capacity`, `i`, `j`, ...), use a fixed-size array. This is the fastest option: O(1) lookup, no hashing, cache-friendly.

### Sentinel choice

Pick a sentinel that **cannot be a real answer**:

- Use `-1` when all valid answers are `>= 0` (counts, lengths, non-negative costs).
- Use `i32::MIN` / `i32::MAX` when `-1` is a legal answer (e.g., profits or costs that can be negative).
- For boolean DP, use a `Vec<Vec<i32>>` holding `{-1 = unknown, 0 = false, 1 = true}`.

### 1D, 2D, 3D initialization

```rust
// 1D: states indexed by a single integer in [0, n].
let memo1: Vec<i32> = vec![-1; n + 1];

// 2D: states (i, j).
let memo2: Vec<Vec<i32>> = vec![vec![-1; m + 1]; n + 1];

// 3D: states (i, j, k) — e.g., stock with transaction cap, Cherry Pickup II.
let memo3: Vec<Vec<Vec<i32>>> = vec![vec![vec![-1; k]; m]; n];
```

> **Pitfall:** Initializing a 2D structure row-by-row is not needed in Rust since the `vec!` macro fills all values. Use the macro with an initial value as shown above.

### Boolean DP with a `Vec<Vec<i32>>` (-1 sentinel)

```rust
let mut memo: Vec<Vec<i32>> = vec![vec![-1; target + 1]; n + 1];  // -1 = "not computed"

fn subset_sum(idx: usize, target: usize, nums: &[i32], memo: &mut Vec<Vec<i32>>) -> bool {
    if target == 0 { return true; }
    if idx == 0 { return nums[0] == target as i32; }
    if memo[idx][target] != -1 { return memo[idx][target] == 1; }

    let not_take = subset_sum(idx - 1, target, nums, memo);
    let mut take = false;
    if nums[idx] as usize <= target {
        take = subset_sum(idx - 1, target - nums[idx] as usize, nums, memo);
    }

    memo[idx][target] = if take || not_take { 1 } else { 0 };
    take || not_take
}
```

---

## 3. HashMap-Based Memo (sparse or non-integer states)

Use a `HashMap` when the state space is **huge but sparsely visited**, **not naturally rectangular**, or **keyed by something other than small ints** (strings, sets, doubles).

### When to prefer a map over an array

| Situation | Use |
|-----------|-----|
| Dense, small, integer-bounded state | `Vec<Vec<i32>>` array |
| State range is large but only a few states are actually reached | `HashMap` |
| State key is a `String`, object, or bitmask that doesn't index cleanly | `HashMap` |
| Memory of the full rectangular table would not fit | `HashMap` |

### Pattern A — encode the state to a single `i64` key

When you have 2–3 bounded integer dimensions, pack them into one `i64`. This avoids object allocation per key and is faster than a string key.

```rust
use std::collections::HashMap;

// Encode (i, j) where 0 <= j < BASE into a single i64.
const BASE: i64 = 100000;

fn encode(i: i32, j: i32) -> i64 {
    i as i64 * BASE + j as i64
}

fn solve(i: i32, j: i32, memo: &mut HashMap<i64, i32>) -> i32 {
    if /* base case */ false { return 0; /* base value */ }
    let key = encode(i, j);
    if let Some(&cached) = memo.get(&key) { return cached; }

    let ans = 0; /* recurrence using solve(...) */
    memo.insert(key, ans);
    ans
}
```

### Pattern B — `HashMap<String, i32>` for irregular keys

Readable and flexible (e.g., Boolean Evaluation keys an operator range plus a target flag). Slightly slower due to string building and hashing.

```rust
use std::collections::HashMap;

fn count_ways(i: usize, j: usize, is_true: bool, exp: &str, memo: &mut HashMap<String, i32>) -> i32 {
    if i > j { return 0; }
    if i == j { /* single operand */ return 0; /* ... */ }
    let key = format!("{}|{}|{}", i, j, is_true as i32);
    if let Some(&cached) = memo.get(&key) { return cached; }

    let mut ways = 0;
    let mut k = i + 1;
    while k <= j - 1 {
        // partition on operator at k, combine left/right true/false counts...
        k += 2;
    }
    memo.insert(key, ways);
    ways
}
```

> **Tip:** `contains_key` + `[]` is two lookups. For hot paths, use `get()` and compare against `None` to perform a single lookup and retrieve the value via the returned `Option`.

---

## 4. Top-Down → Bottom-Up Conversion Recipe

Most interviewers accept either, but tabulation removes recursion-stack risk and is often easier to space-optimize. Convert mechanically:

1. **Identify the state and its ranges.** Whatever the memo is indexed by becomes the DP table dimensions: `vec![vec![0i32; M+1]; N+1]`.
2. **Translate base cases into table initialization.** Every `if base_case { return v; }` becomes a pre-filled cell or row/column.
3. **Determine iteration order.** A state must be computed *after* all states it depends on. If `solve(i)` calls `solve(i-1)`, iterate `i` from low to high. If it calls `solve(i+1)` (suffix DP), iterate `i` from high to low. For interval DP `dp(i,j)` that depends on smaller ranges, iterate by **increasing length** (or `i` descending, `j` ascending).
4. **Replace each recursive call with a table read.** `solve(i-1)` → `dp[i-1]`. The body of the recurrence is otherwise identical.
5. **Read the answer from the cell corresponding to the original top-level call.**

### Worked example — Fibonacci as a template

```rust
// Top-down (memoized)
fn fib_memo(n: usize, memo: &mut Vec<i32>) -> i32 {
    if n <= 1 { return n as i32; }             // base case
    if memo[n] != -1 { return memo[n]; }
    memo[n] = fib_memo(n - 1, memo) + fib_memo(n - 2, memo);
    memo[n]
}

// Bottom-up (tabulated) — same recurrence, explicit order
fn fib_tab(n: usize) -> i32 {
    if n <= 1 { return n as i32; }
    let mut dp = vec![0i32; n + 1];
    dp[0] = 0; dp[1] = 1;                     // base cases -> initialization
    for i in 2..=n {                           // order: low -> high (depends on i-1, i-2)
        dp[i] = dp[i - 1] + dp[i - 2];        // call -> table read
    }
    dp[n]                                      // answer location
}

// Space-optimized — only the last two states matter
fn fib_opt(n: usize) -> i32 {
    if n <= 1 { return n as i32; }
    let (mut prev2, mut prev1) = (0i32, 1i32);
    for _ in 2..=n {
        let cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    prev1
}
```

The progression `recursion → memoization → tabulation → space-optimized` applies to **every** problem in this topic. Space optimization is possible whenever the current row/state depends only on a constant number of previous rows/states.

---

## 5. When to Use Which

| Approach | Pros | Cons | Use when |
|----------|------|------|----------|
| **Memoization (top-down)** | Mirrors the recurrence directly; only computes *reachable* states; easiest to write from brute force. | Recursion stack overhead; risk of stack overflow on deep states; harder to space-optimize. | The recurrence is clear, the state graph is sparse, or you're deriving the solution live. |
| **Tabulation (bottom-up)** | No recursion stack; predictable iteration; easy to space-optimize; often faster (no call overhead). | Must compute *all* states even if some are unreachable; iteration order requires thought. | Depth is large, you need the space-optimized form, or all states are visited anyway. |
| **Space-optimized** | O(1) or O(n) memory instead of O(n²)/O(n·m). | Loses the full table, so path reconstruction/printing becomes impossible. | Only the final value is needed and the state depends on a bounded number of prior states. |

> If you must **reconstruct the solution** (Print LCS, Shortest Common Supersequence, Print LIS), keep the full table — do *not* space-optimize.

---

## 6. Recursion Depth & Stack Overflow

Rust's default thread stack (typically **8 MB**) can overflow with deep recursion. A top-down DP over `n = 10^5` along a single chain (e.g., LIS, Word Break, Frog Jump on a large array) can blow the stack.

Mitigations, in order of preference:

1. **Convert to tabulation.** The cleanest fix — no recursion at all.
2. **Spawn a thread with a larger stack** when you must keep top-down:
   ```rust
   // Spawn a thread with a 64 MB stack for deep recursion.
   std::thread::Builder::new()
       .stack_size(64 * 1024 * 1024)
       .spawn(|| {
           // your DP function here
       })
       .unwrap()
       .join()
       .unwrap();
   ```
3. **Iterative explicit-stack DFS** for the rare case where the recurrence is awkward to tabulate.

---

## 7. Complexity of Memoization

> **Time = (number of distinct states) × (work per state, excluding recursive calls).**
> **Space = (number of distinct states for the memo) + (max recursion depth for the stack).**

Each state is computed exactly once (after that it's a cache hit), so the total work is the state count multiplied by the cost of one transition.

| DP family | States | Work / state | Time | Space |
|-----------|--------|--------------|------|-------|
| Fibonacci / 1D | `O(n)` | `O(1)` | `O(n)` | `O(n)` → `O(1)` opt |
| Knapsack / Subset Sum | `O(n · W)` | `O(1)` | `O(n · W)` | `O(n · W)` → `O(W)` opt |
| LCS / Edit Distance / grids | `O(n · m)` | `O(1)` | `O(n · m)` | `O(n · m)` → `O(m)` opt |
| LIS (DP) | `O(n)` | `O(n)` | `O(n²)` | `O(n)` |
| Stock (K transactions) | `O(n · K · 2)` | `O(1)` | `O(n · K)` | `O(n · K)` → `O(K)` opt |
| Partition / MCM / interval | `O(n²)` | `O(n)` (the split loop) | `O(n³)` | `O(n²)` |

> **Rule of thumb:** to estimate a DP's complexity, multiply the size of the memo table by the cost of the loop inside one call. The split-point loop in interval DP is what turns `O(n²)` states into `O(n³)` time.

---

## 8. End-to-End Example — Generic Memoizer for 0/1 Knapsack

Putting the framework together: brute force, then array memo, then the array memo's space-optimized tabulation. This is the template behind every problem in [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md).

```rust
// ---- Step 1: brute-force recursion (exponential) ----
fn rec(i: i32, cap: usize, wt: &[i32], val: &[i32]) -> i32 {
    if i < 0 || cap == 0 { return 0; }
    let ui = i as usize;
    let not_take = rec(i - 1, cap, wt, val);
    let mut take = i32::MIN;
    if wt[ui] as usize <= cap {
        take = val[ui] + rec(i - 1, cap - wt[ui] as usize, wt, val);
    }
    take.max(not_take)
}

// ---- Step 2: top-down memoization ----
fn memo_helper(i: i32, cap: usize, wt: &[i32], val: &[i32], memo: &mut Vec<Vec<i32>>) -> i32 {
    if i < 0 || cap == 0 { return 0; }
    let ui = i as usize;
    if memo[ui][cap] != -1 { return memo[ui][cap]; }
    let not_take = memo_helper(i - 1, cap, wt, val, memo);
    let mut take = i32::MIN;
    if wt[ui] as usize <= cap {
        take = val[ui] + memo_helper(i - 1, cap - wt[ui] as usize, wt, val, memo);
    }
    memo[ui][cap] = take.max(not_take);
    memo[ui][cap]
}

fn memoized(n: usize, w: usize, wt: &[i32], val: &[i32]) -> i32 {
    let mut memo = vec![vec![-1i32; w + 1]; n];
    memo_helper(n as i32 - 1, w, wt, val, &mut memo)
}

// ---- Step 3: bottom-up tabulation ----
fn tabulation(n: usize, w: usize, wt: &[i32], val: &[i32]) -> i32 {
    let mut dp = vec![vec![0i32; w + 1]; n + 1];   // dp[i][cap] using items 0..i-1
    for i in 1..=n {
        for cap in 0..=w {
            let not_take = dp[i - 1][cap];
            let mut take = i32::MIN;
            if wt[i - 1] as usize <= cap {
                take = val[i - 1] + dp[i - 1][cap - wt[i - 1] as usize];
            }
            dp[i][cap] = take.max(not_take);
        }
    }
    dp[n][w]
}

// ---- Step 4: 1D space optimization (iterate capacity DOWNWARD for 0/1) ----
fn space_optimized(n: usize, w: usize, wt: &[i32], val: &[i32]) -> i32 {
    let mut dp = vec![0i32; w + 1];
    for i in 0..n {
        for cap in (wt[i] as usize..=w).rev() {    // downward => each item used once
            dp[cap] = dp[cap].max(val[i] + dp[cap - wt[i] as usize]);
        }
    }
    dp[w]
}
```

The four functions return identical answers; they differ only in time/space and in whether they use the recursion stack.

---

## 9. Checklist for Writing Any DP

1. **Define the state** — the minimal set of arguments that determines the answer.
2. **Write the recurrence** — express the answer in terms of smaller states.
3. **Establish base cases** — the smallest states with direct answers.
4. **Choose memo storage** — array if dense/integer-indexed, `HashMap` if sparse/irregular; pick a safe sentinel.
5. **Decide order** (for tabulation) — compute each state after its dependencies.
6. **Locate the answer** — the state matching the original top-level call.
7. **Space-optimize last** — only if you don't need to reconstruct the path.

---

## See Also

- [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md) — the simplest memo → tab → O(1) progression.
- [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md) — index + capacity state, 1D space optimization.
- [Partition DP (MCM)](../Patterns/Partition%20DP%20(MCM).md) — interval states, O(n³) split-loop analysis.
- [Dynamic Programming README](../README.md) — the full 55-problem index and methodology.

---

> **Last Updated:** 2026-06-26
