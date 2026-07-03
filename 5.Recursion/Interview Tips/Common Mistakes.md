# Common Mistakes — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Not Copying Path Before Adding to Results

```cpp
// BUG: all result entries point to the same mutating path (if using pointer/ref)
result.push_back(path);

// FIX: explicitly copy (C++ vectors copy by value, but be intentional)
result.push_back(vector<int>(path));
```

---

## 2. Missing Undo Step

```cpp
// BUG: path grows but never shrinks — leaks into sibling branches
path.push_back(nums[i]);
backtrack(nums, i + 1, path, res);
// forgot: path.pop_back();

// FIX: every push_back must have a matching pop_back
path.push_back(nums[i]);
backtrack(nums, i + 1, path, res);
path.pop_back();
```

---

## 3. Wrong Duplicate Skip Condition

```cpp
// BUG: skips first occurrence in sub-branches too
if (i > 0 && nums[i] == nums[i-1]) continue;

// FIX: only skip at same recursion depth
if (i > start && nums[i] == nums[i-1]) continue;
//      ^^^^^^ not i > 0
```

---

## 4. Sudoku — Wrong Box Index Formula

```cpp
// BUG: naive row/col doesn't map to 3×3 box
if (board[3*(r/3)+r][3*(c/3)+c] == ch) ... // incorrect

// FIX: iterate i from 0..8 with derived row/col
int boxR = 3 * (r / 3) + i / 3;
int boxC = 3 * (c / 3) + i % 3;
if (board[boxR][boxC] == ch) return false;
```

---

## 5. N-Queens — One Diagonal Set Instead of Two

```cpp
#include <bits/stdc++.h>
using namespace std;

// BUG: only checking one diagonal direction
unordered_set<int> diag;
diag.insert(row - col); // only main diagonal

// FIX: need both
unordered_set<int> diag1; // row - col (main diagonal)
unordered_set<int> diag2; // row + col (anti-diagonal)
```

---

## 6. Flood Fill — No Same-Color Guard

```cpp
#include <bits/stdc++.h>
using namespace std;

// BUG: if original == newColor, infinite recursion
vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
    dfs(image, sr, sc, image[sr][sc], color); // no guard

// FIX
if (image[sr][sc] == color) return image; // already this color, stop
```

---

## 7. Permutations II — `!used[i-1]` Reversed

```cpp
// BUG: condition reads "if previous was used" — wrong logic
if (i > 0 && nums[i] == nums[i-1] && used[i-1]) continue;

// FIX: skip if previous duplicate was NOT used (already tried and backtracked)
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
//                                     ^^ NOT used
```

---

## 8. Word Search — Not Restoring Board Cell

```cpp
// BUG: cell stays '#' even on backtrack — future DFS cannot visit it
board[r][c] = '#';
bool found = dfs(board, r+1, c, word, k+1) || ...;
// forgot: board[r][c] = tmp;

// FIX
char tmp = board[r][c];
board[r][c] = '#';
bool found = dfs(board, r+1, c, word, k+1) || ...;
board[r][c] = tmp; // always restore, even if found
```

---

## 9. INT_MIN Overflow in Pow

```cpp
// BUG: -INT_MIN overflows to INT_MIN
double myPow(double x, int n) {
    if (n < 0) { x = 1.0/x; n = -n; } // n stays INT_MIN

// FIX: use long
double myPow(double x, int n) {
    return power(x, (long) n);
}
double power(double x, long n) {
    if (n < 0) { x = 1.0/x; n = -n; } // safe with long
```

---

## 10. Combination Sum — Passing `i+1` Instead of `i`

```cpp
// BUG: elements cannot be reused because we skip past current index
backtrack(cands, i + 1, remaining - cands[i], path, res);

// FIX for Combination Sum (reuse allowed)
backtrack(cands, i, remaining - cands[i], path, res);
// Use i+1 only for Combination Sum II (no reuse)
```

---

## Quick Checklist Before Submitting

- [ ] Path copied by value when adding to results (use `result.push_back(path)`)?
- [ ] Undo step mirrors make step exactly (use `pop_back()` after `push_back()`)?
- [ ] `i > start` (not `i > 0`) for duplicate skip?
- [ ] Two diagonal sets in N-Queens?
- [ ] `!used[i-1]` (not `used[i-1]`) in Permutations II?
- [ ] Board cell restored after Word Search DFS?
- [ ] `(long) n` cast in Pow(x, n)?
- [ ] `i` (not `i+1`) in Combination Sum reuse case?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Recursion Tree Visualization](./Recursion%20Tree%20Visualization.md)

> **Last Updated:** 2026-06-26
