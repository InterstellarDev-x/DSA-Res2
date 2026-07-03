# Backtracking

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** N-Queens, Sudoku, Word Search, Generate Parentheses, Palindrome Partitioning

---

## Core Idea

**Try → Recurse → Undo.** At each decision point, make a choice, recurse to explore that branch, then undo the choice to restore state before trying the next option.

```rust
fn backtrack(state: &mut State, choices: &[Choice]) {
    if goal_reached(state) {
        results.push(state.clone());
        return;
    }
    for choice in choices {
        if is_valid(choice, state) {
            make_choice(choice, state);    // Try
            backtrack(next_state, ...);    // Recurse
            undo_choice(choice, state);    // Undo
        }
    }
}
```

**Key discipline:** The undo step must exactly mirror the make step.

---

## Template 1 — Generate Parentheses (LC 22)

Constraint: `open ≤ n`, `close ≤ open`.

```rust
fn generate_parenthesis(n: i32) -> Vec<String> {
    let mut result = Vec::new();
    let mut sb = String::new();
    backtrack(&mut result, &mut sb, 0, 0, n);
    result
}

fn backtrack(res: &mut Vec<String>, sb: &mut String, open: i32, close: i32, n: i32) {
    if sb.len() == (2 * n) as usize { res.push(sb.clone()); return; }

    if open < n {
        sb.push('(');
        backtrack(res, sb, open + 1, close, n);
        sb.pop(); // undo
    }
    if close < open {
        sb.push(')');
        backtrack(res, sb, open, close + 1, n);
        sb.pop(); // undo
    }
}
```

**Note:** Use `String` with `push(char)` and `pop()` — `String` concatenation with `+` or `format!` creates a new allocation at every call (O(n) per concat vs O(1) `push`/`pop`).

---

## Template 2 — N-Queens (LC 51)

```rust
use std::collections::HashSet;

fn solve_n_queens(n: i32) -> Vec<Vec<String>> {
    let mut result = Vec::new();
    let mut queens = vec![-1i32; n as usize]; // queens[row] = column
    let mut cols: HashSet<i32> = HashSet::new();
    let mut diag1: HashSet<i32> = HashSet::new();
    let mut diag2: HashSet<i32> = HashSet::new();
    backtrack(&mut result, &mut queens, n, 0, &mut cols, &mut diag1, &mut diag2);
    result
}

fn backtrack(
    res: &mut Vec<Vec<String>>,
    queens: &mut Vec<i32>,
    n: i32,
    row: i32,
    cols: &mut HashSet<i32>,
    diag1: &mut HashSet<i32>,
    diag2: &mut HashSet<i32>,
) {
    if row == n {
        res.push(build_board(queens, n));
        return;
    }
    for col in 0..n {
        if cols.contains(&col) || diag1.contains(&(row - col)) || diag2.contains(&(row + col)) {
            continue;
        }
        queens[row as usize] = col;
        cols.insert(col); diag1.insert(row - col); diag2.insert(row + col);
        backtrack(res, queens, n, row + 1, cols, diag1, diag2);
        queens[row as usize] = -1;
        cols.remove(&col); diag1.remove(&(row - col)); diag2.remove(&(row + col));
    }
}

fn build_board(queens: &[i32], n: i32) -> Vec<String> {
    let mut board = Vec::new();
    for row in 0..n as usize {
        let mut row_chars: Vec<char> = vec!['.'; n as usize];
        row_chars[queens[row] as usize] = 'Q';
        board.push(row_chars.iter().collect());
    }
    board
}
```

**Diagonal invariants:**
- Main diagonal (top-left to bottom-right): `row - col` is constant
- Anti-diagonal (top-right to bottom-left): `row + col` is constant

**Bit manipulation version for N-Queens II (count only):**

```rust
fn total_n_queens(n: i32) -> i32 {
    solve(n, 0, 0, 0, 0)
}

fn solve(n: i32, row: i32, cols: i32, diag1: i32, diag2: i32) -> i32 {
    if row == n { return 1; }
    let mut count = 0;
    let mut available_mask = ((1 << n) - 1) & !(cols | diag1 | diag2);
    while available_mask != 0 {
        let bit = available_mask & (-available_mask); // lowest set bit
        available_mask &= available_mask - 1;          // clear lowest bit
        count += solve(n, row + 1, cols | bit, (diag1 | bit) << 1, (diag2 | bit) >> 1);
    }
    count
}
```

---

## Template 3 — Sudoku Solver (LC 37)

```rust
fn solve_sudoku(board: &mut Vec<Vec<char>>) {
    solve(board);
}

fn solve(board: &mut Vec<Vec<char>>) -> bool {
    for r in 0..9 {
        for c in 0..9 {
            if board[r][c] != '.' { continue; }
            for digit in b'1'..=b'9' {
                let ch = digit as char;
                if is_valid(board, r, c, ch) {
                    board[r][c] = ch;
                    if solve(board) { return true; } // found solution
                    board[r][c] = '.';               // undo
                }
            }
            return false; // no valid digit → backtrack
        }
    }
    true // all cells filled
}

fn is_valid(board: &[Vec<char>], r: usize, c: usize, ch: char) -> bool {
    for i in 0..9 {
        if board[r][i] == ch { return false; }             // same row
        if board[i][c] == ch { return false; }             // same col
        let box_r = 3 * (r / 3) + i / 3;
        let box_c = 3 * (c / 3) + i % 3;
        if board[box_r][box_c] == ch { return false; }     // same 3x3 box
    }
    true
}
```

---

## Template 4 — Word Search (LC 79)

```rust
fn exist(board: &mut Vec<Vec<char>>, word: &str) -> bool {
    let m = board.len();
    let n = board[0].len();
    let word: Vec<char> = word.chars().collect();
    for r in 0..m {
        for c in 0..n {
            if dfs(board, r as i32, c as i32, &word, 0) { return true; }
        }
    }
    false
}

fn dfs(board: &mut Vec<Vec<char>>, r: i32, c: i32, word: &[char], k: usize) -> bool {
    if k == word.len() { return true; }
    if r < 0 || r >= board.len() as i32 || c < 0 || c >= board[0].len() as i32 { return false; }
    let (ru, cu) = (r as usize, c as usize);
    if board[ru][cu] != word[k] { return false; }

    let tmp = board[ru][cu];
    board[ru][cu] = '#';  // mark visited in-place (no extra array)
    let found = dfs(board, r + 1, c, word, k + 1) ||
                dfs(board, r - 1, c, word, k + 1) ||
                dfs(board, r, c + 1, word, k + 1) ||
                dfs(board, r, c - 1, word, k + 1);
    board[ru][cu] = tmp;  // restore
    found
}
```

---

## Template 5 — Palindrome Partitioning (LC 131)

```rust
fn partition(s: &str) -> Vec<Vec<String>> {
    let mut result = Vec::new();
    let mut path: Vec<String> = Vec::new();
    let chars: Vec<char> = s.chars().collect();
    backtrack(&chars, 0, &mut path, &mut result);
    result
}

fn backtrack(s: &[char], start: usize, path: &mut Vec<String>, res: &mut Vec<Vec<String>>) {
    if start == s.len() { res.push(path.clone()); return; }
    for end in (start + 1)..=s.len() {
        if is_palin(s, start, end - 1) {
            path.push(s[start..end].iter().collect());
            backtrack(s, end, path, res);
            path.pop(); // undo
        }
    }
}

fn is_palin(s: &[char], mut lo: usize, mut hi: usize) -> bool {
    while lo < hi {
        if s[lo] != s[hi] { return false; }
        lo += 1;
        hi -= 1;
    }
    true
}
```

**Optimization:** Precompute a 2D palindrome table `dp[i][j]` so `is_palin` is O(1).

---

## Pruning Strategies

| Problem | Pruning Condition |
|---------|-----------------|
| N-Queens | Skip if col/diag already attacked |
| Sudoku | Skip if digit already in row/col/box |
| Word Search | Return false on out-of-bounds or mismatch early |
| Combination Sum | Break loop when `candidates[i] > remaining` (after sort) |
| Subsets II | Skip `i > start && nums[i] == nums[i-1]` to avoid duplicates |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not undoing — state leaks into sibling branches | Mirror every make step with an undo step |
| Forgetting to clone `path` when adding to results | `res.push(path.clone())` clones the Vec in Rust |
| String concat in inner loop | Use `String` with `push(char)` and `pop()` |
| Marking visited but not restoring — loses cells | Restore `board[r][c] = tmp` after DFS |
| N-Queens diag: using only one diagonal set | Need two sets: `row - col` AND `row + col` |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Generate Parentheses | O(4^n / √n) — Catalan number | O(n) stack |
| N-Queens | O(n!) pruned | O(n) queens array + sets |
| N-Queens II (bitmask) | O(n!) pruned | O(n) |
| Sudoku | O(9^(empty cells)) | O(1) extra |
| Word Search | O(m×n × 4^L) | O(L) stack |
| Palindrome Partition | O(n × 2^n) | O(n) stack |

---

## Related Patterns

- [Subsets & Combinations](./Subsets%20and%20Combinations.md) — structured subset of backtracking
- [Permutations](./Permutations.md) — backtracking with swap/used[]
- [Basic Recursion](./Basic%20Recursion.md) — recursion without undo step

---

**Back:** [Recursion README](../README.md) | **Prev:** [Basic Recursion](./Basic%20Recursion.md) | **Next:** [Subsets & Combinations](./Subsets%20and%20Combinations.md)

> **Last Updated:** 2026-06-26
