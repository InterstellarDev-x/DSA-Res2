> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Problems

# Google — DP Interview Deep Dives

Google interviews probe *why* a recurrence is correct, not just whether you remember it. These four — Edit Distance, Burst Balloons, Regex Matching, Number of LIS — are the ones where the derivation is the interview. Each includes the state, a traced recurrence, base cases, full Java, complexity, and the level (L4/L5) at which it typically appears.

See also: [OA-Qns → Google](../OA-Qns/Google.md) · [Interview Problems → Amazon](./Amazon.md) · [Interview Problems → Microsoft](./Microsoft.md) · [Topic README](../README.md)

---

## 1. Edit Distance — three transitions, traced (L4)

**Problem.** Minimum number of insert / delete / replace operations to convert `word1` into `word2`.

### State and recurrence

- **State:** `dp[i][j]` = min operations to convert `word1[0..i)` into `word2[0..j)`.
- **Base cases:** `dp[0][j] = j` (insert `j` chars into empty), `dp[i][0] = i` (delete `i` chars).
- **Recurrence:**
  - If `word1[i-1] == word2[j-1]`: characters align for free → `dp[i][j] = dp[i-1][j-1]`.
  - Else `dp[i][j] = 1 + min(`
    - `dp[i-1][j]`   → **delete** `word1[i-1]`,
    - `dp[i][j-1]`   → **insert** `word2[j-1]`,
    - `dp[i-1][j-1]` → **replace** `word1[i-1]` with `word2[j-1]` `)`.

### Why exactly these three transitions

Aligning the suffixes ends in one of three ways: the last char of `word1` is consumed (delete), the last char of `word2` is produced (insert), or both last chars are paired (match or replace). There is no fourth option — this exhaustiveness is the proof the interviewer wants.

**Trace** `word1="horse"`, `word2="ros"` → answer 3 (replace `h→r`, delete `r`, delete `e`). The dp table fills row by row; `dp[5][3] = 3`.

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 0; i <= m; i++) dp[i][0] = i;
    for (int j = 0; j <= n; j++) dp[0][j] = j;
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = 1 + Math.min(dp[i - 1][j - 1],
                              Math.min(dp[i - 1][j], dp[i][j - 1]));
            }
        }
    }
    return dp[m][n];
}
```

Time `O(m·n)`, space `O(m·n)`.

### Space optimization to O(n)

Each row needs only the previous row plus the diagonal. Keep two rows (or one row + a saved diagonal):

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[] prev = new int[n + 1];
    for (int j = 0; j <= n; j++) prev[j] = j;
    for (int i = 1; i <= m; i++) {
        int[] cur = new int[n + 1];
        cur[0] = i;
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                cur[j] = prev[j - 1];
            } else {
                cur[j] = 1 + Math.min(prev[j - 1], Math.min(prev[j], cur[j - 1]));
            }
        }
        prev = cur;
    }
    return prev[n];
}
```

Space `O(n)`.

### Common follow-ups
- "Only insert/delete allowed" → `m + n - 2·LCS(word1, word2)`.
- "Each op has a different cost" → replace the `1 +` with op-specific weights.
- "Reconstruct the edit script" → backtrack through the table.

---

## 2. Burst Balloons — the "last balloon to burst" reframing (L5)

**Problem.** Balloons with values `nums`. Bursting balloon `i` yields `nums[left]·nums[i]·nums[right]` where `left`/`right` are the *current* neighbors. Maximize total coins.

### Why the obvious recursion fails

If you ask "which balloon to burst **first** in range `(i, j)`?", the moment you burst it the two halves are no longer independent — the neighbors across the gap change, so subproblems overlap in a way you can't combine. The fix is to ask **"which balloon `k` bursts *last*?"** When `k` bursts last in the open interval `(i, j)`, its neighbors are exactly the fixed boundaries `i` and `j` (every other balloon inside is already gone). That makes the left part `(i, k)` and right part `(k, j)` fully independent.

### State and recurrence (pad the array with 1s at both ends)

- **Padded array:** `arr = [1] + nums + [1]`.
- **State:** `dp[i][j]` = max coins from bursting all balloons strictly between indices `i` and `j` (exclusive).
- **Recurrence:** `dp[i][j] = max over k in (i, j) of dp[i][k] + arr[i]·arr[k]·arr[j] + dp[k][j]`.
- **Base:** `dp[i][j] = 0` when `j - i < 2` (nothing strictly between).
- **Order:** increasing interval length, since `dp[i][j]` needs shorter `dp[i][k]` and `dp[k][j]`.

```java
public int maxCoins(int[] nums) {
    int n = nums.length;
    int[] arr = new int[n + 2];
    arr[0] = arr[n + 1] = 1;
    for (int i = 0; i < n; i++) arr[i + 1] = nums[i];

    int[][] dp = new int[n + 2][n + 2];
    for (int len = 2; len <= n + 1; len++) {          // interval length (exclusive ends)
        for (int i = 0; i + len <= n + 1; i++) {
            int j = i + len;
            for (int k = i + 1; k < j; k++) {         // k bursts LAST in (i, j)
                int coins = arr[i] * arr[k] * arr[j] + dp[i][k] + dp[k][j];
                dp[i][j] = Math.max(dp[i][j], coins);
            }
        }
    }
    return dp[0][n + 1];
}
```

Time `O(n³)`, space `O(n²)`. This is the canonical [Partition DP (MCM)](../Patterns/Partition%20DP%20(MCM).md) skeleton.

### Common follow-ups
- "Min cost to cut a stick" (LC 1547) — same interval DP, minimize instead.
- "Why not memoized recursion?" — equivalent; `dp(i,j)` with `Integer[][]` memo and the same "last to burst" loop.

---

## 3. Regex Matching — the `*` zero-or-more case (L4/L5)

**Problem.** Implement matching for `.` (any single char) and `*` (zero or more of the *preceding* element). Match must cover the *entire* input.

### State and recurrence

- **State:** `dp[i][j]` = does `s[0..i)` match pattern `p[0..j)`?
- **Base:** `dp[0][0] = true`. For an empty string, a pattern like `a*b*c*` can still match → `dp[0][j] = dp[0][j-2]` when `p[j-1] == '*'`.
- **Recurrence:**
  - If `p[j-1]` is a normal char or `.` and matches `s[i-1]`: `dp[i][j] = dp[i-1][j-1]`.
  - If `p[j-1] == '*'`:
    - **zero of the preceding element:** `dp[i][j] = dp[i][j-2]`.
    - **one or more:** if `p[j-2]` matches `s[i-1]` (`p[j-2]=='.'` or `==s[i-1]`): `dp[i][j] |= dp[i-1][j]`.

### Why `*` has two branches

`x*` can stand for **nothing** (drop both `x` and `*`, look back 2 in the pattern) or for **at least one** `x` (consume one char of `s`, keep the `x*` available, look back 1 in the string). Missing the "look back 2" branch is the #1 failure on Google's hidden tests.

```java
public boolean isMatch(String s, String p) {
    int m = s.length(), n = p.length();
    boolean[][] dp = new boolean[m + 1][n + 1];
    dp[0][0] = true;
    // empty string vs patterns like a*, a*b*, ...
    for (int j = 1; j <= n; j++) {
        if (p.charAt(j - 1) == '*') {
            dp[0][j] = dp[0][j - 2];
        }
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            char pc = p.charAt(j - 1);
            if (pc == '*') {
                dp[i][j] = dp[i][j - 2];                       // zero occurrences
                char prev = p.charAt(j - 2);
                if (prev == '.' || prev == s.charAt(i - 1)) {
                    dp[i][j] = dp[i][j] || dp[i - 1][j];       // one or more
                }
            } else if (pc == '.' || pc == s.charAt(i - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            }
        }
    }
    return dp[m][n];
}
```

Time `O(m·n)`, space `O(m·n)`. Contrast with the simpler greedy of [Wildcard Matching](../Patterns/DP%20on%20Strings.md), where `*` matches *any sequence* (no preceding-element dependency).

### Common follow-ups
- "Add `+` (one or more)" — `x+` = `xx*`; rewrite or add a branch.
- "Space optimize" — two rolling rows.

---

## 4. Number of Longest Increasing Subsequences — dual arrays (L4)

**Problem.** Return how many strictly-increasing subsequences have the maximum length.

### Why a single dp array is not enough

`length[i]` (longest LIS ending at `i`) tells you the size but not how many ways achieve it. Carry a parallel `count[i]` = number of LIS of length `length[i]` ending at `i`.

### State and recurrence

For each `i`, scan `j < i` with `nums[j] < nums[i]`:
- If `length[j] + 1 > length[i]`: we found a strictly longer chain → adopt its length and **reset** count: `length[i] = length[j] + 1; count[i] = count[j]`.
- Else if `length[j] + 1 == length[i]`: another way to reach the same length → **accumulate**: `count[i] += count[j]`.

Finally sum `count[i]` over all `i` with the global max length.

```java
public int findNumberOfLIS(int[] nums) {
    int n = nums.length;
    int[] length = new int[n];
    int[] count = new int[n];
    Arrays.fill(length, 1);
    Arrays.fill(count, 1);
    int maxLen = 1;
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[j] < nums[i]) {
                if (length[j] + 1 > length[i]) {
                    length[i] = length[j] + 1;
                    count[i] = count[j];           // reset to the new winner's count
                } else if (length[j] + 1 == length[i]) {
                    count[i] += count[j];          // tie → accumulate
                }
            }
        }
        maxLen = Math.max(maxLen, length[i]);
    }
    int total = 0;
    for (int i = 0; i < n; i++) {
        if (length[i] == maxLen) total += count[i];
    }
    return total;
}
```

Time `O(n²)`, space `O(n)`. See [Longest Increasing Subsequence](../Patterns/Longest%20Increasing%20Subsequence.md) for the O(n log n) length-only variant.

### Common follow-ups
- "Just the length, O(n log n)" — patience sorting with a `tails[]` array and binary search.
- "Print one LIS" — store predecessor indices.

---

## L4 vs L5 expectations on these problems

| Problem | Typical level | What separates a hire from a strong hire |
|---------|---------------|-------------------------------------------|
| Edit Distance | L4 | Strong hire derives the three transitions *and* space-optimizes unprompted. |
| Number of LIS | L4 | Strong hire explains the reset-vs-accumulate logic crisply. |
| Regex Matching | L4/L5 | Strong hire nails the empty-string base row and both `*` branches. |
| Burst Balloons | L5 | Hire bar is the "last balloon" reframing *without* a hint. |

> **Last Updated:** 2026-06-26
