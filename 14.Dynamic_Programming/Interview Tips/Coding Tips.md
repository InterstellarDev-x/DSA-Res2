> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 1 of 4**

# Coding Tips — Dynamic Programming

Ten habits that make DP problems fast to write and hard to get wrong in an interview. Read alongside [Common Mistakes](./Common%20Mistakes.md), [State Design](./State%20Design.md), and [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Use the 5-step DP framework, out loud

Every DP problem yields to the same five questions. Narrate them — interviewers grade the process:

1. **State** — what does `dp[...]` mean? ("min coins to make amount `a`")
2. **Recurrence** — how does a state combine smaller states?
3. **Base case(s)** — the smallest states you can fill directly.
4. **Iteration order** — the order that guarantees dependencies are ready.
5. **Answer** — which cell holds the final result?

If you can answer all five, the code writes itself. If you're stuck, you're missing one of them.

## 2. Start with recursion, then memoize

Write the brute-force recursion first — it makes the state and recurrence obvious. Then add a memo. This top-down path is faster to *derive* than jumping to tabulation, and it lets you show the overlapping subproblems explicitly.

```java
// 1) brute force
int solve(int i, int cap) { ... }
// 2) add memo, same body
Integer[][] memo;
int solve(int i, int cap) {
    if (memo[i][cap] != null) return memo[i][cap];
    ...
    return memo[i][cap] = result;
}
```

## 3. Initialize memo to a sentinel that can't be a real answer

For `int[][]`, use `Arrays.fill(memo, -1)` (and `Arrays.fill(row, -1)` per row for 2D) **only when `-1` is not a valid answer**. When `0` is a legitimate result (counting, boolean), prefer `Integer[]`/`Boolean[]` and test for `null`. This avoids the "0 collides with not-computed" bug — see [Common Mistakes](./Common%20Mistakes.md).

```java
int[][] memo = new int[n][cap + 1];
for (int[] row : memo) Arrays.fill(row, -1);
```

## 4. Use 1-indexed dp arrays to kill boundary checks

Sizing `dp` as `(n+1)` (or `(m+1)×(n+1)`) and treating index 0 as the empty base case removes `if (i-1 >= 0)` clutter from the inner loop. Edit Distance, LCS, Knapsack, and Maximal Square are all cleanest this way.

```java
int[][] dp = new int[m + 1][n + 1];   // row 0 / col 0 are the empty-string base cases
```

## 5. Space-optimize: 2 rows → rolling, then → single array

Most 2D grid/string DP only reads the previous row (and sometimes the diagonal). Collapse to two 1D arrays, then often to one (iterate carefully to not clobber values you still need).

```java
int[] prev = new int[n + 1], cur = new int[n + 1];
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) cur[j] = f(prev[j], prev[j-1], cur[j-1]);
    prev = cur.clone();   // or swap references
}
```

## 6. Know the knapsack loop-order rule cold

- **0/1 knapsack (each item once):** iterate capacity **descending** in the 1D form, so each item is used at most once.
- **Unbounded (item reusable, e.g. Coin Change II):** iterate capacity **ascending**, so an item can be reused.

```java
// 0/1: capacity DESCENDING
for (int item : items)
    for (int c = cap; c >= item; c--) dp[c] = ...;
// unbounded: capacity ASCENDING
for (int item : items)
    for (int c = item; c <= cap; c++) dp[c] += dp[c - item];
```

See [Knapsack (Subset Sum)](../Patterns/Knapsack%20(Subset%20Sum).md).

## 7. Encode multi-dimensional state into a HashMap key when the array is sparse/huge

When dimensions are large or unbounded (e.g. remaining sum can be negative, or state is `(i, j, k)`), a dense array wastes memory. Encode the tuple into a single key:

```java
Map<Long, Integer> memo = new HashMap<>();
long key = ((long) i << 20) | ((long) j << 10) | k;   // pack, mind the bit widths
```

Use a `long` and verify the shift widths exceed each dimension's max so packing is collision-free.

## 8. Precompute palindrome tables before partition DP

For Palindrome Partitioning II and similar, precompute `isPal[i][j]` in O(n²) first, then the partition DP reads it in O(1). Doing the palindrome check inside the partition loop turns O(n²) into O(n³).

```java
boolean[][] isPal = new boolean[n][n];
for (int i = n - 1; i >= 0; i--)
    for (int j = i; j < n; j++)
        isPal[i][j] = s.charAt(i) == s.charAt(j) && (j - i < 2 || isPal[i + 1][j - 1]);
```

## 9. Reach for binary search to make LIS O(n log n)

The O(n²) `dp[i]` LIS is the default, but the moment the interviewer says "can you do better?", switch to the patience-sorting `tails[]` array with binary search:

```java
int lengthOfLIS(int[] nums) {
    int[] tails = new int[nums.length];
    int size = 0;
    for (int x : nums) {
        int lo = 0, hi = size;
        while (lo < hi) {                 // first index with tails[idx] >= x
            int mid = (lo + hi) >>> 1;
            if (tails[mid] < x) lo = mid + 1; else hi = mid;
        }
        tails[lo] = x;
        if (lo == size) size++;
    }
    return size;
}
```

See [Longest Increasing Subsequence](../Patterns/Longest%20Increasing%20Subsequence.md).

## 10. Draw the table for any 2D DP

Before coding a grid/string DP, sketch the table, mark the base row/column, draw the arrows showing which neighbors each cell reads, and pick a fill order that satisfies them. Five minutes of drawing prevents the off-by-one and iteration-order bugs that dominate [Common Mistakes](./Common%20Mistakes.md).

---

## Quick reference

| Tip | One-liner |
|-----|-----------|
| 1 | State → recurrence → base → order → answer. |
| 2 | Recursion first, then memo. |
| 3 | Sentinel must not collide with a real answer. |
| 4 | 1-index dp to drop boundary checks. |
| 5 | Two rows, then one. |
| 6 | 0/1 = descending capacity; unbounded = ascending. |
| 7 | Pack `(i,j,k)` into a `long` key for HashMap memo. |
| 8 | Precompute `isPal[][]` before partition DP. |
| 9 | LIS → `tails[]` + binary search for O(n log n). |
| 10 | Draw the table, mark base, draw the arrows. |

> **Last Updated:** 2026-06-26
