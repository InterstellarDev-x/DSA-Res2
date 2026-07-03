# Backtracking

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** N-Queens, Sudoku, Word Search, Generate Parentheses, Palindrome Partitioning

---

## Core Idea

**Try → Recurse → Undo.** At each decision point, make a choice, recurse to explore that branch, then undo the choice to restore state before trying the next option.

```cpp
#include <bits/stdc++.h>
using namespace std;

void backtrack(State& state, vector<Choice>& choices) {
    if (goalReached(state)) {
        results.push_back(copy of state);
        return;
    }
    for (auto& choice : choices) {
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

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> generateParenthesis(int n) {
    vector<string> result;
    string sb;
    backtrack(result, sb, 0, 0, n);
    return result;
}

void backtrack(vector<string>& res, string& sb, int open, int close, int n) {
    if (sb.length() == 2 * n) { res.push_back(sb); return; }

    if (open < n) {
        sb += '(';
        backtrack(res, sb, open + 1, close, n);
        sb.pop_back(); // undo
    }
    if (close < open) {
        sb += ')';
        backtrack(res, sb, open, close + 1, n);
        sb.pop_back(); // undo
    }
}
```

**Note:** Use `string` with `+=` and `pop_back()` — `string` concatenation with `+` creates a new object at every call (O(n) per concat vs O(1) `+=`/`pop_back()`).

---

## Template 2 — N-Queens (LC 51)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<string>> solveNQueens(int n) {
    vector<vector<string>> result;
    vector<int> queens(n, -1); // queens[row] = column
    unordered_set<int> cols, diag1, diag2;
    backtrack(result, queens, n, 0, cols, diag1, diag2);
    return result;
}

void backtrack(vector<vector<string>>& res, vector<int>& queens, int n, int row,
               unordered_set<int>& cols, unordered_set<int>& diag1, unordered_set<int>& diag2) {
    if (row == n) {
        res.push_back(buildBoard(queens, n));
        return;
    }
    for (int col = 0; col < n; col++) {
        if (cols.count(col) || diag1.count(row - col) || diag2.count(row + col))
            continue;
        queens[row] = col;
        cols.insert(col); diag1.insert(row - col); diag2.insert(row + col);
        backtrack(res, queens, n, row + 1, cols, diag1, diag2);
        queens[row] = -1;
        cols.erase(col); diag1.erase(row - col); diag2.erase(row + col);
    }
}

vector<string> buildBoard(vector<int>& queens, int n) {
    vector<string> board;
    for (int row = 0; row < n; row++) {
        string rowStr(n, '.');
        rowStr[queens[row]] = 'Q';
        board.push_back(rowStr);
    }
    return board;
}
```

**Diagonal invariants:**
- Main diagonal (top-left to bottom-right): `row - col` is constant
- Anti-diagonal (top-right to bottom-left): `row + col` is constant

**Bit manipulation version for N-Queens II (count only):**

```cpp
#include <bits/stdc++.h>
using namespace std;

int totalNQueens(int n) {
    return solve(n, 0, 0, 0, 0);
}
int solve(int n, int row, int cols, int diag1, int diag2) {
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

```cpp
#include <bits/stdc++.h>
using namespace std;

void solveSudoku(vector<vector<char>>& board) {
    solve(board);
}

bool solve(vector<vector<char>>& board) {
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

bool isValid(vector<vector<char>>& board, int r, int c, char ch) {
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

```cpp
#include <bits/stdc++.h>
using namespace std;

bool exist(vector<vector<char>>& board, string word) {
    int m = board.size(), n = board[0].size();
    for (int r = 0; r < m; r++)
        for (int c = 0; c < n; c++)
            if (dfs(board, r, c, word, 0)) return true;
    return false;
}

bool dfs(vector<vector<char>>& board, int r, int c, string& word, int k) {
    if (k == (int)word.length()) return true;
    if (r < 0 || r >= (int)board.size() || c < 0 || c >= (int)board[0].size()) return false;
    if (board[r][c] != word[k]) return false;

    char tmp = board[r][c];
    board[r][c] = '#';  // mark visited in-place (no extra array)
    bool found = dfs(board, r+1, c, word, k+1) ||
                 dfs(board, r-1, c, word, k+1) ||
                 dfs(board, r, c+1, word, k+1) ||
                 dfs(board, r, c-1, word, k+1);
    board[r][c] = tmp;  // restore
    return found;
}
```

---

## Template 5 — Palindrome Partitioning (LC 131)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<string>> partition(string s) {
    vector<vector<string>> result;
    vector<string> path;
    backtrack(s, 0, path, result);
    return result;
}

void backtrack(string& s, int start, vector<string>& path, vector<vector<string>>& res) {
    if (start == (int)s.length()) { res.push_back(path); return; }
    for (int end = start + 1; end <= (int)s.length(); end++) {
        if (isPalin(s, start, end - 1)) {
            path.push_back(s.substr(start, end - start));
            backtrack(s, end, path, res);
            path.pop_back(); // undo
        }
    }
}

bool isPalin(string& s, int lo, int hi) {
    while (lo < hi) if (s[lo++] != s[hi--]) return false;
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
| Forgetting to copy `path` when adding to results | `res.push_back(path)` copies by value in C++ |
| String concat in inner loop | Use `string` with `+=` and `pop_back()` |
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
