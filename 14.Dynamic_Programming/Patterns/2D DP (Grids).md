> **Topic:** [Dynamic Programming](../README.md) · **Pattern 2 of 8**

# 2D DP (Grids)

Grid DP is the natural next step after [1D DP (Fibonacci Style)](./1D%20DP%20(Fibonacci%20Style).md). Instead of a single dimension of state, the answer at a cell depends on a small set of *neighbouring* cells — typically the cell above, to the left, or diagonally adjacent. The recurrence is almost always a `min`/`max`/`sum` over those neighbours.

The defining trait of this pattern is that the **state is a coordinate** (a `(row, col)` pair, sometimes augmented with a second coordinate or a small categorical variable). Because every cell only looks at the previous row (and possibly the current row to its left), almost every grid problem collapses from `O(m·n)` memory to `O(n)` using a **rolling row** — replacing the full `dp[m][n]` table with one or two 1D arrays. This space optimization is the recurring theme below, and you should reach for it instinctively once the tabulation is correct.

This document covers problems **8–14** in the DP catalogue. Each problem is presented with its state definition, recurrence, base cases, complexity, and at least one dry-run for the representative cases. All code is full, compilable C++.

---

## 1. Unique Paths (LC 62)

A robot starts at the top-left of an `m × n` grid and can only move **right** or **down**. Count the number of distinct paths to the bottom-right.

**State.** `dp[i][j]` = number of ways to reach cell `(i, j)` from `(0, 0)`.

**Recurrence.** `dp[i][j] = dp[i-1][j] + dp[i][j-1]` — you arrive at a cell either from above or from the left.

**Base cases.** The entire first row and first column have exactly one path (`dp[0][j] = dp[i][0] = 1`), since there is only one way to travel along an edge.

### Combinatorial formula

Any path consists of exactly `(m-1)` down-moves and `(n-1)` right-moves, in some order. The total number of moves is `(m-1)+(n-1) = m+n-2`, and we choose which of them are down-moves:

> **C(m + n − 2, m − 1)**

```cpp
#include <bits/stdc++.h>
using namespace std;

class UniquePathsMath {
public:
    // Choose (m-1) down moves out of (m+n-2) total moves.
    // Compute C(m+n-2, m-1) iteratively to avoid overflow / factorials.
    int uniquePaths(int m, int n) {
        int total = m + n - 2;   // total moves
        int k = m - 1;           // pick the smaller for fewer iterations
        long long result = 1;
        for (int i = 1; i <= k; i++) {
            // result *= (total - k + i) / i ; done in this order to stay integral
            result = result * (total - k + i) / i;
        }
        return (int) result;
    }
};
```

### DP grid (`O(m·n)` time, `O(m·n)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class UniquePathsGrid {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m, vector<int>(n));

        // Base cases: first column and first row each have exactly one path.
        for (int i = 0; i < m; i++) dp[i][0] = 1;
        for (int j = 0; j < n; j++) dp[0][j] = 1;

        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        return dp[m - 1][n - 1];
    }
};
```

### Space optimization: rolling row (`O(n)` space)

Row `i` only needs row `i-1`. Process left to right within a single array: at the moment we read `dp[j]` it still holds the value from the *previous* row (the cell above), and `dp[j-1]` already holds the *current* row (the cell to the left).

```cpp
#include <bits/stdc++.h>
using namespace std;

class UniquePathsRolling {
public:
    int uniquePaths(int m, int n) {
        vector<int> dp(n, 1);          // first row is all 1s

        for (int i = 1; i < m; i++) {
            // dp[0] stays 1 (first column). Start from j = 1.
            for (int j = 1; j < n; j++) {
                // dp[j]   == value from row i-1 (cell above)
                // dp[j-1] == value already updated for row i (cell left)
                dp[j] = dp[j] + dp[j - 1];
            }
        }
        return dp[n - 1];
    }
};
```

| Approach            | Time       | Space      |
| ------------------- | ---------- | ---------- |
| Combinatorial       | `O(min(m,n))` | `O(1)`  |
| DP grid             | `O(m·n)`   | `O(m·n)`   |
| Rolling row         | `O(m·n)`   | `O(n)`     |

---

## 2. Unique Paths II (LC 63 — obstacles)

Same setup as Unique Paths, but some cells contain obstacles (`grid[i][j] == 1`). An obstacle cell is unreachable, so its path count is `0`.

**State.** `dp[i][j]` = number of paths to `(i, j)` avoiding obstacles.

**Recurrence.**
- If `grid[i][j] == 1` (obstacle): `dp[i][j] = 0`.
- Otherwise: `dp[i][j] = dp[i-1][j] + dp[i][j-1]`.

**Base cases.** The start cell is `1` if it is not an obstacle. The first row/column propagate `1` only until the first obstacle is hit, after which everything downstream is `0` — this falls out naturally from the recurrence if we guard the edges.

### Rolling-row solution (`O(n)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class UniquePathsII {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        int m = obstacleGrid.size();
        int n = obstacleGrid[0].size();
        vector<int> dp(n);

        // Start cell: 1 path if it is open, else 0 (and answer is 0).
        dp[0] = (obstacleGrid[0][0] == 1) ? 0 : 1;

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (obstacleGrid[i][j] == 1) {
                    dp[j] = 0;                       // blocked cell, no paths
                } else if (j > 0) {
                    dp[j] = dp[j] + dp[j - 1];       // above + left
                }
                // j == 0 && open: dp[0] keeps its carried-down value
            }
        }
        return dp[n - 1];
    }
};
```

| Approach    | Time     | Space  |
| ----------- | -------- | ------ |
| Rolling row | `O(m·n)` | `O(n)` |

---

## 3. Minimum Path Sum (LC 64)

Given an `m × n` grid of non-negative numbers, find a path from top-left to bottom-right (moving only right or down) that minimizes the sum of values along the path.

**State.** `dp[i][j]` = minimum sum to reach `(i, j)`.

**Recurrence.** `dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1])`.

**Base cases.** `dp[0][0] = grid[0][0]`. The first row/column accumulate prefix sums (only one way in).

### Step 1 — Recursion (top-down, exponential)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinPathSumRecursion {
public:
    int minPathSum(vector<vector<int>>& grid) {
        return solve(grid, grid.size() - 1, grid[0].size() - 1);
    }

    // f(i, j) = min cost to reach (i, j) from (0, 0)
    int solve(vector<vector<int>>& grid, int i, int j) {
        if (i == 0 && j == 0) return grid[0][0];
        if (i < 0 || j < 0) return INT_MAX; // out of bounds = invalid

        int up = solve(grid, i - 1, j);
        int left = solve(grid, i, j - 1);
        return grid[i][j] + min(up, left);
    }
};
```

### Step 2 — Memoization (`vector<vector<int>> memo`, initialized to `-1`)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinPathSumMemo {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> memo(m, vector<int>(n, -1));
        return solve(grid, m - 1, n - 1, memo);
    }

    int solve(vector<vector<int>>& grid, int i, int j, vector<vector<int>>& memo) {
        if (i == 0 && j == 0) return grid[0][0];
        if (i < 0 || j < 0) return INT_MAX;
        if (memo[i][j] != -1) return memo[i][j];

        int up = solve(grid, i - 1, j, memo);
        int left = solve(grid, i, j - 1, memo);
        return memo[i][j] = grid[i][j] + min(up, left);
    }
};
```

### Step 3 — Tabulation (`O(m·n)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinPathSumTab {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<vector<int>> dp(m, vector<int>(n));

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (i == 0 && j == 0) {
                    dp[i][j] = grid[i][j];
                } else {
                    int up   = (i > 0) ? dp[i - 1][j] : INT_MAX;
                    int left = (j > 0) ? dp[i][j - 1] : INT_MAX;
                    dp[i][j] = grid[i][j] + min(up, left);
                }
            }
        }
        return dp[m - 1][n - 1];
    }
};
```

### Step 4 — Rolling-row space optimization (`O(n)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinPathSumRolling {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        vector<int> prev(n);

        for (int i = 0; i < m; i++) {
            vector<int> curr(n);
            for (int j = 0; j < n; j++) {
                if (i == 0 && j == 0) {
                    curr[j] = grid[i][j];
                } else {
                    int up   = (i > 0) ? prev[j]     : INT_MAX;
                    int left = (j > 0) ? curr[j - 1] : INT_MAX;
                    curr[j] = grid[i][j] + min(up, left);
                }
            }
            prev = curr;
        }
        return prev[n - 1];
    }
};
```

### Dry-run

Grid:

```
1 3 1
1 5 1
4 2 1
```

Filling `dp` (cumulative minimum cost):

| `dp` | j=0 | j=1 | j=2 |
| ---- | --- | --- | --- |
| i=0  | 1   | 4   | 5   |
| i=1  | 2   | 7   | 6   |
| i=2  | 6   | 8   | 7   |

- `dp[0]` = prefix sums: `1, 1+3=4, 4+1=5`.
- `dp[1][0] = 1 + dp[0][0] = 2`; `dp[1][1] = 5 + min(4, 2) = 7`; `dp[1][2] = 1 + min(5, 7) = 6`.
- `dp[2][0] = 4 + 2 = 6`; `dp[2][1] = 2 + min(7, 6) = 8`; `dp[2][2] = 1 + min(6, 8) = 7`.

Answer: `dp[2][2] = 7` (path `1 → 3 → 1 → 1 → 1`).

| Approach              | Time     | Space    |
| --------------------- | -------- | -------- |
| Recursion             | `O(2^(m+n))` | `O(m+n)` |
| Memoization           | `O(m·n)` | `O(m·n)` + stack |
| Tabulation            | `O(m·n)` | `O(m·n)` |
| Rolling row           | `O(m·n)` | `O(n)`   |

---

## 4. Triangle (LC 120)

Given a triangle (a list of rows, where row `i` has `i+1` elements), find the minimum path sum from top to bottom. From index `j` in a row you may move to index `j` or `j+1` in the next row.

**State (bottom-up).** `dp[j]` = minimum path sum from cell `(i, j)` down to the bottom row.

**Recurrence (processing rows from bottom to top).** `dp[j] = triangle[i][j] + min(dp[j], dp[j+1])`.

**Base case.** The last row is the answer for itself: `dp[j] = triangle[last][j]`.

Processing bottom-up lets us reuse a single 1D array (`O(n)`), and the answer is `dp[0]`.

### `O(n)` space, in-place over a 1D array

```cpp
#include <bits/stdc++.h>
using namespace std;

class TriangleMinPath {
public:
    int minimumTotal(vector<vector<int>>& triangle) {
        int n = triangle.size();
        // Initialize dp with the last row.
        vector<int> dp(n);
        auto& last = triangle[n - 1];
        for (int j = 0; j < n; j++) dp[j] = last[j];

        // Move upward, collapsing each row into dp.
        for (int i = n - 2; i >= 0; i--) {
            auto& row = triangle[i];
            for (int j = 0; j <= i; j++) {
                dp[j] = row[j] + min(dp[j], dp[j + 1]);
            }
        }
        return dp[0];
    }
};
```

### Dry-run

Triangle:

```
   2
  3 4
 6 5 7
4 1 8 3
```

- Init `dp` from last row: `[4, 1, 8, 3]`.
- Row `6 5 7`: `dp[0]=6+min(4,1)=7`, `dp[1]=5+min(1,8)=6`, `dp[2]=7+min(8,3)=10` → `[7, 6, 10, 3]`.
- Row `3 4`: `dp[0]=3+min(7,6)=9`, `dp[1]=4+min(6,10)=10` → `[9, 10, ...]`.
- Row `2`: `dp[0]=2+min(9,10)=11`.

Answer: `11` (path `2 → 3 → 5 → 1`).

| Approach          | Time     | Space  |
| ----------------- | -------- | ------ |
| Bottom-up 1D DP   | `O(n²)`  | `O(n)` |

---

## 5. Minimum Falling Path Sum (LC 931)

Given an `n × n` matrix, a falling path starts at any cell in row 0 and chooses, at each step, the cell **directly below**, **below-left**, or **below-right**. Return the minimum sum of such a path.

**State.** `dp[i][j]` = minimum falling-path sum ending at cell `(i, j)`.

**Recurrence.** `dp[i][j] = matrix[i][j] + min(dp[i-1][j-1], dp[i-1][j], dp[i-1][j+1])`, with out-of-range diagonals treated as `+∞`.

**Base case.** `dp[0][j] = matrix[0][j]`. Answer = `min` over the last row.

### Rolling-row solution (`O(n)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class MinFallingPathSum {
public:
    int minFallingPathSum(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<int> prev(n);
        for (int j = 0; j < n; j++) prev[j] = matrix[0][j]; // base: first row

        for (int i = 1; i < n; i++) {
            vector<int> curr(n);
            for (int j = 0; j < n; j++) {
                int up        = prev[j];
                int upLeft    = (j > 0)     ? prev[j - 1] : INT_MAX;
                int upRight   = (j < n - 1) ? prev[j + 1] : INT_MAX;
                int best = min(up, min(upLeft, upRight));
                curr[j] = matrix[i][j] + best;
            }
            prev = curr;
        }

        int answer = INT_MAX;
        for (int j = 0; j < n; j++) answer = min(answer, prev[j]);
        return answer;
    }
};
```

| Approach    | Time     | Space  |
| ----------- | -------- | ------ |
| Tabulation  | `O(n²)`  | `O(n²)`|
| Rolling row | `O(n²)`  | `O(n)` |

---

## 6. Ninja's Training (LC 1186-style / GFG)

A ninja trains over `n` days. Each day offers three activities with point values `points[day][0..2]`, but the **same activity cannot be performed on two consecutive days**. Maximize total points.

**State.** `dp[day][last]` = maximum points obtainable from `day` onward, given that the activity chosen on `day+1`'s perspective — i.e. `last` is the activity performed on the *previous* day (`last ∈ {0,1,2}`, plus a sentinel `3` meaning "no restriction" on day 0).

**Recurrence.** On `day`, try every activity `task != last`:

> `dp[day][last] = max over task != last of ( points[day][task] + f(day + 1, task) )`

**Base case.** Past the last day there are no more points: `f(n, last) = 0`.

### Recursion + memoization (`vector<vector<int>> memo`, initialized to `-1`)

```cpp
#include <bits/stdc++.h>
using namespace std;

class NinjaTrainingMemo {
public:
    int maximumPoints(vector<vector<int>>& points, int n) {
        // last in {0,1,2} = activity done previous day; 3 = none (day 0)
        vector<vector<int>> memo(n, vector<int>(4, -1));
        return solve(0, 3, points, n, memo);
    }

    int solve(int day, int last, vector<vector<int>>& points, int n, vector<vector<int>>& memo) {
        if (day == n) return 0;
        if (memo[day][last] != -1) return memo[day][last];

        int best = 0;
        for (int task = 0; task < 3; task++) {
            if (task == last) continue;
            int pointsToday = points[day][task] + solve(day + 1, task, points, n, memo);
            best = max(best, pointsToday);
        }
        return memo[day][last] = best;
    }
};
```

### Tabulation (`O(n)` rows, `O(n·4)` space)

```cpp
#include <bits/stdc++.h>
using namespace std;

class NinjaTrainingTab {
public:
    int maximumPoints(vector<vector<int>>& points, int n) {
        vector<vector<int>> dp(n + 1, vector<int>(4, 0));
        // dp[n][*] = 0 (base case, already zero-initialized)

        for (int day = n - 1; day >= 0; day--) {
            for (int last = 0; last < 4; last++) {
                int best = 0;
                for (int task = 0; task < 3; task++) {
                    if (task == last) continue;
                    best = max(best, points[day][task] + dp[day + 1][task]);
                }
                dp[day][last] = best;
            }
        }
        return dp[0][3];
    }
};
```

### Space optimization to two rows (`O(1)` extra, just 4 ints)

Since `dp[day]` depends only on `dp[day+1]`, we keep a single `next[4]` array (the "two rows" being `next` and the `curr` we build).

```cpp
#include <bits/stdc++.h>
using namespace std;

class NinjaTrainingSpaceOpt {
public:
    int maximumPoints(vector<vector<int>>& points, int n) {
        vector<int> next(4, 0); // dp for day+1, base = all zeros

        for (int day = n - 1; day >= 0; day--) {
            vector<int> curr(4);
            for (int last = 0; last < 4; last++) {
                int best = 0;
                for (int task = 0; task < 3; task++) {
                    if (task == last) continue;
                    best = max(best, points[day][task] + next[task]);
                }
                curr[last] = best;
            }
            next = curr;
        }
        return next[3]; // started with last = 3 (no restriction)
    }
};
```

### Dry-run

`points`:

```
day 0:  1 2 5
day 1:  3 1 1
day 2:  3 3 3
```

Working bottom-up (`next` = day+1 values, indexed by `last`):

- After day 2 (`next` was all 0): for each `last`, pick best `task != last` of `points[2][task]` → all give `3`. So `curr = [3, 3, 3, 3]`, becomes `next`.
- Day 1 (`points[1] = 3,1,1`): `last=0` → max(1+3, 1+3) = 4; `last=1` → max(3+3, 1+3) = 6; `last=2` → max(3+3, 1+3) = 6; `last=3` → max(3+3, 1+3, 1+3) = 6. `curr = [4, 6, 6, 6]`.
- Day 0 (`points[0] = 1,2,5`), `last=3`: max(1+next[0], 2+next[1], 5+next[2]) = max(1+4, 2+6, 5+6) = `11`.

Answer: `11` (day0: activity 2 = 5, day1: activity 0 = 3, day2: activity 0/1 = 3 → 5+3+3 = 11).

| Approach            | Time    | Space  |
| ------------------- | ------- | ------ |
| Memoization         | `O(n·4·3)` = `O(n)` | `O(n·4)` |
| Tabulation          | `O(n)`  | `O(n·4)` |
| Space optimized     | `O(n)`  | `O(1)` (4 ints) |

---

## 7. Cherry Pickup II (LC 1463 — two robots)

Given a `rows × cols` grid of cherry counts, **two robots** start at the top of column `0` and column `cols-1` respectively. Each robot moves down one row per step, and within a step may shift its column by `-1, 0, +1`. They collect the cherries in the cells they occupy (counting a shared cell only once). Maximize total cherries collected when both reach the bottom row.

### Why the two robots are processed row-synchronously

The key modelling insight: **both robots are always on the same row**. They each take exactly one downward step per turn, so after `k` steps both are on row `k`. This lets us use a *single* row index `row` for both robots instead of tracking two independent positions in time. The full state is therefore `(row, col1, col2)` — the shared row plus each robot's column. If they were allowed to move at different speeds, this synchronization would break and the state would explode; the simultaneous-descent rule is exactly what keeps the state to 3 dimensions.

**State.** `dp[row][col1][col2]` = maximum cherries collected from `row` downward, given robot 1 is at `col1` and robot 2 is at `col2`.

**Recurrence.** Collect the current row's cherries (avoid double-counting if `col1 == col2`), then try all **9 transition combinations** (each robot independently moves `-1, 0, +1`):

> `dp[row][c1][c2] = cherries(row, c1, c2) + max over (d1, d2) in {-1,0,1}² of dp[row+1][c1+d1][c2+d2]`

where `cherries(row, c1, c2) = grid[row][c1] + (c1 == c2 ? 0 : grid[row][c2])`.

**Base case.** On the last row, `dp[rows-1][c1][c2] = cherries(rows-1, c1, c2)`.

**Initial answer.** `f(0, 0, cols-1)`.

### Memoization — 3D (`vector<vector<vector<int>>> memo`, init to a sentinel)

```cpp
#include <bits/stdc++.h>
using namespace std;

class CherryPickupIIMemo {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<vector<int>>> memo(rows, vector<vector<int>>(cols, vector<int>(cols, -1)));
        return solve(0, 0, cols - 1, grid, rows, cols, memo);
    }

    int solve(int row, int c1, int c2, vector<vector<int>>& grid,
              int rows, int cols, vector<vector<vector<int>>>& memo) {
        // Out of bounds is invalid -> treat as impossible.
        if (c1 < 0 || c1 >= cols || c2 < 0 || c2 >= cols) return INT_MIN;
        if (memo[row][c1][c2] != -1) return memo[row][c1][c2];

        // Cherries on the current row (count shared cell once).
        int collected = grid[row][c1];
        if (c1 != c2) collected += grid[row][c2];

        if (row == rows - 1) return memo[row][c1][c2] = collected;

        // Explore all 9 (d1, d2) combinations for the next row.
        int best = INT_MIN;
        for (int d1 = -1; d1 <= 1; d1++) {
            for (int d2 = -1; d2 <= 1; d2++) {
                int nxt = solve(row + 1, c1 + d1, c2 + d2, grid, rows, cols, memo);
                best = max(best, nxt);
            }
        }
        return memo[row][c1][c2] = collected + best;
    }
};
```

### Tabulation (full 3D table)

```cpp
#include <bits/stdc++.h>
using namespace std;

class CherryPickupIITab {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<vector<int>>> dp(rows, vector<vector<int>>(cols, vector<int>(cols)));

        // Base row: last row.
        for (int c1 = 0; c1 < cols; c1++) {
            for (int c2 = 0; c2 < cols; c2++) {
                dp[rows - 1][c1][c2] = cherries(grid, rows - 1, c1, c2);
            }
        }

        for (int row = rows - 2; row >= 0; row--) {
            for (int c1 = 0; c1 < cols; c1++) {
                for (int c2 = 0; c2 < cols; c2++) {
                    int best = INT_MIN;
                    for (int d1 = -1; d1 <= 1; d1++) {
                        for (int d2 = -1; d2 <= 1; d2++) {
                            int n1 = c1 + d1, n2 = c2 + d2;
                            if (n1 < 0 || n1 >= cols || n2 < 0 || n2 >= cols) continue;
                            best = max(best, dp[row + 1][n1][n2]);
                        }
                    }
                    dp[row][c1][c2] = cherries(grid, row, c1, c2) + best;
                }
            }
        }
        return dp[0][0][cols - 1];
    }

    int cherries(vector<vector<int>>& grid, int row, int c1, int c2) {
        return (c1 == c2) ? grid[row][c1] : grid[row][c1] + grid[row][c2];
    }
};
```

### Space optimization to two 2D layers (`O(cols²)` space)

`dp[row]` depends only on `dp[row+1]`, so we keep one `next` 2D layer (`cols × cols`) and build the current layer from it — the "two layers" of the optimization.

```cpp
#include <bits/stdc++.h>
using namespace std;

class CherryPickupIISpaceOpt {
public:
    int cherryPickup(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        vector<vector<int>> next(cols, vector<int>(cols));

        // Base: last row.
        for (int c1 = 0; c1 < cols; c1++)
            for (int c2 = 0; c2 < cols; c2++)
                next[c1][c2] = cherries(grid, rows - 1, c1, c2);

        for (int row = rows - 2; row >= 0; row--) {
            vector<vector<int>> curr(cols, vector<int>(cols));
            for (int c1 = 0; c1 < cols; c1++) {
                for (int c2 = 0; c2 < cols; c2++) {
                    int best = INT_MIN;
                    for (int d1 = -1; d1 <= 1; d1++) {
                        for (int d2 = -1; d2 <= 1; d2++) {
                            int n1 = c1 + d1, n2 = c2 + d2;
                            if (n1 < 0 || n1 >= cols || n2 < 0 || n2 >= cols) continue;
                            best = max(best, next[n1][n2]);
                        }
                    }
                    curr[c1][c2] = cherries(grid, row, c1, c2) + best;
                }
            }
            next = curr;
        }
        return next[0][cols - 1];
    }

    int cherries(vector<vector<int>>& grid, int row, int c1, int c2) {
        return (c1 == c2) ? grid[row][c1] : grid[row][c1] + grid[row][c2];
    }
};
```

| Approach            | Time              | Space        |
| ------------------- | ----------------- | ------------ |
| Memoization (3D)    | `O(rows·cols²·9)` | `O(rows·cols²)` |
| Tabulation (3D)     | `O(rows·cols²·9)` | `O(rows·cols²)` |
| Two 2D layers       | `O(rows·cols²·9)` | `O(cols²)`   |

---

## 8. Recognition Signals

| Signal in the problem statement                                         | Likely problem / state                          |
| ----------------------------------------------------------------------- | ----------------------------------------------- |
| "Move only right or down", "count paths"                                | Unique Paths — `dp[i][j]` count                 |
| "Move only right or down" + blocked cells                               | Unique Paths II — obstacle = 0                  |
| "Minimize/maximize sum along a top-to-bottom or left-to-right path"     | Minimum Path Sum — `dp[i][j]` = best cost       |
| Triangular grid, move to `j` or `j+1` below                             | Triangle — 1D bottom-up                         |
| "Falling path", choose below-left / below / below-right                 | Min Falling Path — min of three above           |
| "Can't repeat the previous choice on consecutive steps"                 | Ninja's Training — `dp[day][last]`              |
| Two agents / robots moving in lockstep down a grid                      | Cherry Pickup II — `dp[row][c1][c2]`            |
| Answer at a cell depends only on the previous row                       | → collapse to a **rolling row** (`O(n)` space)  |

---

## 9. Summary

- **Core idea.** Grid DP indexes state by coordinates. The recurrence aggregates a `min`/`max`/`sum` over a fixed, small set of neighbour cells (above, left, the three diagonals below, or — for two-agent problems — a pair of columns).
- **Direction of fill.** Path-count and min-path problems usually fill top-left → bottom-right; triangle, falling-path, ninja, and cherry-pickup are cleanest bottom-up. Pick whichever direction makes the base case a single boundary row.
- **Always look for the rolling row.** Whenever `dp[i]` depends only on `dp[i-1]` (and possibly already-computed cells of row `i`), you can drop a whole dimension: `O(m·n) → O(n)`, or for Cherry Pickup II, `O(rows·cols²) → O(cols²)` using two 2D layers. This is the highest-value optimization in this pattern and interviewers expect it.
- **C++ hygiene.** Declare tables as `vector<vector<int>> dp`; initialize memo tables with `-1` using nested vector constructors (or a 3D loop); use `min` / `max`; guard out-of-range neighbours with `INT_MAX` / `INT_MIN` sentinels rather than special-casing; prefer explicit comparison over subtraction for comparisons to avoid overflow.
- **Multi-agent state.** Cherry Pickup II generalizes the pattern: the two robots descend row-synchronously, which collapses what could be two time axes into a single shared `row` index, keeping the state at three dimensions.

See also: [1D DP (Fibonacci Style)](./1D%20DP%20(Fibonacci%20Style).md).

> **Last Updated:** 2026-06-26
