> **Topic:** [Dynamic Programming](../README.md) · **Pattern 2 of 8**

# 2D DP (Grids)

Grid DP is the natural next step after [1D DP (Fibonacci Style)](./1D%20DP%20(Fibonacci%20Style).md). Instead of a single dimension of state, the answer at a cell depends on a small set of *neighbouring* cells — typically the cell above, to the left, or diagonally adjacent. The recurrence is almost always a `min`/`max`/`sum` over those neighbours.

The defining trait of this pattern is that the **state is a coordinate** (a `(row, col)` pair, sometimes augmented with a second coordinate or a small categorical variable). Because every cell only looks at the previous row (and possibly the current row to its left), almost every grid problem collapses from `O(m·n)` memory to `O(n)` using a **rolling row** — replacing the full `dp[m][n]` table with one or two 1D arrays. This space optimization is the recurring theme below, and you should reach for it instinctively once the tabulation is correct.

This document covers problems **8–14** in the DP catalogue. Each problem is presented with its state definition, recurrence, base cases, complexity, and at least one dry-run for the representative cases. All code is full, compilable Rust.

---

## 1. Unique Paths (LC 62)

A robot starts at the top-left of an `m × n` grid and can only move **right** or **down**. Count the number of distinct paths to the bottom-right.

**State.** `dp[i][j]` = number of ways to reach cell `(i, j)` from `(0, 0)`.

**Recurrence.** `dp[i][j] = dp[i-1][j] + dp[i][j-1]` — you arrive at a cell either from above or from the left.

**Base cases.** The entire first row and first column have exactly one path (`dp[0][j] = dp[i][0] = 1`), since there is only one way to travel along an edge.

### Combinatorial formula

Any path consists of exactly `(m-1)` down-moves and `(n-1)` right-moves, in some order. The total number of moves is `(m-1)+(n-1) = m+n-2`, and we choose which of them are down-moves:

> **C(m + n − 2, m − 1)**

```rust
struct UniquePathsMath;

impl UniquePathsMath {
    // Choose (m-1) down moves out of (m+n-2) total moves.
    // Compute C(m+n-2, m-1) iteratively to avoid overflow / factorials.
    fn unique_paths(m: i32, n: i32) -> i32 {
        let total = m + n - 2;   // total moves
        let k = m - 1;           // pick the smaller for fewer iterations
        let mut result: i64 = 1;
        for i in 1..=k {
            // result *= (total - k + i) / i ; done in this order to stay integral
            result = result * (total - k + i) as i64 / i as i64;
        }
        result as i32
    }
}
```

### DP grid (`O(m·n)` time, `O(m·n)` space)

```rust
struct UniquePathsGrid;

impl UniquePathsGrid {
    fn unique_paths(m: usize, n: usize) -> i32 {
        let mut dp = vec![vec![0i32; n]; m];

        // Base cases: first column and first row each have exactly one path.
        for i in 0..m { dp[i][0] = 1; }
        for j in 0..n { dp[0][j] = 1; }

        for i in 1..m {
            for j in 1..n {
                dp[i][j] = dp[i - 1][j] + dp[i][j - 1];
            }
        }
        dp[m - 1][n - 1]
    }
}
```

### Space optimization: rolling row (`O(n)` space)

Row `i` only needs row `i-1`. Process left to right within a single array: at the moment we read `dp[j]` it still holds the value from the *previous* row (the cell above), and `dp[j-1]` already holds the *current* row (the cell to the left).

```rust
struct UniquePathsRolling;

impl UniquePathsRolling {
    fn unique_paths(m: usize, n: usize) -> i32 {
        let mut dp = vec![1i32; n];          // first row is all 1s

        for _i in 1..m {
            // dp[0] stays 1 (first column). Start from j = 1.
            for j in 1..n {
                // dp[j]   == value from row i-1 (cell above)
                // dp[j-1] == value already updated for row i (cell left)
                dp[j] = dp[j] + dp[j - 1];
            }
        }
        dp[n - 1]
    }
}
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

```rust
struct UniquePathsII;

impl UniquePathsII {
    fn unique_paths_with_obstacles(obstacle_grid: &Vec<Vec<i32>>) -> i32 {
        let m = obstacle_grid.len();
        let n = obstacle_grid[0].len();
        let mut dp = vec![0i32; n];

        // Start cell: 1 path if it is open, else 0 (and answer is 0).
        dp[0] = if obstacle_grid[0][0] == 1 { 0 } else { 1 };

        for i in 0..m {
            for j in 0..n {
                if obstacle_grid[i][j] == 1 {
                    dp[j] = 0;                       // blocked cell, no paths
                } else if j > 0 {
                    dp[j] = dp[j] + dp[j - 1];       // above + left
                }
                // j == 0 && open: dp[0] keeps its carried-down value
            }
        }
        dp[n - 1]
    }
}
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

```rust
struct MinPathSumRecursion;

impl MinPathSumRecursion {
    fn min_path_sum(grid: &Vec<Vec<i32>>) -> i32 {
        Self::solve(grid, (grid.len() - 1) as i32, (grid[0].len() - 1) as i32)
    }

    // f(i, j) = min cost to reach (i, j) from (0, 0)
    fn solve(grid: &Vec<Vec<i32>>, i: i32, j: i32) -> i32 {
        if i == 0 && j == 0 { return grid[0][0]; }
        if i < 0 || j < 0 { return i32::MAX; } // out of bounds = invalid

        let up = Self::solve(grid, i - 1, j);
        let left = Self::solve(grid, i, j - 1);
        // At most one of up/left is i32::MAX (both can't be invalid simultaneously)
        grid[i as usize][j as usize] + up.min(left)
    }
}
```

### Step 2 — Memoization (`Vec<Vec<i32>>` memo, initialized to `-1`)

```rust
struct MinPathSumMemo;

impl MinPathSumMemo {
    fn min_path_sum(grid: &Vec<Vec<i32>>) -> i32 {
        let m = grid.len();
        let n = grid[0].len();
        let mut memo = vec![vec![-1i32; n]; m];
        Self::solve(grid, (m - 1) as i32, (n - 1) as i32, &mut memo)
    }

    fn solve(grid: &Vec<Vec<i32>>, i: i32, j: i32, memo: &mut Vec<Vec<i32>>) -> i32 {
        if i == 0 && j == 0 { return grid[0][0]; }
        if i < 0 || j < 0 { return i32::MAX; }
        if memo[i as usize][j as usize] != -1 { return memo[i as usize][j as usize]; }

        let up = Self::solve(grid, i - 1, j, memo);
        let left = Self::solve(grid, i, j - 1, memo);
        let result = grid[i as usize][j as usize] + up.min(left);
        memo[i as usize][j as usize] = result;
        result
    }
}
```

### Step 3 — Tabulation (`O(m·n)` space)

```rust
struct MinPathSumTab;

impl MinPathSumTab {
    fn min_path_sum(grid: &Vec<Vec<i32>>) -> i32 {
        let m = grid.len();
        let n = grid[0].len();
        let mut dp = vec![vec![0i32; n]; m];

        for i in 0..m {
            for j in 0..n {
                if i == 0 && j == 0 {
                    dp[i][j] = grid[i][j];
                } else {
                    let up   = if i > 0 { dp[i - 1][j] } else { i32::MAX };
                    let left = if j > 0 { dp[i][j - 1] } else { i32::MAX };
                    dp[i][j] = grid[i][j] + up.min(left);
                }
            }
        }
        dp[m - 1][n - 1]
    }
}
```

### Step 4 — Rolling-row space optimization (`O(n)` space)

```rust
struct MinPathSumRolling;

impl MinPathSumRolling {
    fn min_path_sum(grid: &Vec<Vec<i32>>) -> i32 {
        let m = grid.len();
        let n = grid[0].len();
        let mut prev = vec![0i32; n];

        for i in 0..m {
            let mut curr = vec![0i32; n];
            for j in 0..n {
                if i == 0 && j == 0 {
                    curr[j] = grid[i][j];
                } else {
                    let up   = if i > 0 { prev[j] }     else { i32::MAX };
                    let left = if j > 0 { curr[j - 1] } else { i32::MAX };
                    curr[j] = grid[i][j] + up.min(left);
                }
            }
            prev = curr;
        }
        prev[n - 1]
    }
}
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

```rust
struct TriangleMinPath;

impl TriangleMinPath {
    fn minimum_total(triangle: &Vec<Vec<i32>>) -> i32 {
        let n = triangle.len();
        // Initialize dp with the last row.
        let mut dp = triangle[n - 1].clone();

        // Move upward, collapsing each row into dp.
        for i in (0..n - 1).rev() {
            let row = &triangle[i];
            for j in 0..=i {
                dp[j] = row[j] + dp[j].min(dp[j + 1]);
            }
        }
        dp[0]
    }
}
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

```rust
struct MinFallingPathSum;

impl MinFallingPathSum {
    fn min_falling_path_sum(matrix: &Vec<Vec<i32>>) -> i32 {
        let n = matrix.len();
        let mut prev: Vec<i32> = matrix[0].clone(); // base: first row

        for i in 1..n {
            let mut curr = vec![0i32; n];
            for j in 0..n {
                let up       = prev[j];
                let up_left  = if j > 0     { prev[j - 1] } else { i32::MAX };
                let up_right = if j < n - 1 { prev[j + 1] } else { i32::MAX };
                let best = up.min(up_left).min(up_right);
                curr[j] = matrix[i][j] + best;
            }
            prev = curr;
        }

        *prev.iter().min().unwrap()
    }
}
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

### Recursion + memoization (`Vec<Vec<i32>>` memo, initialized to `-1`)

```rust
struct NinjaTrainingMemo;

impl NinjaTrainingMemo {
    fn maximum_points(points: &Vec<Vec<i32>>, n: usize) -> i32 {
        // last in {0,1,2} = activity done previous day; 3 = none (day 0)
        let mut memo = vec![vec![-1i32; 4]; n];
        Self::solve(0, 3, points, n, &mut memo)
    }

    fn solve(day: usize, last: usize, points: &Vec<Vec<i32>>, n: usize, memo: &mut Vec<Vec<i32>>) -> i32 {
        if day == n { return 0; }
        if memo[day][last] != -1 { return memo[day][last]; }

        let mut best = 0;
        for task in 0..3usize {
            if task == last { continue; }
            let points_today = points[day][task] + Self::solve(day + 1, task, points, n, memo);
            best = best.max(points_today);
        }
        memo[day][last] = best;
        best
    }
}
```

### Tabulation (`O(n)` rows, `O(n·4)` space)

```rust
struct NinjaTrainingTab;

impl NinjaTrainingTab {
    fn maximum_points(points: &Vec<Vec<i32>>, n: usize) -> i32 {
        let mut dp = vec![vec![0i32; 4]; n + 1];
        // dp[n][*] = 0 (base case, already zero-initialized)

        for day in (0..n).rev() {
            for last in 0..4usize {
                let mut best = 0;
                for task in 0..3usize {
                    if task == last { continue; }
                    best = best.max(points[day][task] + dp[day + 1][task]);
                }
                dp[day][last] = best;
            }
        }
        dp[0][3]
    }
}
```

### Space optimization to two rows (`O(1)` extra, just 4 ints)

Since `dp[day]` depends only on `dp[day+1]`, we keep a single `next[4]` array (the "two rows" being `next` and the `curr` we build).

```rust
struct NinjaTrainingSpaceOpt;

impl NinjaTrainingSpaceOpt {
    fn maximum_points(points: &Vec<Vec<i32>>, n: usize) -> i32 {
        let mut next = vec![0i32; 4]; // dp for day+1, base = all zeros

        for day in (0..n).rev() {
            let mut curr = vec![0i32; 4];
            for last in 0..4usize {
                let mut best = 0;
                for task in 0..3usize {
                    if task == last { continue; }
                    best = best.max(points[day][task] + next[task]);
                }
                curr[last] = best;
            }
            next = curr;
        }
        next[3] // started with last = 3 (no restriction)
    }
}
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

### Memoization — 3D (`Vec<Vec<Vec<i32>>>` memo, init to a sentinel)

```rust
struct CherryPickupIIMemo;

impl CherryPickupIIMemo {
    fn cherry_pickup(grid: &Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let mut memo = vec![vec![vec![-1i32; cols]; cols]; rows];
        Self::solve(0, 0, cols - 1, grid, rows, cols, &mut memo)
    }

    fn solve(row: usize, c1: usize, c2: usize, grid: &Vec<Vec<i32>>,
             rows: usize, cols: usize, memo: &mut Vec<Vec<Vec<i32>>>) -> i32 {
        if memo[row][c1][c2] != -1 { return memo[row][c1][c2]; }

        // Cherries on the current row (count shared cell once).
        let collected = grid[row][c1] + if c1 != c2 { grid[row][c2] } else { 0 };

        if row == rows - 1 {
            memo[row][c1][c2] = collected;
            return collected;
        }

        // Explore all 9 (d1, d2) combinations for the next row.
        let mut best = i32::MIN;
        for d1 in -1i32..=1 {
            for d2 in -1i32..=1 {
                let n1 = c1 as i32 + d1;
                let n2 = c2 as i32 + d2;
                // Out of bounds is invalid -> skip.
                if n1 < 0 || n1 >= cols as i32 || n2 < 0 || n2 >= cols as i32 { continue; }
                let nxt = Self::solve(row + 1, n1 as usize, n2 as usize, grid, rows, cols, memo);
                best = best.max(nxt);
            }
        }
        memo[row][c1][c2] = collected + best;
        collected + best
    }
}
```

### Tabulation (full 3D table)

```rust
struct CherryPickupIITab;

impl CherryPickupIITab {
    fn cherry_pickup(grid: &Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let mut dp = vec![vec![vec![0i32; cols]; cols]; rows];

        // Base row: last row.
        for c1 in 0..cols {
            for c2 in 0..cols {
                dp[rows - 1][c1][c2] = Self::cherries(grid, rows - 1, c1, c2);
            }
        }

        for row in (0..rows - 1).rev() {
            for c1 in 0..cols {
                for c2 in 0..cols {
                    let mut best = i32::MIN;
                    for d1 in -1i32..=1 {
                        for d2 in -1i32..=1 {
                            let n1 = c1 as i32 + d1;
                            let n2 = c2 as i32 + d2;
                            if n1 < 0 || n1 >= cols as i32 || n2 < 0 || n2 >= cols as i32 { continue; }
                            best = best.max(dp[row + 1][n1 as usize][n2 as usize]);
                        }
                    }
                    dp[row][c1][c2] = Self::cherries(grid, row, c1, c2) + best;
                }
            }
        }
        dp[0][0][cols - 1]
    }

    fn cherries(grid: &Vec<Vec<i32>>, row: usize, c1: usize, c2: usize) -> i32 {
        if c1 == c2 { grid[row][c1] } else { grid[row][c1] + grid[row][c2] }
    }
}
```

### Space optimization to two 2D layers (`O(cols²)` space)

`dp[row]` depends only on `dp[row+1]`, so we keep one `next` 2D layer (`cols × cols`) and build the current layer from it — the "two layers" of the optimization.

```rust
struct CherryPickupIISpaceOpt;

impl CherryPickupIISpaceOpt {
    fn cherry_pickup(grid: &Vec<Vec<i32>>) -> i32 {
        let rows = grid.len();
        let cols = grid[0].len();
        let mut next = vec![vec![0i32; cols]; cols];

        // Base: last row.
        for c1 in 0..cols {
            for c2 in 0..cols {
                next[c1][c2] = Self::cherries(grid, rows - 1, c1, c2);
            }
        }

        for row in (0..rows - 1).rev() {
            let mut curr = vec![vec![0i32; cols]; cols];
            for c1 in 0..cols {
                for c2 in 0..cols {
                    let mut best = i32::MIN;
                    for d1 in -1i32..=1 {
                        for d2 in -1i32..=1 {
                            let n1 = c1 as i32 + d1;
                            let n2 = c2 as i32 + d2;
                            if n1 < 0 || n1 >= cols as i32 || n2 < 0 || n2 >= cols as i32 { continue; }
                            best = best.max(next[n1 as usize][n2 as usize]);
                        }
                    }
                    curr[c1][c2] = Self::cherries(grid, row, c1, c2) + best;
                }
            }
            next = curr;
        }
        next[0][cols - 1]
    }

    fn cherries(grid: &Vec<Vec<i32>>, row: usize, c1: usize, c2: usize) -> i32 {
        if c1 == c2 { grid[row][c1] } else { grid[row][c1] + grid[row][c2] }
    }
}
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

- **Core idea.** Grid DP indexes state by coordinates. The recurrence aggregates a `.min()`/`.max()`/`sum` over a fixed, small set of neighbour cells (above, left, the three diagonals below, or — for two-agent problems — a pair of columns).
- **Direction of fill.** Path-count and min-path problems usually fill top-left → bottom-right; triangle, falling-path, ninja, and cherry-pickup are cleanest bottom-up. Pick whichever direction makes the base case a single boundary row.
- **Always look for the rolling row.** Whenever `dp[i]` depends only on `dp[i-1]` (and possibly already-computed cells of row `i`), you can drop a whole dimension: `O(m·n) → O(n)`, or for Cherry Pickup II, `O(rows·cols²) → O(cols²)` using two 2D layers. This is the highest-value optimization in this pattern and interviewers expect it.
- **Rust hygiene.** Declare tables as `Vec<Vec<i32>>`; initialize memo tables with `-1` using nested `vec!` macros; use `.min()` / `.max()`; guard out-of-range neighbours with `i32::MAX` / `i32::MIN` sentinels rather than special-casing; prefer explicit comparison over subtraction for comparisons to avoid overflow.
- **Multi-agent state.** Cherry Pickup II generalizes the pattern: the two robots descend row-synchronously, which collapses what could be two time axes into a single shared `row` index, keeping the state at three dimensions.

See also: [1D DP (Fibonacci Style)](./1D%20DP%20(Fibonacci%20Style).md).

> **Last Updated:** 2026-06-26
