> **Topic:** [Dynamic Programming](../README.md) · **Section:** OA-Qns

# Adobe — DP Online Assessment Questions

Adobe's OA keeps DP at the **gentle end**: warm-up counting (Climbing Stairs, Unique Paths) and one string DP (Longest Palindromic Substring). The intent is to confirm you recognize a DP and can write the recurrence cleanly, not to stress interval DP or O(n log n) tricks. Adobe often pairs the OA with a short "explain your approach" prompt, so a crisp recurrence statement matters as much as passing tests.

See also: [Amazon](./Amazon.md) · [Google](./Google.md) · [Microsoft](./Microsoft.md) · [Goldman Sachs](./Goldman%20Sachs.md) · [Topic README](../README.md)

---

## Frequency Table

| Problem | Frequency | Difficulty | Pattern | LC | Source note |
|---------|-----------|------------|---------|----|-------------|
| Climbing Stairs | ⭐⭐⭐ | Easy | [1D DP (Fibonacci Style)](../Patterns/1D%20DP%20(Fibonacci%20Style).md) | 70 | Classic warm-up. Sometimes the "ways to tile a 2×n board" rephrasing (same Fibonacci recurrence). |
| Unique Paths | ⭐⭐⭐ | Medium | [2D DP (Grids)](../Patterns/2D%20DP%20(Grids).md) | 62 | "Number of ways a cursor reaches the bottom-right of a canvas grid." |
| Longest Palindromic Substring | ⭐⭐⭐ | Medium | [DP on Strings](../Patterns/DP%20on%20Strings.md) | 5 | Expand-around-center accepted; O(n²) DP also fine for the OA input sizes. |

---

## What the auto-grader is really testing

1. **Recognition.** Both Climbing Stairs and Unique Paths are "secretly the same" — a counting recurrence with two incoming transitions. Adobe wants to see you spot that.
2. **Substring vs subsequence.** Longest Palindromic *Substring* must be contiguous; a common failed submission solves the subsequence version by mistake.
3. **Single-character / empty edge cases.** `"a"` is a palindrome of length 1; `""` returns `""`.

---

## Follow-ups Adobe adds

| Base problem | Follow-up | What it forces |
|--------------|-----------|----------------|
| Climbing Stairs | "Steps of size 1, 2, or 3" | Tribonacci-style three-term recurrence. |
| Unique Paths | "Obstacles block some cells" | Unique Paths II. |
| Unique Paths | "Minimize the path *cost*" | Minimum Path Sum (switch from counting to minimizing). |
| Longest Palindromic Substring | "Count all palindromic substrings" | Palindromic Substrings (LC 647). |

---

## One-glance recurrences

- **Climbing Stairs:** `dp[i] = dp[i-1] + dp[i-2]`.
- **Unique Paths:** `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.
- **Longest Palindromic Substring (expand):** for each center (and each gap), expand while `s[l]==s[r]`; track the longest span.

These overlap directly with [Microsoft](./Microsoft.md) (Climbing Stairs, Unique Paths) and [Amazon](./Amazon.md) (Longest Palindromic Substring). For full Java, see [Interview Problems → Amazon](../Interview%20Problems/Amazon.md) (Longest Palindromic Substring) and [Interview Problems → Microsoft](../Interview%20Problems/Microsoft.md) (Unique Paths).

> **Last Updated:** 2026-06-26
