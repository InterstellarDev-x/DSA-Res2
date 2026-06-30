> **Topic:** [Dynamic Programming](../README.md) · **Section:** OA-Qns

# Goldman Sachs — DP Online Assessment Questions

Goldman Sachs runs its OA on **HackerRank** with a strong tilt toward problems with a **financial / optimization narrative**: maximizing returns, scheduling non-overlapping deals, allocating capital across denominations. The DP set is small and classic — House Robber, Coin Change, and the Stock problems dominate. Difficulty is *medium*, but the grader is strict on time limits, so memoization or tabulation is mandatory.

See also: [Amazon](./Amazon.md) · [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Adobe](./Adobe.md) · [Topic README](../README.md)

---

## Frequency Table

| Problem | Frequency | Difficulty | Pattern | LC | Source note |
|---------|-----------|------------|---------|----|-------------|
| House Robber | ⭐⭐⭐⭐ | Medium | [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md) | 198 | Framed as "maximize profit from non-adjacent investments / trades". Natural fit for the finance story. |
| Coin Change (min coins) | ⭐⭐⭐ | Medium | [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md) | 322 | "Minimum number of bills to settle an amount" — the denomination framing. |
| Best Time to Buy and Sell Stock | ⭐⭐⭐ | Easy | [Stock Buy Sell](../Patterns/Stock%20Buy%20Sell.md) | 121 | The natural GS problem. Single-transaction max profit; follow-up extends to multiple transactions and fees. |

---

## What the auto-grader is really testing

1. **Single pass over the obvious nested loop.** Best Time to Buy/Sell I is O(n) with a running min — a naive O(n²) compare-all-pairs TLEs on the large GS test set.
2. **Non-negativity / no-trade case.** Stock I must return `0` when prices only fall.
3. **Sentinel handling** in Coin Change (`-1` for unreachable).
4. **Circular follow-up** on House Robber (House Robber II).

---

## Follow-ups Goldman Sachs adds

| Base problem | Follow-up | What it forces |
|--------------|-----------|----------------|
| House Robber | "Investments arranged in a **circle**" | House Robber II — two linear passes. |
| Best Time to Buy/Sell | "Allow **unlimited** transactions" → "**at most 2**" → "**at most k**" → "**with a fee/cooldown**" | The full [Stock Buy Sell](../Patterns/Stock%20Buy%20Sell.md) state machine. |
| Coin Change | "Count ways" / "largest denomination usage" | Coin Change II / greedy-vs-DP discussion. |

---

## One-glance recurrences

- **House Robber:** `dp[i] = max(dp[i-1], dp[i-2] + nums[i])`.
- **Coin Change (min):** `dp[a] = min(dp[a], dp[a-coin] + 1)`.
- **Stock I (single transaction):** track `minPrice`; `profit = max(profit, price - minPrice)`.
- **Stock II (unlimited):** sum every positive `prices[i] - prices[i-1]`.

These intersect with [Amazon](./Amazon.md) (Coin Change, House Robber) and the broader stock progression. For deep dives, the closest write-ups are [Interview Problems → Amazon](../Interview%20Problems/Amazon.md) (House Robber, Coin Change).

> **Last Updated:** 2026-06-26
