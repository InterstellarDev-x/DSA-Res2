# Google Interview Problems — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | N-Queens | L4–L6 | Hard | ⭐⭐⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [51](https://leetcode.com/problems/n-queens/) |
| 2 | Sudoku Solver | L4 | Hard | ⭐⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [37](https://leetcode.com/problems/sudoku-solver/) |
| 3 | Expression Add Operators | L5 | Hard | ⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [282](https://leetcode.com/problems/expression-add-operators/) |
| 4 | Palindrome Partitioning | L4 | Medium | ⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [131](https://leetcode.com/problems/palindrome-partitioning/) |
| 5 | Regular Expression Matching | L5 | Hard | ⭐⭐⭐ | [Divide & Conquer](../Patterns/Divide%20and%20Conquer.md) | [10](https://leetcode.com/problems/regular-expression-matching/) |

---

## Deep Dive: N-Queens

### Google Follow-ups

**Q1: Can you solve it with bitmasks? (N-Queens II for count)**

```java
public int totalNQueens(int n) {
    return solve(n, 0, 0, 0, 0);
}

// cols, diag1, diag2 are bitmasks of attacked positions
private int solve(int n, int row, int cols, int diag1, int diag2) {
    if (row == n) return 1;
    int count = 0;
    // All positions NOT attacked by cols, diag1, or diag2 in this row
    int available = ((1 << n) - 1) & ~(cols | diag1 | diag2);
    while (available != 0) {
        int bit = available & (-available);  // isolate lowest set bit
        available &= available - 1;           // clear lowest set bit
        // Shift diagonals: left-diagonal shifts left, right-diagonal shifts right
        count += solve(n, row + 1, cols | bit, (diag1 | bit) << 1, (diag2 | bit) >> 1);
    }
    return count;
}
```

Time: O(n!) pruned, much faster in practice.

**Q2: How would you reconstruct paths from the bitmask version?**
Track which `bit` was chosen per row in an `int[] colsChosen` array, then build board from it.

**Q3: What is the runtime of the HashSet approach?**
Each set contains/add is O(1) average. Overall O(n!) with pruning. The bitmask approach has lower constant factor due to no HashMap overhead.

---

## Deep Dive: Sudoku Solver — Optimization

**Constraint Propagation before backtracking** (interviewers at Google L5+ may ask this):

Before trying all 9 digits, compute the set of valid digits for each empty cell by intersecting row/col/box constraints. This dramatically reduces branching.

```java
// Precompute valid digits for (r, c):
Set<Integer> valid = new HashSet<>(Arrays.asList(1,2,3,4,5,6,7,8,9));
for (int i = 0; i < 9; i++) {
    if (board[r][i] != '.') valid.remove(board[r][i] - '0');
    if (board[i][c] != '.') valid.remove(board[i][c] - '0');
    int br = 3*(r/3)+i/3, bc = 3*(c/3)+i%3;
    if (board[br][bc] != '.') valid.remove(board[br][bc] - '0');
}
```

Then pick the cell with **fewest valid options** (MRV heuristic) to reduce tree width.

---

## Google Interview Framework for Backtracking

1. **State:** What is the current partial solution? (e.g., queens placed so far)
2. **Choice:** What are the valid next decisions? (e.g., columns for next row)
3. **Constraint:** What makes a choice invalid? (attacks for N-Queens)
4. **Goal:** When is the solution complete? (all rows filled)
5. **Undo:** Exactly what was changed when we made the choice? (undo it)

---

## Related Files

- [Google OA-Qns](../OA-Qns/Google.md)
- [Recursion README](../README.md)

> **Last Updated:** 2026-06-26
