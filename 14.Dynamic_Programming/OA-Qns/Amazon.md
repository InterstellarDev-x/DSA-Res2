> **Topic:** [Dynamic Programming](../README.md) · **Section:** OA-Qns

# Amazon — DP Online Assessment Questions

Amazon's Online Assessment (OA) for SDE-1 and SDE-2 leans on **"count the ways"** and **"minimize the cost"** DP that can be auto-graded with hidden tests. The bar is rarely a hard interval-DP; instead it is a *medium* problem dressed in a business story (warehouse coins, package selection, palindromic SKUs). The grader checks correctness on large inputs, so an exponential brute-force will TLE — you must reach the tabulated or memoized solution.

See also: [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md) · [Topic README](../README.md)

---

## Frequency Table

| Problem | Frequency | Difficulty | Pattern | LC | Source note |
|---------|-----------|------------|---------|----|-------------|
| Coin Change (min coins) | ⭐⭐⭐⭐⭐ | Medium | [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md) | 322 | The single most common Amazon DP OA. Usually reskinned as "minimum number of denominations / packages". |
| House Robber | ⭐⭐⭐⭐ | Medium | [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md) | 198 | Framed as "max value from non-adjacent shelves / warehouses". II (circular) appears as a follow-up. |
| Word Break | ⭐⭐⭐⭐ | Medium | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 139 | Rising fast 2024-2026. Given as "can this product code be split into valid SKUs". |
| Longest Palindromic Substring | ⭐⭐⭐ | Medium | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 5 | Often allows expand-around-center; the OA grader accepts O(n²) either way. |
| Maximal Square / Minimum Path Sum | ⭐⭐⭐ | Medium | [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md) | 221 / 64 | Grid DP framed as "largest fulfilment region" or "cheapest robot path through a grid". |

---

## What the auto-grader is really testing

1. **Did you beat exponential time?** Coin Change brute-force recursion is `O(amount^coins)`; you need `O(amount · coins)`.
2. **Sentinel / overflow handling.** Coin Change min must return `-1` when unreachable. The common bug is using `Integer.MAX_VALUE` then doing `min + 1` and overflowing. Use `amount + 1` as the "infinity" sentinel.
3. **Edge inputs the hidden tests hammer:** `amount = 0` → `0`; empty word/dictionary; single-cell grid; all-equal-value robber.

---

## Follow-ups Amazon adds in the OA or the phone screen

| Base problem | Common follow-up | What it forces |
|--------------|------------------|----------------|
| Coin Change | "Now **count** the number of ways" | Switch to Coin Change II — loop order flips (coins outer, amount inner) to avoid double counting. |
| House Robber | "Houses are in a **circle**" | House Robber II — run the linear solver twice (exclude first / exclude last) and take the max. |
| Word Break | "Return **one valid sentence**" / "return **all** sentences" | Word Break II — backtracking with memo on the suffix to avoid re-exploring. |
| Longest Palindromic Substring | "Return the **count** of palindromic substrings" | Palindromic Substrings (LC 647) — same expand-around-center, accumulate count. |
| Minimum Path Sum | "Robot may also move **diagonally**" | Extra transition term in the recurrence; same O(m·n). |

---

## Amazon Leadership Principle alignment

The OA is auto-graded, but the **bar-raiser debrief** of any onsite that follows scores how you reasoned. Map your DP narration to LPs:

| Leadership Principle | How to demonstrate it on a DP problem |
|----------------------|----------------------------------------|
| **Dive Deep** | Derive the recurrence out loud from a brute-force recursion; show the overlapping subproblems before caching. |
| **Insist on the Highest Standards** | Call out the `-1`/overflow edge cases *before* the interviewer asks. |
| **Bias for Action** | Start coding the recursion immediately, then memoize — don't stall hunting for the optimal tabulation first. |
| **Invent and Simplify** | Reduce the 2D table to a 1D rolling array and explain the space win. |

---

## One-glance recurrences (memorize before the OA)

- **Coin Change (min):** `dp[a] = min(dp[a], dp[a - coin] + 1)` for each coin; init `dp[0]=0`, rest `amount+1`.
- **House Robber:** `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.
- **Word Break:** `dp[i] = OR over j<i of (dp[j] && dict.contains(s[j..i]))`.
- **Longest Palindromic Substring:** `dp[i][j] = (s[i]==s[j]) && (j-i<2 || dp[i+1][j-1])`.
- **Minimum Path Sum:** `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.

For full Java with reasoning and edge cases, see [Interview Problems → Amazon](../Interview%20Problems/Amazon.md).

> **Last Updated:** 2026-06-26
