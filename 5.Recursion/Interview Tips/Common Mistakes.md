# Common Mistakes — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Not Copying Path Before Adding to Results

```java
// BUG: all result entries are references to the same mutating list
result.add(path);

// FIX
result.add(new ArrayList<>(path));
```

---

## 2. Missing Undo Step

```java
// BUG: path grows but never shrinks — leaks into sibling branches
path.add(nums[i]);
backtrack(nums, i + 1, path, res);
// forgot: path.remove(path.size() - 1);

// FIX: every add must have a matching remove
path.add(nums[i]);
backtrack(nums, i + 1, path, res);
path.remove(path.size() - 1);
```

---

## 3. Wrong Duplicate Skip Condition

```java
// BUG: skips first occurrence in sub-branches too
if (i > 0 && nums[i] == nums[i-1]) continue;

// FIX: only skip at same recursion depth
if (i > start && nums[i] == nums[i-1]) continue;
//      ^^^^^^ not i > 0
```

---

## 4. Sudoku — Wrong Box Index Formula

```java
// BUG: naive row/col doesn't map to 3×3 box
if (board[3*(r/3)+r][3*(c/3)+c] == ch) ... // incorrect

// FIX: iterate i from 0..8 with derived row/col
int boxR = 3 * (r / 3) + i / 3;
int boxC = 3 * (c / 3) + i % 3;
if (board[boxR][boxC] == ch) return false;
```

---

## 5. N-Queens — One Diagonal Set Instead of Two

```java
// BUG: only checking one diagonal direction
Set<Integer> diag = new HashSet<>();
diag.add(row - col); // only main diagonal

// FIX: need both
Set<Integer> diag1 = new HashSet<>(); // row - col (main diagonal)
Set<Integer> diag2 = new HashSet<>(); // row + col (anti-diagonal)
```

---

## 6. Flood Fill — No Same-Color Guard

```java
// BUG: if original == newColor, infinite recursion
public int[][] floodFill(int[][] image, int sr, int sc, int color) {
    dfs(image, sr, sc, image[sr][sc], color); // no guard

// FIX
if (image[sr][sc] == color) return image; // already this color, stop
```

---

## 7. Permutations II — `!used[i-1]` Reversed

```java
// BUG: condition reads "if previous was used" — wrong logic
if (i > 0 && nums[i] == nums[i-1] && used[i-1]) continue;

// FIX: skip if previous duplicate was NOT used (already tried and backtracked)
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
//                                     ^^ NOT used
```

---

## 8. Word Search — Not Restoring Board Cell

```java
// BUG: cell stays '#' even on backtrack — future DFS cannot visit it
board[r][c] = '#';
boolean found = dfs(board, r+1, c, word, k+1) || ...;
// forgot: board[r][c] = tmp;

// FIX
char tmp = board[r][c];
board[r][c] = '#';
boolean found = dfs(board, r+1, c, word, k+1) || ...;
board[r][c] = tmp; // always restore, even if found
```

---

## 9. Integer.MIN_VALUE Overflow in Pow

```java
// BUG: -Integer.MIN_VALUE overflows to Integer.MIN_VALUE
public double myPow(double x, int n) {
    if (n < 0) { x = 1.0/x; n = -n; } // n stays MIN_VALUE

// FIX: use long
public double myPow(double x, int n) {
    return power(x, (long) n);
}
private double power(double x, long n) {
    if (n < 0) { x = 1.0/x; n = -n; } // safe with long
```

---

## 10. Combination Sum — Passing `i+1` Instead of `i`

```java
// BUG: elements cannot be reused because we skip past current index
backtrack(cands, i + 1, remaining - cands[i], path, res);

// FIX for Combination Sum (reuse allowed)
backtrack(cands, i, remaining - cands[i], path, res);
// Use i+1 only for Combination Sum II (no reuse)
```

---

## Quick Checklist Before Submitting

- [ ] `new ArrayList<>(path)` when adding to results?
- [ ] Undo step mirrors make step exactly?
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
