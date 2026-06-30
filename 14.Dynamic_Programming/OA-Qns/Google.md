> **Topic:** [Dynamic Programming](../README.md) · **Section:** OA-Qns

# Google — DP Online Assessment Questions

Google's screening DP is **harder and more derivation-heavy** than the typical OA. Where Amazon rewards a memorized template, Google rewards understanding *why* the recurrence is correct. The signature is **Edit Distance** and its string-DP cousins, plus **interval DP** (Burst Balloons) as an L5 differentiator. Expect the grader to push O(n²) string problems to lengths where a sloppy O(n³) or memo-key collision fails.

See also: [Amazon](./Amazon.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Adobe](./Adobe.md) · [Topic README](../README.md)

---

## Frequency Table

| Problem | Frequency | Difficulty | Pattern | LC | Source note |
|---------|-----------|------------|---------|----|-------------|
| Edit Distance | ⭐⭐⭐⭐⭐ | Hard | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 72 | **The** Google DP signature. Reskinned as "minimum operations to transform config A into config B / autocorrect cost". |
| Burst Balloons | ⭐⭐⭐⭐ | Hard | [Partition DP (MCM)](../Patterns/Partition%20DP%20(MCM).md) | 312 | L5 hard differentiator. The "last balloon to burst" reframing is what they probe. |
| Regex Matching | ⭐⭐⭐⭐ | Hard | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 10 | Pattern with `.` and `*`. The `*` zero-or-more case separates passes from fails. |
| Longest Increasing Subsequence | ⭐⭐⭐ | Medium | [Longest Increasing Subsequence](../Patterns/Longest%20Increasing%20Subsequence.md) | 300 | O(n²) accepted in OA, but follow-up demands the O(n log n) patience-sorting version. |
| Interleaving String | ⭐⭐⭐ | Medium | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 97 | 2D boolean DP over two source strings; the off-by-one base case trips people. |

---

## What the interviewer (not just the grader) is testing

1. **Recurrence justification.** For Edit Distance you must explain why the three transitions (insert / delete / replace) are exhaustive and why a match costs `0`.
2. **The reframing insight.** Burst Balloons only becomes O(n³) DP once you think "which balloon bursts *last* in `(i,j)`?" — bursting first leaves an undefined neighbor.
3. **Correct memo keys.** Multi-dimensional state (`i,j`) encoded into a HashMap key must be collision-free; the grader uses large inputs to expose `i*n+j` overflow or accidental reuse.
4. **Asymptotic upgrade on demand.** LIS at O(n²) passes the OA but the live follow-up is "can you do better?" → tails array + binary search.

---

## Follow-ups Google layers on

| Base problem | Follow-up | What it forces |
|--------------|-----------|----------------|
| Edit Distance | "Only **insertions and deletions** allowed" | Reduces to LCS-based distance: `len(a)+len(b) - 2·LCS`. |
| Edit Distance | "Reduce the space" | Two rolling rows → O(min(m,n)) space. |
| Burst Balloons | "Minimize cost of **cutting a stick**" | Same interval DP skeleton (Min Cost to Cut a Stick, LC 1547). |
| LIS | "Do it in O(n log n)" | Patience sorting / `tails[]` with binary search. |
| LIS | "**Count** the number of LIS" | Dual arrays `length[]` and `count[]`. |
| Interleaving String | "Reconstruct the interleaving" | Backtrack through the boolean DP table. |

---

## L4 vs L5 framing

- **L4 (SDE-2 equivalent):** Edit Distance / LIS expected clean, with space optimization on request.
- **L5 (Senior):** Burst Balloons / interval DP expected *without hints*, plus a crisp argument for why the interval order (length-increasing) is the only valid evaluation order.

---

## One-glance recurrences

- **Edit Distance:** match → `dp[i-1][j-1]`; else `1 + min(insert dp[i][j-1], delete dp[i-1][j], replace dp[i-1][j-1])`.
- **Burst Balloons:** `dp[i][j] = max over k in (i,j) of dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j]` on padded array.
- **Regex `*`:** zero of preceding → `dp[i][j-2]`; one-or-more → `dp[i-1][j]` if `s[i]` matches `p[j-1]`.
- **LIS (O(n²)):** `dp[i] = 1 + max(dp[j])` for `j<i, nums[j]<nums[i]`.
- **Interleaving:** `dp[i][j] = (a[i-1]==c[i+j-1] && dp[i-1][j]) || (b[j-1]==c[i+j-1] && dp[i][j-1])`.

For full Java traces and the "last balloon" derivation, see [Interview Problems → Google](../Interview%20Problems/Google.md).

> **Last Updated:** 2026-06-26
