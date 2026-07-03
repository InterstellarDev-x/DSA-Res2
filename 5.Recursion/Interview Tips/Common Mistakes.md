# Common Mistakes — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Not Copying Path Before Adding to Results

```rust
// BUG: all result entries point to the same mutating path (if using pointer/ref)
result.push(path.clone());

// FIX: explicitly clone (Rust requires explicit cloning)
result.push(path.clone());
```

---

## 2. Missing Undo Step

```rust
// BUG: path grows but never shrinks — leaks into sibling branches
path.push(nums[i]);
backtrack(&nums, i + 1, &mut path, &mut res);
// forgot: path.pop();

// FIX: every push must have a matching pop
path.push(nums[i]);
backtrack(&nums, i + 1, &mut path, &mut res);
path.pop();
```

---

## 3. Wrong Duplicate Skip Condition

```rust
// BUG: skips first occurrence in sub-branches too
if i > 0 && nums[i] == nums[i - 1] { continue; }

// FIX: only skip at same recursion depth
if i > start && nums[i] == nums[i - 1] { continue; }
//      ^^^^^^ not i > 0
```

---

## 4. Sudoku — Wrong Box Index Formula

```rust
// BUG: naive row/col doesn't map to 3×3 box
if board[3 * (r / 3) + r][3 * (c / 3) + c] == ch { ... } // incorrect

// FIX: iterate i from 0..9 with derived row/col
let box_r = 3 * (r / 3) + i / 3;
let box_c = 3 * (c / 3) + i % 3;
if board[box_r][box_c] == ch { return false; }
```

---

## 5. N-Queens — One Diagonal Set Instead of Two

```rust
use std::collections::HashSet;

// BUG: only checking one diagonal direction
let mut diag: HashSet<i32> = HashSet::new();
diag.insert(row as i32 - col as i32); // only main diagonal

// FIX: need both
let mut diag1: HashSet<i32> = HashSet::new(); // row - col (main diagonal)
let mut diag2: HashSet<i32> = HashSet::new(); // row + col (anti-diagonal)
```

---

## 6. Flood Fill — No Same-Color Guard

```rust
// BUG: if original == newColor, infinite recursion
fn flood_fill(image: &mut Vec<Vec<i32>>, sr: usize, sc: usize, color: i32) -> Vec<Vec<i32>> {
    dfs(image, sr, sc, image[sr][sc], color); // no guard
    image.clone()
}

// FIX
if image[sr][sc] == color { return image.clone(); } // already this color, stop
```

---

## 7. Permutations II — `!used[i-1]` Reversed

```rust
// BUG: condition reads "if previous was used" — wrong logic
if i > 0 && nums[i] == nums[i - 1] && used[i - 1] { continue; }

// FIX: skip if previous duplicate was NOT used (already tried and backtracked)
if i > 0 && nums[i] == nums[i - 1] && !used[i - 1] { continue; }
//                                      ^^ NOT used
```

---

## 8. Word Search — Not Restoring Board Cell

```rust
// BUG: cell stays '#' even on backtrack — future DFS cannot visit it
board[r][c] = '#';
let found = dfs(board, r + 1, c, word, k + 1) || ...;
// forgot: board[r][c] = tmp;

// FIX
let tmp = board[r][c];
board[r][c] = '#';
let found = dfs(board, r + 1, c, word, k + 1) || ...;
board[r][c] = tmp; // always restore, even if found
```

---

## 9. i32::MIN Overflow in Pow

```rust
// BUG: -i32::MIN overflows back to i32::MIN
fn my_pow(x: f64, n: i32) -> f64 {
    // if n < 0 { x = 1.0/x; n = -n; }  // n stays i32::MIN — overflow!
    todo!()
}

// FIX: use i64
fn my_pow(x: f64, n: i32) -> f64 {
    power(x, n as i64)
}
fn power(mut x: f64, mut n: i64) -> f64 {
    if n < 0 { x = 1.0 / x; n = -n; } // safe with i64
    todo!()
}
```

---

## 10. Combination Sum — Passing `i+1` Instead of `i`

```rust
// BUG: elements cannot be reused because we skip past current index
backtrack(&cands, i + 1, remaining - cands[i], &mut path, &mut res);

// FIX for Combination Sum (reuse allowed)
backtrack(&cands, i, remaining - cands[i], &mut path, &mut res);
// Use i+1 only for Combination Sum II (no reuse)
```

---

## Quick Checklist Before Submitting

- [ ] Path cloned when adding to results (use `result.push(path.clone())`)?
- [ ] Undo step mirrors make step exactly (use `pop()` after `push()`)?
- [ ] `i > start` (not `i > 0`) for duplicate skip?
- [ ] Two diagonal sets in N-Queens?
- [ ] `!used[i-1]` (not `used[i-1]`) in Permutations II?
- [ ] Board cell restored after Word Search DFS?
- [ ] `n as i64` cast in Pow(x, n)?
- [ ] `i` (not `i+1`) in Combination Sum reuse case?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Recursion Tree Visualization](./Recursion%20Tree%20Visualization.md)

> **Last Updated:** 2026-06-26
