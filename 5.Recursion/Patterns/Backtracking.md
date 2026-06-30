# Backtracking

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** N-Queens, Sudoku, Word Search, Generate Parentheses, Palindrome Partitioning

---

## Core Idea

**Try → Recurse → Undo.** At each decision point, make a choice, recurse to explore that branch, then undo the choice to restore state before trying the next option.

```java
void backtrack(state, choices) {
    if (goalReached(state)) {
        results.add(copy of state);
        return;
    }
    for (choice : choices) {
        if (isValid(choice, state)) {
            makeChoice(choice, state);    // Try
            backtrack(nextState, ...);    // Recurse
            undoChoice(choice, state);    // Undo
        }
    }
}
```

**Key discipline:** The undo step must exactly mirror the make step.

---

## Template 1 — Generate Parentheses (LC 22)

Constraint: `open ≤ n`, `close ≤ open`.

```java
public List<String> generateParenthesis(int n) {
    List<String> result = new ArrayList<>();
    backtrack(result, new StringBuilder(), 0, 0, n);
    return result;
}

private void backtrack(List<String> res, StringBuilder sb, int open, int close, int n) {
    if (sb.length() == 2 * n) { res.add(sb.toString()); return; }

    if (open < n) {
        sb.append('(');
        backtrack(res, sb, open + 1, close, n);
        sb.deleteCharAt(sb.length() - 1); // undo
    }
    if (close < open) {
        sb.append(')');
        backtrack(res, sb, open, close + 1, n);
        sb.deleteCharAt(sb.length() - 1); // undo
    }
}
```

**Note:** Use `StringBuilder` — `String` concatenation creates a new object at every call (O(n) per concat vs O(1) append/deleteCharAt).

---

## Template 2 — N-Queens (LC 51)

```java
public List<List<String>> solveNQueens(int n) {
    List<List<String>> result = new ArrayList<>();
    int[] queens = new int[n]; // queens[row] = column
    Arrays.fill(queens, -1);
    Set<Integer> cols = new HashSet<>(), diag1 = new HashSet<>(), diag2 = new HashSet<>();
    backtrack(result, queens, n, 0, cols, diag1, diag2);
    return result;
}

private void backtrack(List<List<String>> res, int[] queens, int n, int row,
                        Set<Integer> cols, Set<Integer> diag1, Set<Integer> diag2) {
    if (row == n) {
        res.add(buildBoard(queens, n));
        return;
    }
    for (int col = 0; col < n; col++) {
        if (cols.contains(col) || diag1.contains(row - col) || diag2.contains(row + col))
            continue;
        queens[row] = col;
        cols.add(col); diag1.add(row - col); diag2.add(row + col);
        backtrack(res, queens, n, row + 1, cols, diag1, diag2);
        queens[row] = -1;
        cols.remove(col); diag1.remove(row - col); diag2.remove(row + col);
    }
}

private List<String> buildBoard(int[] queens, int n) {
    List<String> board = new ArrayList<>();
    for (int row = 0; row < n; row++) {
        char[] rowArr = new char[n];
        Arrays.fill(rowArr, '.');
        rowArr[queens[row]] = 'Q';
        board.add(new String(rowArr));
    }
    return board;
}
```

**Diagonal invariants:**
- Main diagonal (top-left to bottom-right): `row - col` is constant
- Anti-diagonal (top-right to bottom-left): `row + col` is constant

**Bit manipulation version for N-Queens II (count only):**

```java
public int totalNQueens(int n) {
    return solve(n, 0, 0, 0, 0);
}
private int solve(int n, int row, int cols, int diag1, int diag2) {
    if (row == n) return 1;
    int count = 0;
    int availableMask = ((1 << n) - 1) & ~(cols | diag1 | diag2);
    while (availableMask != 0) {
        int bit = availableMask & (-availableMask); // lowest set bit
        availableMask &= availableMask - 1;          // clear lowest bit
        count += solve(n, row + 1, cols | bit, (diag1 | bit) << 1, (diag2 | bit) >> 1);
    }
    return count;
}
```

---

## Template 3 — Sudoku Solver (LC 37)

```java
public void solveSudoku(char[][] board) {
    solve(board);
}

private boolean solve(char[][] board) {
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (board[r][c] != '.') continue;
            for (char ch = '1'; ch <= '9'; ch++) {
                if (isValid(board, r, c, ch)) {
                    board[r][c] = ch;
                    if (solve(board)) return true; // found solution
                    board[r][c] = '.';             // undo
                }
            }
            return false; // no valid digit → backtrack
        }
    }
    return true; // all cells filled
}

private boolean isValid(char[][] board, int r, int c, char ch) {
    for (int i = 0; i < 9; i++) {
        if (board[r][i] == ch) return false;             // same row
        if (board[i][c] == ch) return false;             // same col
        int boxR = 3 * (r / 3) + i / 3;
        int boxC = 3 * (c / 3) + i % 3;
        if (board[boxR][boxC] == ch) return false;       // same 3x3 box
    }
    return true;
}
```

---

## Template 4 — Word Search (LC 79)

```java
public boolean exist(char[][] board, String word) {
    int m = board.length, n = board[0].length;
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            if (dfs(board, r, c, word, 0)) return true;
    return false;
}

private boolean dfs(char[][] board, int r, int c, String word, int k) {
    if (k == word.length()) return true;
    if (r < 0 || r >= board.length || c < 0 || c >= board[0].length) return false;
    if (board[r][c] != word.charAt(k)) return false;

    char tmp = board[r][c];
    board[r][c] = '#';  // mark visited in-place (no extra array)
    boolean found = dfs(board, r+1, c, word, k+1) ||
                    dfs(board, r-1, c, word, k+1) ||
                    dfs(board, r, c+1, word, k+1) ||
                    dfs(board, r, c-1, word, k+1);
    board[r][c] = tmp;  // restore
    return found;
}
```

---

## Template 5 — Palindrome Partitioning (LC 131)

```java
public List<List<String>> partition(String s) {
    List<List<String>> result = new ArrayList<>();
    backtrack(s, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(String s, int start, List<String> path, List<List<String>> res) {
    if (start == s.length()) { res.add(new ArrayList<>(path)); return; }
    for (int end = start + 1; end <= s.length(); end++) {
        if (isPalin(s, start, end - 1)) {
            path.add(s.substring(start, end));
            backtrack(s, end, path, res);
            path.remove(path.size() - 1); // undo
        }
    }
}

private boolean isPalin(String s, int lo, int hi) {
    while (lo < hi) if (s.charAt(lo++) != s.charAt(hi--)) return false;
    return true;
}
```

**Optimization:** Precompute a 2D palindrome table `dp[i][j]` so `isPalin` is O(1).

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
| Storing reference to `path` instead of copy | `new ArrayList<>(path)` when adding to results |
| String concat in inner loop | Use `StringBuilder` with `append`/`deleteCharAt` |
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
