> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 3 of 4**

# State Design — Dynamic Programming

The hardest part of DP is not the code — it is **picking the right state**. This guide gives you a single question to ask, the canonical state shapes, how to compress and reduce dimensions, and five worked derivations. Read with [Coding Tips](./Coding%20Tips.md), [Common Mistakes](./Common%20Mistakes.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## The one question: "What varies as I move through the problem?"

A DP state is the **minimal set of variables that fully describes a subproblem**. Find them by asking: as I make decisions one at a time, what do I need to remember to make the next decision correctly?

- If I only need the current index → 1D state.
- If I need an index *and* a budget → 2D state.
- If I need two positions in two strings → `(i, j)`.
- If I need a position *and* a mode/flag → add a dimension.

A good test: two subproblems with the **same state values must have the same answer**. If they don't, you're missing a dimension.

---

## Canonical state shapes

### 1. Single index — `dp[i]` (Fibonacci-style)

The answer at `i` depends on a constant number of earlier indices. *What varies:* position only.

- Climbing Stairs, House Robber, Max Sum Non-Adjacent.
- Pattern: [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md).

### 2. Index + remaining capacity — `dp[i][cap]` (Knapsack)

*What varies:* which item I'm considering, and how much capacity is left.

- 0/1 Knapsack, Subset Sum, Coin Change, Target Sum, Partition Equal Subset Sum.
- Pattern: [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md).

### 3. Two pointers — `dp[i][j]` over two sequences (LCS / strings)

*What varies:* how far I've consumed each of the two strings.

- LCS, Edit Distance, Distinct Subsequences, Interleaving String, Regex/Wildcard Matching.
- Patterns: [Longest Common Subsequence](../Patterns/Longest%20Common%20Subsequence.md), [DP on Strings](../Patterns/DP%20on%20Strings.md).

### 4. Day + holding + transactions — `dp[day][holding][k]` (Stock)

*What varies:* which day, whether I currently hold a share, and how many transactions remain.

- All six stock variants specialize from this machine.
- Pattern: [Stock Buy Sell](../Patterns/Stock%20Buy%20Sell.md).

### 5. Interval — `dp[i][j]` over a range (Partition DP)

*What varies:* the left and right bounds of the subarray I'm solving; I try every split `k` inside.

- Matrix Chain Multiplication, Burst Balloons, Palindrome Partitioning II, Min Cost to Cut a Stick.
- Pattern: [Partition DP (MCM)](../Patterns/Partition%20DP%20(MCM).md).

---

## State compression and bitmask

When a dimension is a **set of used items** (TSP-style, "assign jobs to people"), encode it as a bitmask integer:

```rust
// dp[mask] = best cost having assigned the people in `mask`
let mut dp: Vec<i32> = vec![i32::MAX; 1 << n];
dp[0] = 0;
for mask in 0usize..(1 << n) {
    if dp[mask] == i32::MAX { continue; } // skip unreachable states
    let i = mask.count_ones() as usize; // next task index = count of set bits
    for p in 0..n {
        if (mask & (1 << p)) == 0 {     // person p still free
            let next = mask | (1 << p);
            dp[next] = dp[next].min(dp[mask] + cost[i][p]);
        }
    }
}
```

`O(2^n · n)` — viable only for small `n` (≈ 20).

---

## Reducing dimensions

Drop a dimension when it is **redundant or derivable**:

- **Derivable from the others.** In bitmask assignment above, the task index equals `bitCount(mask)`, so it isn't a separate dimension.
- **Only the previous layer is read.** Grid/string DP that reads only row `i-1` collapses to two rolling rows, then often one — a *space* reduction, not a state reduction, but it follows from the same analysis (see [Coding Tips](./Coding%20Tips.md)).
- **A flag with few values folds into the recurrence.** Stock's `holding` is just 2 states; you can keep two scalars (`hold`, `cash`) instead of a `[2]` array.

---

## Five worked derivations

### A. Coin Change → `dp[amount]`

*What varies as I add coins?* Only the amount still to be made. The coin index does **not** need to be in the state because coins are reusable and order-independent for the *min* version. → 1D state `dp[a]`, recurrence `dp[a] = min(dp[a-c]+1)`.

### B. 0/1 Knapsack → `dp[i][cap]`

*What varies?* Which item I'm deciding on (each usable once → I must track progress through items) **and** remaining capacity. → `dp[i][cap] = max(skip dp[i-1][cap], take dp[i-1][cap-w[i]] + v[i])`. The item index *is* required here (unlike Coin Change min) because items can't repeat.

### C. Edit Distance → `dp[i][j]`

*What varies?* How much of each string I've transformed. Subproblem = "convert prefix of length `i` into prefix of length `j`". Same state values ⇒ same min ops, so `(i, j)` is sufficient. → three transitions (insert/delete/replace).

### D. Best Time to Buy/Sell III (at most 2 txns) → `dp[day][k][holding]`

*What varies?* The day, transactions used so far (`k ≤ 2`), and whether I hold a share. Drop any one and two different situations share a state but differ in answer — proof all three are needed. Specializes to scalars `buy1, sell1, buy2, sell2`.

### E. Burst Balloons → `dp[i][j]`

*What varies?* The open interval of balloons still present. The key reframing ("which bursts **last**") is what makes `(i, j)` self-contained: the last balloon's neighbors are exactly the fixed bounds `i` and `j`, so the two sides are independent. → interval DP, `O(n³)`.

---

## A checklist for "I can't find the state"

1. Write the brute-force recursion; its **parameters are your state**.
2. Ask: do two calls with the same parameters always return the same value? If not, add the missing parameter.
3. Ask: is any parameter derivable from the others? If so, drop it.
4. Bound each dimension's range — that product is your table size and time (see [Complexity Analysis](./Complexity%20Analysis.md)).

> **Last Updated:** 2026-06-26
