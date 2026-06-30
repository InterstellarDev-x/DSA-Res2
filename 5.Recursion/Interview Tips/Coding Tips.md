# Coding Tips — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Backtracking Template — Never Deviate

```java
void backtrack(state, choices, results) {
    if (goal reached) {
        results.add(copy of state);  // COPY, not reference
        return;
    }
    for (choice : validChoices) {
        makeChoice(state, choice);   // mutate state
        backtrack(state, ...);       // recurse
        undoChoice(state, choice);   // restore state exactly
    }
}
```

Every backtracking problem fits this template. Identify:
1. What is `state`? (path list, grid cell, queen positions)
2. What is `validChoices`? (digits 1–9, positions 0..n-1, indices start..n)
3. What is `copy`? (`new ArrayList<>(path)`, `board.clone()`)
4. What is `undo`? (`path.remove(last)`, `board[r][c] = '.'`)

---

## StringBuilder for Path (not String)

```java
// SLOW — O(n) per concat, O(n²) total for path of length n
String path = "";
path += ch;         // new String object every time
path = path.substring(0, path.length()-1); // undo is O(n)

// FAST — O(1) amortized append/delete
StringBuilder sb = new StringBuilder();
sb.append(ch);
sb.deleteCharAt(sb.length() - 1); // O(1) undo
```

Rule: If you're building a string incrementally in a recursion, always use `StringBuilder`.

---

## List Path Copy (not Reference)

```java
// BUG: all results point to the same list object
result.add(path); // adds reference

// FIX: snapshot the current state
result.add(new ArrayList<>(path));
```

---

## Sorting Before Backtracking

Always sort the input before backtracking when:
1. You need to skip duplicates (`i > start && nums[i] == nums[i-1]`)
2. You want to prune with `if (nums[i] > remaining) break`

```java
Arrays.sort(nums); // O(n log n) one time — enables O(1) pruning per node
```

---

## Integer.MIN_VALUE in Pow(x, n)

```java
// OVERFLOW: -Integer.MIN_VALUE = Integer.MIN_VALUE (still negative!)
int n = Integer.MIN_VALUE;
n = -n; // BUG: still MIN_VALUE

// FIX: cast to long before negating
long N = (long) n;
N = -N; // correct: 2147483648
```

---

## In-Place Grid Marking (Word Search)

Instead of a separate `boolean[][] visited` array:

```java
char tmp = board[r][c];
board[r][c] = '#';  // mark — any non-letter sentinel
dfs(board, r+1, c, word, k+1);
board[r][c] = tmp;  // restore
```

Saves O(m×n) extra space.

---

## `used[i-1]` Duplicate Trick in Permutations II

```java
// Skip duplicate at same recursion level:
if (i > 0 && nums[i] == nums[i-1] && !used[i-1]) continue;
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
