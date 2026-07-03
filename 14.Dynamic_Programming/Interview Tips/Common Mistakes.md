> **Topic:** [Dynamic Programming](../README.md) · **Section:** Interview Tips · **Tips 2 of 4**

# Common Mistakes — Dynamic Programming

Twelve bugs that fail hidden tests or get flagged in interviews, each as a wrong/correct pair. Pair with [Coding Tips](./Coding%20Tips.md), [State Design](./State%20Design.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Memo sentinel collides with a valid `0`

`0` is a legitimate answer (zero coins, zero ways-as-boolean), so `-1`-init `vector<int>` is fine, but `0`-init silently returns "0" for uncomputed states.

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: default vector<int> is 0; can't tell "not computed" from "answer is 0"
vector<int> memo(n);
int solve(int i) { if (memo[i] != 0) return memo[i]; ... }

// CORRECT: use -1 sentinel (when -1 is impossible) and fill it
vector<int> memo(n, -1);
int solve(int i) { if (memo[i] != -1) return memo[i]; ... }
// or vector<optional<int>> memo(n); test memo[i].has_value() when 0 IS a valid answer
```

## 2. Wrong knapsack loop order (0/1 vs unbounded)

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: 0/1 knapsack with ASCENDING capacity reuses each item → unbounded behavior
for (auto item : items)
    for (int c = item; c <= cap; c++) dp[c] = max(dp[c], dp[c - item] + val);

// CORRECT: 0/1 needs DESCENDING capacity so each item is used once
for (auto item : items)
    for (int c = cap; c >= item; c--) dp[c] = max(dp[c], dp[c - item] + val);
```

## 3. Off-by-one in dp size (`n` vs `n+1`)

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: size n, then index dp[n] → index out of bounds; no room for empty base
vector<int> dp(n);
// CORRECT: size n+1, index 0 is the empty-prefix base case
vector<int> dp(n + 1);
dp[0] = baseValue;
```

## 4. House Robber II — forgetting the circular constraint

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: treats circular houses as linear → may rob both first and last (adjacent)
return robLinear(nums, 0, n - 1);

// CORRECT: first and last are adjacent → take the better of excluding one end
if (n == 1) return nums[0];
return max(robLinear(nums, 0, n - 2), robLinear(nums, 1, n - 1));
```

## 5. Coin Change overflow / missing sentinel

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: INT_MAX + 1 overflows and corrupts the min
fill(dp.begin(), dp.end(), INT_MAX);
dp[a] = min(dp[a], dp[a - c] + 1);

// CORRECT: amount+1 is a safe "infinity" (no real answer exceeds amount)
fill(dp.begin(), dp.end(), amount + 1);
dp[a] = min(dp[a], dp[a - c] + 1);
return dp[amount] > amount ? -1 : dp[amount];
```

## 6. LCS base case row/column not zero-initialized conceptually

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: starting the loops at 0 and reading dp[i-1]/dp[j-1] at the edge
for (int i = 0; i <= m; i++)
    for (int j = 0; j <= n; j++) dp[i][j] = dp[i-1][j-1] + 1;   // i-1 = -1 crash

// CORRECT: row 0 and col 0 are LCS=0 (empty string); real loop starts at 1
// vector<vector<int>> dp(m+1, vector<int>(n+1, 0));  // zero-initialized, row 0 / col 0 = 0
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = (a[i-1] == b[j-1])
                 ? dp[i-1][j-1] + 1
                 : max(dp[i-1][j], dp[i][j-1]);
```

## 7. Edit Distance base case (empty-string costs)

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: leaving the first row/column as 0 → claims converting "" to "abc" is free
vector<vector<int>> dp(m+1, vector<int>(n+1, 0));   // all zeros

// CORRECT: building from/to empty costs the string length
for (int i = 0; i <= m; i++) dp[i][0] = i;   // delete i chars
for (int j = 0; j <= n; j++) dp[0][j] = j;   // insert j chars
```

## 8. Decode Ways — mishandling `'0'`

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: always adds both branches, accepts "06", "30", standalone "0"
dp[i] = dp[i-1] + dp[i-2];

// CORRECT: single digit only if != '0'; two digits only if in 10..26
if (s[i-1] != '0') dp[i] += dp[i-1];
int two = (s[i-2]-'0')*10 + (s[i-1]-'0');
if (two >= 10 && two <= 26) dp[i] += dp[i-2];
```

## 9. Palindrome DP wrong iteration order

`dp[i][j]` depends on `dp[i+1][j-1]` (the inner substring), so you must fill **shorter substrings first**.

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: top-down i then j reads dp[i+1][j-1] before it's computed
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) dp[i][j] = s[i]==s[j] && dp[i+1][j-1];

// CORRECT: i descending, j ascending (or iterate by increasing length)
for (int i = n - 1; i >= 0; i--)
    for (int j = i; j < n; j++)
        dp[i][j] = s[i] == s[j] && (j - i < 2 || dp[i+1][j-1]);
```

## 10. Burst Balloons — missing boundary padding

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: bursting first / reading nums[i-1] or nums[j+1] off the ends → index errors
// and the subproblems aren't independent.

// CORRECT: pad with virtual 1s and think "which balloon k bursts LAST in (i,j)"
vector<int> arr(n + 2);
arr[0] = arr[n + 1] = 1;
for (int i = 0; i < n; i++) arr[i + 1] = nums[i];
// dp[i][j] = max over k in (i,j) of dp[i][k] + arr[i]*arr[k]*arr[j] + dp[k][j];
```

## 11. Subset Sum — negative / out-of-range index

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: blindly indexing dp[c - num] when num > c → negative index crash
dp[c] = dp[c] || dp[c - num];

// CORRECT: guard the capacity (or only loop c >= num)
if (num <= c) dp[c] = dp[c] || dp[c - num];
```

## 12. Stock state machine — wrong transition

```cpp
#include <bits/stdc++.h>
using namespace std;
// WRONG: lets you sell without ever having bought, or buy twice in a row
hold = max(hold, cash + price);   // sign / source state mixed up

// CORRECT: hold comes from (stay holding) or (buy today from cash);
//          cash comes from (stay in cash) or (sell today what you hold)
int hold = INT_MIN, cash = 0;
for (auto price : prices) {
    int prevCash = cash;
    cash = max(cash, hold + price);   // sell
    hold = max(hold, prevCash - price); // buy
}
return cash;
```

See [Stock Buy Sell](../Patterns/Stock%20Buy%20Sell.md) for the full state machine.

---

## Mistake → fix cheat sheet

| # | Mistake | Fix |
|---|---------|-----|
| 1 | `0`-init memo collides with valid 0 | `-1` sentinel or `optional<int>`/`nullopt` |
| 2 | Ascending cap for 0/1 knapsack | descending capacity |
| 3 | dp sized `n` not `n+1` | size `n+1`, base at index 0 |
| 4 | Linear robber on circular houses | two passes, exclude each end |
| 5 | `INT_MAX + 1` overflow | `amount+1` sentinel |
| 6 | LCS edge crash | loop from 1, row/col 0 = 0 |
| 7 | Edit Distance zero base row | `dp[i][0]=i`, `dp[0][j]=j` |
| 8 | Decode Ways ignores `'0'` | guard single + two-digit ranges |
| 9 | Palindrome wrong order | shorter substrings first |
| 10 | No balloon padding | pad 1s, "last to burst" |
| 11 | Negative subset index | guard `num <= c` |
| 12 | Bad stock transition | hold/cash from correct prior states |

> **Last Updated:** 2026-06-26
