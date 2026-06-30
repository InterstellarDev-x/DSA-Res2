> **Topic:** [Dynamic Programming](../README.md) · **Section:** OA-Qns

# Microsoft — DP Online Assessment Questions

Microsoft's OA (often delivered via Codility / HackerRank) favors **clean, classic DP with a clear story**: counting paths, decoding strings, climbing stairs. The difficulty sits at *easy-medium* with one *hard-ish* string DP. Microsoft graders reward correctness on the boundary cases (`'0'` in Decode Ways, single column grids) more than asymptotic cleverness.

See also: [Amazon](./Amazon.md) · [Google](./Google.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md) · [Topic README](../README.md)

---

## Frequency Table

| Problem | Frequency | Difficulty | Pattern | LC | Source note |
|---------|-----------|------------|---------|----|-------------|
| Climbing Stairs | ⭐⭐⭐⭐ | Easy | [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md) | 70 | The warm-up. Sometimes generalized to "steps of size 1..k" (Frog Jump K). |
| Coin Change (min coins) | ⭐⭐⭐⭐ | Medium | [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md) | 322 | Same staple as Amazon; expect the `-1` unreachable case to be tested. |
| Edit Distance | ⭐⭐⭐⭐ | Hard | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 72 | Framed as a spell-check / diff-tool cost; production framing in the onsite. |
| Decode Ways | ⭐⭐⭐ | Medium | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 91 | The `'0'` handling is the entire difficulty; many submissions fail on `"06"`, `"100"`. |
| Unique Paths | ⭐⭐⭐ | Medium | [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md) | 62 | Combinatorial closed form accepted, but DP is the expected answer in the screen. |

---

## What the auto-grader is really testing

1. **Boundary correctness over speed.** Decode Ways hidden tests are almost entirely `'0'` edge cases: leading zero, `"10"`, `"30"` (invalid), `"100"`.
2. **The `-1` sentinel** in Coin Change for unreachable amounts.
3. **1-indexed vs 0-indexed dp arrays** — Unique Paths and Edit Distance are cleanest with `n+1` sized arrays and an extra base row/column.

---

## Follow-ups Microsoft adds

| Base problem | Follow-up | What it forces |
|--------------|-----------|----------------|
| Climbing Stairs | "You can take 1..k steps" | Frog Jump K — windowed sum, O(n·k) or sliding-window O(n). |
| Coin Change | "Count ways instead of min" | Coin Change II; loop order matters. |
| Edit Distance | "Show the actual sequence of edits" | Backtrack through the dp table; reconstruct operations. |
| Decode Ways | "Letters can also be `'*'` wildcards" | Decode Ways II (LC 639) — multiply by counts of digit possibilities. |
| Unique Paths | "Some cells are blocked" | Unique Paths II — set blocked cells to 0 ways. |

---

## One-glance recurrences

- **Climbing Stairs:** `dp[i] = dp[i-1] + dp[i-2]`.
- **Coin Change (min):** `dp[a] = min(dp[a], dp[a-coin] + 1)`.
- **Edit Distance:** match → diag; else `1 + min(left, up, diag)`.
- **Decode Ways:** `dp[i] = (s[i]!='0' ? dp[i-1] : 0) + (10..26 valid ? dp[i-2] : 0)`.
- **Unique Paths:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.

For the full Decode Ways `'0'` walkthrough and the diff-tool framing of Edit Distance, see [Interview Problems → Microsoft](../Interview%20Problems/Microsoft.md).

> **Last Updated:** 2026-06-26
