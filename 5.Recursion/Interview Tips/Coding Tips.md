# Coding Tips — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Backtracking Template — Never Deviate

```rust
fn backtrack(state: &mut State, valid_choices: &[Choice], results: &mut Vec<State>) {
    if goal_reached(state) {
        results.push(state.clone()); // clone to capture current state
        return;
    }
    for choice in valid_choices {
        make_choice(state, choice);   // mutate state
        backtrack(state, ...);        // recurse
        undo_choice(state, choice);   // restore state exactly
    }
}
```

Every backtracking problem fits this template. Identify:
1. What is `state`? (path list, grid cell, queen positions)
2. What is `valid_choices`? (digits 1–9, positions 0..n-1, indices start..n)
3. What is `copy`? (`path.clone()` — or just `path` since `push` clones on capture, `board` snapshot via `.clone()`)
4. What is `undo`? (`path.pop()`, `board[r][c] = '.'`)

---

## String with pop for Path (not concat)

```rust
// SLOW — O(n) per concat, O(n²) total for path of length n
let mut path = String::new();
path.push(ch);                              // push a char O(1) amortized
path = path[..path.len() - 1].to_string(); // undo is O(n) — reallocates

// FAST — O(1) amortized append/delete
let mut sb = String::new();
sb.push(ch);
sb.pop(); // O(1) undo — returns Option<char>
```

Rule: If you're building a string incrementally in a recursion, always use `String` with `pop()` for undo instead of a slice-and-clone.

---

## Vec Path Clone (not Reference)

```rust
// NOTE: In Rust, clone() explicitly copies the Vec — no aliasing bug
result.push(path.clone()); // safe: clones the current state

// Equivalent (same thing):
result.push(path.to_vec());
```

---

## Sorting Before Backtracking

Always sort the input before backtracking when:
1. You need to skip duplicates (`i > start && nums[i] == nums[i-1]`)
2. You want to prune with `if nums[i] > remaining { break; }`

```rust
nums.sort(); // O(n log n) one time — enables O(1) pruning per node
```

---

## i32::MIN in Pow(x, n)

```rust
// OVERFLOW: i32::MIN.wrapping_neg() = i32::MIN (still negative; debug mode panics!)
let n: i32 = i32::MIN;
let _ = n.wrapping_neg(); // BUG: still i32::MIN

// FIX: cast to i64 before negating
let big_n: i64 = n as i64;
let big_n = -big_n; // correct: 2147483648
```

---

## In-Place Grid Marking (Word Search)

Instead of a separate `Vec<Vec<bool>>` visited array:

```rust
let tmp = board[r][c];
board[r][c] = '#'; // mark — any non-letter sentinel
dfs(board, r + 1, c, word, k + 1);
board[r][c] = tmp;  // restore
```

Saves O(m×n) extra space.

---

## `used[i-1]` Duplicate Trick in Permutations II

```rust
// Skip duplicate at same recursion level:
if i > 0 && nums[i] == nums[i - 1] && !used[i - 1] { continue; }
```

Mental model: among equal elements, we always pick them left-to-right. If `nums[i-1]` was picked and backtracked (`!used[i-1]`), picking `nums[i]` (same value) again at this level would create a duplicate permutation.

---

## start Index vs 0 in Combinations vs Permutations

| Problem | Start Next Recursion At | Reason |
|---------|------------------------|--------|
| Subsets / Combinations | `i + 1` | Forward only, no repeats |
| Combination Sum I | `i` | Allow reuse of same element |
| Permutations | `0` (with `used[]`) | Any position; no forward bias |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Recursion Tree Visualization](./Recursion%20Tree%20Visualization.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
