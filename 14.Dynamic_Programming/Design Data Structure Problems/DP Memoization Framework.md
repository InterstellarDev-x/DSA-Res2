> **Topic:** [Dynamic Programming](../README.md) · **Design Data Structure Problems**

# DP Memoization Framework

Dynamic programming is, at its heart, **recursion + a cache**. Every top-down DP is a plain recursive function whose results are stored so that each distinct subproblem is solved exactly once. This file is a reusable toolkit: how to size and initialize memo tables, how to memoize non-integer / sparse states, how to mechanically convert a top-down solution into a bottom-up one, and how to reason about correctness, complexity, and stack depth.

If you internalize the recipes here, you can solve *any* of the 55 problems in this topic by writing the brute-force recursion first and then "switching on" the cache.

---

## 1. The Core Idea — Recursion With a Cache

A subproblem is uniquely identified by its **state** (the arguments that actually vary across recursive calls). Memoization stores the answer for each state the first time it is computed and returns the stored value on every subsequent visit.

```java
// Skeleton of every top-down DP.
int solve(State s) {
    if (isBaseCase(s)) return baseValue(s);   // 1. base case
    if (memo.contains(s)) return memo.get(s);  // 2. cache hit -> return
    int ans = combine(                         // 3. recurrence / transition
        solve(next1(s)),
        solve(next2(s)),
        ...
    );
    memo.put(s, ans);                          // 4. cache the result
    return ans;
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
- Use `Integer.MIN_VALUE` / `Integer.MAX_VALUE` when `-1` is a legal answer (e.g., profits or costs that can be negative).
- For boolean DP, use an `int[][]` holding `{-1 = unknown, 0 = false, 1 = true}`, or a `Boolean[][]` (whose default `null` *is* the "unknown" sentinel).

### 1D, 2D, 3D initialization

```java
// 1D: states indexed by a single integer in [0, n].
int[] memo1 = new int[n + 1];
Arrays.fill(memo1, -1);

// 2D: states (i, j). Arrays.fill works per-row only.
int[][] memo2 = new int[n + 1][m + 1];
for (int[] row : memo2) Arrays.fill(row, -1);

// 3D: states (i, j, k) — e.g., stock with transaction cap, Cherry Pickup II.
int[][][] memo3 = new int[n][m][k];
for (int[][] plane : memo3)
    for (int[] row : plane)
        Arrays.fill(row, -1);
```

> **Pitfall:** `Arrays.fill(memo2, -1)` does **not** work for a 2D array — it would try to fill the outer array with the `int` `-1`, which fails to compile (or fills rows with `null` for object arrays). Always fill row-by-row.

### Boolean DP with a `Boolean[][]` (null sentinel)

```java
Boolean[][] memo = new Boolean[n + 1][target + 1];   // defaults to null = "not computed"

boolean subsetSum(int idx, int target, int[] nums) {
    if (target == 0) return true;
    if (idx == 0)    return nums[0] == target;
    if (memo[idx][target] != null) return memo[idx][target];

    boolean notTake = subsetSum(idx - 1, target, nums);
    boolean take = false;
    if (nums[idx] <= target) take = subsetSum(idx - 1, target - nums[idx], nums);

    return memo[idx][target] = (take || notTake);
}
```

---

## 3. HashMap-Based Memo (sparse or non-integer states)

Use a `Map` when the state space is **huge but sparsely visited**, **not naturally rectangular**, or **keyed by something other than small ints** (strings, sets, doubles).

### When to prefer a map over an array

| Situation | Use |
|-----------|-----|
| Dense, small, integer-bounded state | `int[][]` array |
| State range is large but only a few states are actually reached | `HashMap` |
| State key is a `String`, object, or bitmask that doesn't index cleanly | `HashMap` |
| Memory of the full rectangular table would not fit | `HashMap` |

### Pattern A — encode the state to a single `long` key

When you have 2–3 bounded integer dimensions, pack them into one `long`. This avoids object allocation per key and is faster than a string key.

```java
// Encode (i, j) where 0 <= j < BASE into a single long.
private static final long BASE = 100_000L;

long encode(int i, int j) {
    return (long) i * BASE + j;
}

Map<Long, Integer> memo = new HashMap<>();

int solve(int i, int j) {
    if (/* base case */ ) return /* base value */;
    long key = encode(i, j);
    Integer cached = memo.get(key);
    if (cached != null) return cached;

    int ans = /* recurrence using solve(...) */ 0;
    memo.put(key, ans);
    return ans;
}
```

### Pattern B — `Map<String, Integer>` for irregular keys

Readable and flexible (e.g., Boolean Evaluation keys an operator range plus a target flag). Slightly slower due to string building and hashing.

```java
Map<String, Integer> memo = new HashMap<>();

int countWays(int i, int j, boolean isTrue, String exp) {
    if (i > j) return 0;
    if (i == j) { /* single operand */ return /* ... */ 0; }
    String key = i + "|" + j + "|" + isTrue;
    if (memo.containsKey(key)) return memo.get(key);

    int ways = 0;
    for (int k = i + 1; k <= j - 1; k += 2) {
        // partition on operator at k, combine left/right true/false counts...
    }
    memo.put(key, ways);
    return ways;
}
```

> **Tip:** `containsKey` + `get` is two lookups. For hot paths, do a single `get` and compare against `null` (boxing means `null` is a clean "absent" sentinel).

---

## 4. Top-Down → Bottom-Up Conversion Recipe

Most interviewers accept either, but tabulation removes recursion-stack risk and is often easier to space-optimize. Convert mechanically:

1. **Identify the state and its ranges.** Whatever the memo is indexed by becomes the DP table dimensions: `int[][] dp = new int[N+1][M+1]`.
2. **Translate base cases into table initialization.** Every `if (baseCase) return v;` becomes a pre-filled cell or row/column.
3. **Determine iteration order.** A state must be computed *after* all states it depends on. If `solve(i)` calls `solve(i-1)`, iterate `i` from low to high. If it calls `solve(i+1)` (suffix DP), iterate `i` from high to low. For interval DP `dp(i,j)` that depends on smaller ranges, iterate by **increasing length** (or `i` descending, `j` ascending).
4. **Replace each recursive call with a table read.** `solve(i-1)` → `dp[i-1]`. The body of the recurrence is otherwise identical.
5. **Read the answer from the cell corresponding to the original top-level call.**

### Worked example — Fibonacci as a template

```java
// Top-down (memoized)
int fibMemo(int n, int[] memo) {
    if (n <= 1) return n;                 // base case
    if (memo[n] != -1) return memo[n];
    return memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
}

// Bottom-up (tabulated) — same recurrence, explicit order
int fibTab(int n) {
    if (n <= 1) return n;
    int[] dp = new int[n + 1];
    dp[0] = 0; dp[1] = 1;                  // base cases -> initialization
    for (int i = 2; i <= n; i++)           // order: low -> high (depends on i-1, i-2)
        dp[i] = dp[i - 1] + dp[i - 2];     // call -> table read
    return dp[n];                          // answer location
}

// Space-optimized — only the last two states matter
int fibOpt(int n) {
    if (n <= 1) return n;
    int prev2 = 0, prev1 = 1;
    for (int i = 2; i <= n; i++) {
        int cur = prev1 + prev2;
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}
```

The progression `recursion → memoization → tabulation → space-optimized` applies to **every** problem in this topic. Space optimization is possible whenever the current row/state depends only on a constant number of previous rows/states.

---

## 5. When to Use Which

| Approach | Pros | Cons | Use when |
|----------|------|------|----------|
| **Memoization (top-down)** | Mirrors the recurrence directly; only computes *reachable* states; easiest to write from brute force. | Recursion stack overhead; risk of `StackOverflowError` on deep states; harder to space-optimize. | The recurrence is clear, the state graph is sparse, or you're deriving the solution live. |
| **Tabulation (bottom-up)** | No recursion stack; predictable iteration; easy to space-optimize; often faster (no call overhead). | Must compute *all* states even if some are unreachable; iteration order requires thought. | Depth is large, you need the space-optimized form, or all states are visited anyway. |
| **Space-optimized** | O(1) or O(n) memory instead of O(n²)/O(n·m). | Loses the full table, so path reconstruction/printing becomes impossible. | Only the final value is needed and the state depends on a bounded number of prior states. |

> If you must **reconstruct the solution** (Print LCS, Shortest Common Supersequence, Print LIS), keep the full table — do *not* space-optimize.

---

## 6. Recursion Depth & Stack Overflow

Java's default thread stack (~512 KB) overflows at roughly **10,000–20,000** frames of moderate-size methods. A top-down DP over `n = 10^5` along a single chain (e.g., LIS, Word Break, Frog Jump on a large array) can blow the stack.

Mitigations, in order of preference:

1. **Convert to tabulation.** The cleanest fix — no recursion at all.
2. **Run the recursion on a larger thread stack** when you must keep top-down:
   ```java
   public static void main(String[] args) throws InterruptedException {
       Thread t = new Thread(null, () -> {
           // ... call deep recursive DP here ...
       }, "dp", 1 << 26);   // 64 MB stack
       t.start();
       t.join();
   }
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

```java
import java.util.Arrays;

public class KnapsackFramework {

    // ---- Step 1: brute-force recursion (exponential) ----
    static int rec(int i, int cap, int[] wt, int[] val) {
        if (i < 0 || cap == 0) return 0;
        int notTake = rec(i - 1, cap, wt, val);
        int take = Integer.MIN_VALUE;
        if (wt[i] <= cap) take = val[i] + rec(i - 1, cap - wt[i], wt, val);
        return Math.max(take, notTake);
    }

    // ---- Step 2: top-down memoization ----
    static int memoized(int n, int W, int[] wt, int[] val) {
        int[][] memo = new int[n][W + 1];
        for (int[] row : memo) Arrays.fill(row, -1);
        return memo(n - 1, W, wt, val, memo);
    }

    static int memo(int i, int cap, int[] wt, int[] val, int[][] memo) {
        if (i < 0 || cap == 0) return 0;
        if (memo[i][cap] != -1) return memo[i][cap];
        int notTake = memo(i - 1, cap, wt, val, memo);
        int take = Integer.MIN_VALUE;
        if (wt[i] <= cap) take = val[i] + memo(i - 1, cap - wt[i], wt, val, memo);
        return memo[i][cap] = Math.max(take, notTake);
    }

    // ---- Step 3: bottom-up tabulation ----
    static int tabulation(int n, int W, int[] wt, int[] val) {
        int[][] dp = new int[n + 1][W + 1];   // dp[i][cap] using items 0..i-1
        for (int i = 1; i <= n; i++) {
            for (int cap = 0; cap <= W; cap++) {
                int notTake = dp[i - 1][cap];
                int take = Integer.MIN_VALUE;
                if (wt[i - 1] <= cap) take = val[i - 1] + dp[i - 1][cap - wt[i - 1]];
                dp[i][cap] = Math.max(take, notTake);
            }
        }
        return dp[n][W];
    }

    // ---- Step 4: 1D space optimization (iterate capacity DOWNWARD for 0/1) ----
    static int spaceOptimized(int n, int W, int[] wt, int[] val) {
        int[] dp = new int[W + 1];
        for (int i = 0; i < n; i++)
            for (int cap = W; cap >= wt[i]; cap--)        // downward => each item used once
                dp[cap] = Math.max(dp[cap], val[i] + dp[cap - wt[i]]);
        return dp[W];
    }
}
```

The four methods return identical answers; they differ only in time/space and in whether they use the recursion stack.

---

## 9. Checklist for Writing Any DP

1. **Define the state** — the minimal set of arguments that determines the answer.
2. **Write the recurrence** — express the answer in terms of smaller states.
3. **Establish base cases** — the smallest states with direct answers.
4. **Choose memo storage** — array if dense/integer-indexed, HashMap if sparse/irregular; pick a safe sentinel.
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
