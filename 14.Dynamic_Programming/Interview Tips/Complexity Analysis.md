> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 4 of 4**

# Complexity Analysis — Dynamic Programming

How to reason about DP cost, plus a full time/space table for all 55 problems. The core model: **DP cost = (number of states) × (work per state)**. Read with [Coding Tips](./Coding%20Tips.md), [Common Mistakes](./Common%20Mistakes.md), [State Design](./State%20Design.md).

---

## The states × transition model

> **Time = number of distinct states × cost to compute one state (the transition).**
> **Space = number of states you must keep live (often reducible below the state count).**

- 1D state, O(1) transition → O(n). (Climbing Stairs)
- 1D state, O(k) transition → O(n·k). (Frog Jump K)
- 2D state, O(1) transition → O(n²) / O(m·n). (LCS, Edit Distance, grids)
- 2D interval state, O(n) transition (try every split `k`) → O(n³). (MCM, Burst Balloons)
- Bitmask state `2^n`, O(n) transition → O(2^n · n). (assignment DP)

This product is the *only* thing you need to quote a DP's complexity correctly.

---

## Pseudo-polynomial knapsack — the W trap

0/1 Knapsack / Coin Change run in **O(n·W)** where `W` is the *capacity value*, not the input size. `W` takes `log W` bits to write, so O(n·W) is exponential in the input *length* — it is **pseudo-polynomial**. Interviewers love to hear this distinction:

- "Coin Change is O(amount × coins)" — correct, but note `amount` is a value; if `amount` were given in binary the algorithm is not polynomial in the bit-length.
- This is why Subset Sum / Partition are NP-complete in general yet have an O(n·sum) DP: the DP is polynomial only when the numbers are small.

---

## O(n²) vs O(n log n) for LIS

| Variant | States × transition | Total |
|---------|---------------------|-------|
| Classic `dp[i]` | n states × O(n) scan back | **O(n²)** |
| Tails + binary search | n elements × O(log n) search | **O(n log n)** |

The O(n log n) version maintains a `tails[]` array where `tails[k]` is the smallest possible tail of an increasing subsequence of length `k+1`; binary search places each element. Expect the upgrade as a Google follow-up — see [Longest Increasing Subsequence](../Patterns/Longest%20Increasing%20Subsequence.md).

---

## Space optimization savings

| Technique | Applies to | Before → After |
|-----------|-----------|----------------|
| Rolling two rows | LCS, Edit Distance, grids, knapsack | O(m·n) → O(min(m,n)) |
| Single 1D array | Knapsack, Coin Change, Subset Sum | O(n·W) → O(W) |
| Two scalars | Climbing Stairs, House Robber, Fibonacci | O(n) → O(1) |
| Scalars per stock state | Stock I/II/III/cooldown/fee | O(n) → O(1) |
| Keep prev row + diagonal | Maximal Square, Min Path Sum | O(m·n) → O(n) |

Caveat: space optimization usually **destroys the ability to reconstruct the path**. If the problem asks to *print* the LCS / edit script / chosen coins, keep the full table.

---

## Memoization vs tabulation overhead

- **Tabulation (bottom-up):** no recursion stack, better cache locality, fills *every* state. Usually the constant-factor winner.
- **Memoization (top-down):** recursion + hashing/array-lookup overhead, but **only computes reachable states** — a win when the reachable state space is sparse relative to the full table. Watch the stack depth (`O(states)` worst case) for recursion-limit issues on large inputs.

---

## Full complexity table — all 55 problems

> `n`, `m` = sequence/grid sizes; `W` = capacity/amount/sum value; `T` = transaction limit. Space column shows the **optimized** figure where a standard optimization exists.

### 1D DP (Fibonacci Style)

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Climbing Stairs | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| Frog Jump (min cost) | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| Frog Jump with K distances | O(n·k) | O(n) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| House Robber | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| House Robber II | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| Max Sum of Non-Adjacent Elements | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| Min/Max Running Cost (1D) | O(n) | O(1) | [1D DP](../Patterns/1D%20DP%20(Fibonacci%20Style).md) |

### 2D DP (Grids)

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Unique Paths | O(m·n) | O(n) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Unique Paths II | O(m·n) | O(n) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Minimum Path Sum | O(m·n) | O(n) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Triangle | O(n²) | O(n) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Minimum Falling Path Sum | O(n²) | O(n) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Ninja's Training | O(n·4·3) = O(n) | O(1) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |
| Cherry Pickup II | O(m·n²) | O(n²) | [2D DP](../Patterns/2D%20DP%20(Grids).md) |

### Knapsack (Subset Sum)

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Subset Sum to Target | O(n·W) | O(W) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Partition Equal Subset Sum | O(n·sum) | O(sum) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Count Subsets with Sum K | O(n·W) | O(W) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| 0/1 Knapsack | O(n·W) | O(W) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Coin Change (min) | O(n·amount) | O(amount) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Coin Change II (count) | O(n·amount) | O(amount) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Target Sum | O(n·sum) | O(sum) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |
| Min Subset Sum Difference | O(n·sum) | O(sum) | [Knapsack](../Patterns/Knapsack%20(Subset%20Sum).md) |

### Longest Common Subsequence

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Longest Common Subsequence | O(m·n) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Print LCS | O(m·n) | O(m·n)* | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Longest Common Substring | O(m·n) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Longest Palindromic Subsequence | O(n²) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Min Insertions to Make Palindrome | O(n²) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Min Insert/Delete to Convert | O(m·n) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Shortest Common Supersequence | O(m·n) | O(m·n)* | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |
| Distinct Subsequences | O(m·n) | O(n) | [LCS](../Patterns/Longest%20Common%20Subsequence.md) |

`*` full table kept for reconstruction.

### Stock Buy Sell

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Best Time to Buy/Sell (I) | O(n) | O(1) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |
| Buy/Sell II (unlimited) | O(n) | O(1) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |
| Buy/Sell III (≤2) | O(n) | O(1) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |
| Buy/Sell IV (≤k) | O(n·k) | O(k) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |
| With Cooldown | O(n) | O(1) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |
| With Transaction Fee | O(n) | O(1) | [Stock](../Patterns/Stock%20Buy%20Sell.md) |

### Longest Increasing Subsequence

| Problem | Time | Space | Pattern |
|---------|------|-------|---------|
| LIS | O(n²) or O(n log n) | O(n) | [LIS](../Patterns/Longest%20Increasing%20Subsequence.md) |
| Print LIS | O(n²) | O(n) | [LIS](../Patterns/Longest%20Increasing%20Subsequence.md) |
| Largest Divisible Subset | O(n²) | O(n) | [LIS](../Patterns/Longest%20Increasing%20Subsequence.md) |
| Longest String Chain | O(n·L²) | O(n) | [LIS](../Patterns/Longest%20Increasing%20Subsequence.md) |
| Number of LIS | O(n²) | O(n) | [LIS](../Patterns/Longest%20Increasing%20Subsequence.md) |

### Partition DP (MCM)

| Problem | Time | Space | Pattern |
|---------|------|-------|---------|
| Matrix Chain Multiplication | O(n³) | O(n²) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |
| Min Cost to Cut a Stick | O(n³) | O(n²) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |
| Burst Balloons | O(n³) | O(n²) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |
| Boolean Evaluation | O(n³) | O(n²) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |
| Palindrome Partitioning II | O(n²) | O(n²) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |
| Partition Array for Max Sum | O(n·k) | O(n) | [Partition DP](../Patterns/Partition%20DP%20(MCM).md) |

### DP on Strings

| Problem | Time | Space (opt) | Pattern |
|---------|------|-------------|---------|
| Edit Distance | O(m·n) | O(n) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Wildcard Matching | O(m·n) | O(n) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Regex Matching | O(m·n) | O(m·n) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Word Break | O(n²·L) | O(n) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Decode Ways | O(n) | O(1) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Palindromic Substrings (count) | O(n²) | O(1) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Longest Palindromic Substring | O(n²) | O(1) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |
| Interleaving String | O(m·n) | O(n) | [DP on Strings](../Patterns/DP%20on%20Strings.md) |

---

## Reading the table in an interview

1. Quote complexity as **states × transition**, never memorized.
2. Flag **pseudo-polynomial** for any knapsack/coin/subset problem.
3. Offer the **space optimization** number and note when it kills path reconstruction.
4. For LIS, always mention both O(n²) and O(n log n).

> **Last Updated:** 2026-06-26
