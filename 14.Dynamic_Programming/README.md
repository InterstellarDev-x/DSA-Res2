[← Graphs](../13.Graphs/README.md) · **Dynamic Programming (Step 16)** · [Tries →](../15.Tries/README.md)

# Dynamic Programming

A complete, production-quality reference for the **Dynamic Programming** topic — 55 problems across 8 core patterns, each with full Java solutions, the recursion → memoization → tabulation → space-optimized progression, complexity analysis, dry-runs, and recognition signals. DP turns exponential brute-force recursion into polynomial time by caching answers to **overlapping subproblems**; once you can spot the state and the recurrence, an entire class of "count the ways / minimize the cost / can you reach" problems collapses into a handful of reusable templates.

---

## Patterns

| # | Pattern | Core idea | Link |
|---|---------|-----------|------|
| 1 | 1D DP (Fibonacci Style) | State is a single index; answer depends on a constant number of previous indices. Reduces to two rolling variables. | [1D DP (Fibonacci Style)](./Patterns/1D%20DP%20(Fibonacci%20Style).md) |
| 2 | 2D DP (Grids) | State is a cell `(row, col)`; move down/right (or down). Optimize O(m·n) tables to rolling rows. | [2D DP (Grids)](./Patterns/2D%20DP%20(Grids).md) |
| 3 | Knapsack (Subset Sum) | State = `index + remaining capacity`. Pick / not-pick recurrence; 0/1 vs unbounded differ by loop order. | [Knapsack (Subset Sum)](./Patterns/Knapsack%20(Subset%20Sum).md) |
| 4 | Longest Common Subsequence | State = `(i, j)` over two strings; match → `1 + diag`, else `max(skip)`. Backbone of most string-pair DP. | [Longest Common Subsequence](./Patterns/Longest%20Common%20Subsequence.md) |
| 5 | Stock Buy Sell | Unified state machine `dp[day][transactionsLeft][holding]`; all six variants specialize from it. | [Stock Buy Sell](./Patterns/Stock%20Buy%20Sell.md) |
| 6 | Longest Increasing Subsequence | `dp[i] =` longest ending at `i` (O(n²)) or a tails array with binary search (O(n log n)). | [Longest Increasing Subsequence](./Patterns/Longest%20Increasing%20Subsequence.md) |
| 7 | Partition DP (MCM) | Interval state `dp(i, j)`; try every partition point `k`; combine left + right + cost. O(n³). | [Partition DP (MCM)](./Patterns/Partition%20DP%20(MCM).md) |
| 8 | DP on Strings | Two-pointer / prefix DP over one or two strings: edit distance, matching, segmentation, palindromes. | [DP on Strings](./Patterns/DP%20on%20Strings.md) |

---

## All 55 Problems

| # | Problem | LC | Difficulty | Pattern | Companies |
|---|---------|----|------------|---------|-----------|
| 1 | Climbing Stairs | 70 | Easy | 1D DP | Amazon, Adobe, Google |
| 2 | Frog Jump (min cost) | GFG | Easy | 1D DP | Amazon, Microsoft |
| 3 | Frog Jump with K distances | GFG | Medium | 1D DP | Amazon, Google |
| 4 | House Robber | 198 | Medium | 1D DP | Amazon, Google, Microsoft, Adobe |
| 5 | House Robber II | 213 | Medium | 1D DP | Amazon, Microsoft, Adobe |
| 6 | Maximum Sum of Non-Adjacent Elements | GFG | Medium | 1D DP | Amazon, Google |
| 7 | Min/Max Running Cost (Ninja's Training 1D) | GFG | Medium | 1D DP | Microsoft, Amazon |
| 8 | Unique Paths | 62 | Medium | 2D DP | Amazon, Google, Goldman Sachs, Adobe |
| 9 | Unique Paths II | 63 | Medium | 2D DP | Amazon, Microsoft, Adobe |
| 10 | Minimum Path Sum | 64 | Medium | 2D DP | Amazon, Google, Goldman Sachs |
| 11 | Triangle | 120 | Medium | 2D DP | Amazon, Google, Adobe |
| 12 | Minimum Falling Path Sum | 931 | Medium | 2D DP | Amazon, Google |
| 13 | Ninja's Training (3 activities) | 1186 | Medium | 2D DP | Amazon, Microsoft |
| 14 | Cherry Pickup II | 1463 | Hard | 2D DP | Google, Amazon, Goldman Sachs |
| 15 | Subset Sum Equal to Target | GFG | Medium | Knapsack | Amazon, Microsoft, Adobe |
| 16 | Partition Equal Subset Sum | 416 | Medium | Knapsack | Amazon, Google, Microsoft, Adobe |
| 17 | Count Subsets with Sum K | GFG | Medium | Knapsack | Amazon, Google |
| 18 | 0/1 Knapsack | GFG | Medium | Knapsack | Amazon, Microsoft, Goldman Sachs, Adobe |
| 19 | Coin Change (min coins) | 322 | Medium | Knapsack | Amazon, Google, Microsoft, Goldman Sachs |
| 20 | Coin Change II (count ways) | 518 | Medium | Knapsack | Amazon, Google, Adobe |
| 21 | Target Sum | 494 | Medium | Knapsack | Amazon, Google, Microsoft |
| 22 | Partition Set Into Two Subsets With Min Difference | GFG | Hard | Knapsack | Amazon, Goldman Sachs |
| 23 | Longest Common Subsequence | 1143 | Medium | LCS | Amazon, Google, Microsoft, Adobe |
| 24 | Print LCS | GFG | Medium | LCS | Amazon, Microsoft |
| 25 | Longest Common Substring | GFG | Medium | LCS | Amazon, Google, Adobe |
| 26 | Longest Palindromic Subsequence | 516 | Medium | LCS | Amazon, Google, Adobe |
| 27 | Minimum Insertions to Make Palindrome | 1312 | Medium | LCS | Amazon, Google |
| 28 | Minimum Insertions/Deletions to Convert A to B | GFG | Medium | LCS | Amazon, Microsoft |
| 29 | Shortest Common Supersequence | 1092 | Hard | LCS | Amazon, Google, Goldman Sachs |
| 30 | Distinct Subsequences | 115 | Hard | LCS | Amazon, Google, Adobe |
| 31 | Best Time to Buy and Sell Stock | 121 | Easy | Stock | Amazon, Google, Microsoft, Goldman Sachs, Adobe |
| 32 | Buy and Sell Stock II | 122 | Medium | Stock | Amazon, Google, Microsoft |
| 33 | Buy and Sell Stock III | 123 | Hard | Stock | Amazon, Google, Goldman Sachs |
| 34 | Buy and Sell Stock IV | 188 | Hard | Stock | Amazon, Google, Adobe |
| 35 | Buy and Sell With Cooldown | 309 | Medium | Stock | Amazon, Microsoft, Adobe |
| 36 | Buy and Sell With Transaction Fee | 714 | Medium | Stock | Amazon, Google |
| 37 | Longest Increasing Subsequence | 300 | Medium | LIS | Amazon, Google, Microsoft, Goldman Sachs, Adobe |
| 38 | Print LIS | GFG | Medium | LIS | Amazon, Microsoft |
| 39 | Largest Divisible Subset | 368 | Medium | LIS | Amazon, Google, Adobe |
| 40 | Longest String Chain | 1048 | Medium | LIS | Amazon, Google |
| 41 | Number of LIS | 673 | Medium | LIS | Amazon, Google, Microsoft |
| 42 | Matrix Chain Multiplication | GFG | Hard | Partition DP | Amazon, Microsoft, Goldman Sachs |
| 43 | Minimum Cost to Cut a Stick | 1547 | Hard | Partition DP | Amazon, Google |
| 44 | Burst Balloons | 312 | Hard | Partition DP | Amazon, Google, Goldman Sachs |
| 45 | Boolean Evaluation (count ways to true) | GFG | Hard | Partition DP | Amazon, Google, Adobe |
| 46 | Palindrome Partitioning II | 132 | Hard | Partition DP | Amazon, Google, Microsoft |
| 47 | Partition Array for Maximum Sum | 1043 | Medium | Partition DP | Amazon, Google |
| 48 | Edit Distance | 72 | Medium | DP on Strings | Amazon, Google, Microsoft, Goldman Sachs, Adobe |
| 49 | Wildcard Matching | 44 | Hard | DP on Strings | Amazon, Google, Adobe |
| 50 | Regular Expression Matching | 10 | Hard | DP on Strings | Amazon, Google, Microsoft |
| 51 | Word Break | 139 | Medium | DP on Strings | Amazon, Google, Microsoft, Adobe |
| 52 | Decode Ways | 91 | Medium | DP on Strings | Amazon, Google, Microsoft, Goldman Sachs |
| 53 | Palindromic Substrings (count) | 647 | Medium | DP on Strings | Amazon, Google, Adobe |
| 54 | Longest Palindromic Substring | 5 | Medium | DP on Strings | Amazon, Google, Microsoft, Adobe |
| 55 | Interleaving String | 97 | Medium | DP on Strings | Amazon, Google |

---

## The DP Methodology — A 5-Step Approach

Every problem in this topic is solved by the same disciplined procedure. Derive these five things and the code writes itself.

| Step | Question to answer | Example (Knapsack) |
|------|--------------------|--------------------|
| **1. Define the state** | What minimal set of parameters uniquely identifies a subproblem? | `f(i, cap)` = max value using items `0..i` with capacity `cap`. |
| **2. Recurrence / transition** | How is the answer built from smaller states? | `f(i,cap) = max(f(i-1,cap), val[i] + f(i-1,cap-wt[i]))`. |
| **3. Base case** | What are the smallest states with direct answers? | `f(_, 0) = 0`, and `i < 0 → 0`. |
| **4. Order of computation** | In tabulation, which states must be computed first? | Increasing `i`, all `cap` — each row depends on the row above. |
| **5. Answer location** | Which state holds the final answer? | `f(n-1, W)` → cell `dp[n][W]`. |

> Always write the **brute-force recursion first** (steps 1–3, 5). Memoization (step 4 implicit) then comes for free by adding a cache; tabulation makes step 4 explicit. See the [DP Memoization Framework](./Design%20Data%20Structure%20Problems/DP%20Memoization%20Framework.md) for the mechanical conversion recipe.

---

## Memoization vs Tabulation vs Space-Optimized

Every DP in this repository is presented as a progression through these three (sometimes four) stages. The recurrence never changes — only *how* and *in what order* the answers are stored.

| Stage | What it is | Time | Space | Trade-off |
|-------|-----------|------|-------|-----------|
| **Recursion (brute force)** | Plain recursive recurrence, no cache. | Exponential | O(depth) stack | Correct but recomputes overlapping subproblems. |
| **Memoization (top-down)** | Same recursion + a cache (`int[][] memo` filled with `-1`, or a HashMap). | O(states × transition) | O(states) + O(depth) stack | Easiest from brute force; only reachable states computed; recursion-stack risk. |
| **Tabulation (bottom-up)** | Iterative fill in dependency order; no recursion. | O(states × transition) | O(states) | No stack risk; computes all states; easy to optimize next. |
| **Space-optimized** | Keep only the rows/variables the recurrence actually reads. | O(states × transition) | O(1) or O(one row) | Smallest memory; **loses the table → can't reconstruct the path**. |

> **Reconstruction caveat:** Print LCS, Shortest Common Supersequence, and Print LIS need the full table — do not space-optimize those.

---

## Recognizing DP — Signal Phrases

Reach for DP when a problem has **optimal substructure** (optimal answer composed of optimal sub-answers) **and overlapping subproblems** (the same sub-question recurs across branches). The wording usually gives it away:

| Signal phrase in the prompt | Likely pattern |
|-----------------------------|----------------|
| *"count the number of ways…"* | Knapsack (Coin Change II, Count Subsets), Decode Ways, Number of LIS, Boolean Evaluation |
| *"minimum / maximum cost / sum / path"* | 1D DP, 2D Grids, Partition DP, Knapsack |
| *"can you reach / is it possible / can you partition"* | Subset Sum, Word Break, Interleaving String |
| *"longest / shortest subsequence / substring / chain"* | LCS, LIS, DP on Strings |
| *"at most K transactions / K choices / K steps"* | Stock Buy Sell, Frog Jump with K |
| *"minimum cuts / splits / partitions"* | Partition DP (Palindrome Partitioning II, MCM) |
| *"edit / convert / transform A into B"* | Edit Distance, LCS family |

> **Greedy vs DP:** if a locally optimal choice provably leads to a global optimum (e.g., Stock II), prefer greedy. If choices interact and you must explore alternatives, it's DP. When unsure, write the recursion — overlapping subproblems confirm DP.

---

## Design / Framework Problems

| Problem | Focus | Link |
|---------|-------|------|
| DP Memoization Framework | Memo table sizing & `Arrays.fill(-1)` init, HashMap/`long`-encoded sparse states, top-down → bottom-up conversion recipe, stack-overflow mitigations, and the states × transition complexity model. | [DP Memoization Framework](./Design%20Data%20Structure%20Problems/DP%20Memoization%20Framework.md) |

---

## Company Coverage

| Company | Representative problems |
|---------|-------------------------|
| **Amazon** | Climbing Stairs, House Robber, Coin Change, Longest Common Subsequence, Edit Distance, Best Time to Buy and Sell Stock, Longest Increasing Subsequence, Word Break, Burst Balloons |
| **Google** | Unique Paths, Cherry Pickup II, Partition Equal Subset Sum, Distinct Subsequences, Regular Expression Matching, Number of LIS, Burst Balloons, Interleaving String |
| **Microsoft** | House Robber II, Ninja's Training, 0/1 Knapsack, Decode Ways, Buy and Sell Stock II, Palindrome Partitioning II, Longest Palindromic Substring |
| **Goldman Sachs** | Unique Paths, Minimum Path Sum, Coin Change, Matrix Chain Multiplication, Best Time to Buy and Sell Stock, Edit Distance, Min Subset Difference |
| **Adobe** | Climbing Stairs, House Robber, Partition Equal Subset Sum, Longest Common Substring, Wildcard Matching, Largest Divisible Subset, Palindromic Substrings |

---

## How to Use This Repository

1. Read the **DP Methodology** above and internalize the 5 steps — they apply to every single problem.
2. Start with **1D DP (Fibonacci Style)** to lock in the recursion → memo → tab → O(1) progression on the simplest states.
3. Move to **2D DP (Grids)** and **Knapsack** — they introduce two-dimensional states and the all-important 1D space optimization.
4. Master **LCS** and **DP on Strings** together; LCS is the backbone that LPS, SCS, Edit Distance, and many others reduce to.
5. Study the **Stock Buy Sell** unified state machine once and you get all six variants; do the same with the **Partition DP** interval template.
6. Learn both LIS forms (**O(n²)** and **O(n log n)**), then see how Largest Divisible Subset, Longest String Chain, and Number of LIS are predicate/count tweaks of it.
7. Keep the [DP Memoization Framework](./Design%20Data%20Structure%20Problems/DP%20Memoization%20Framework.md) open while practicing — it is the cheat sheet for memo init, sparse states, and stack-depth issues.

---

> **Last Updated:** 2026-06-26
