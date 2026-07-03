# Coding Tips — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Backtracking Template — Never Deviate

```cpp
#include <bits/stdc++.h>
using namespace std;

void backtrack(state, choices, results) {
    if (goal reached) {
        results.push_back(state);  // vector copies by value — no explicit copy needed
        return;
    }
    for (auto& choice : validChoices) {
        makeChoice(state, choice);   // mutate state
        backtrack(state, ...);       // recurse
        undoChoice(state, choice);   // restore state exactly
    }
}
```

Every backtracking problem fits this template. Identify:
1. What is `state`? (path list, grid cell, queen positions)
2. What is `validChoices`? (digits 1–9, positions 0..n-1, indices start..n)
3. What is `copy`? (`vector<int>(path)` — or just `path` since `push_back` copies, `board` snapshot via copy)
4. What is `undo`? (`path.pop_back()`, `board[r][c] = '.'`)

---

## String with pop_back for Path (not concat)

```cpp
#include <bits/stdc++.h>
using namespace std;

// SLOW — O(n) per concat, O(n²) total for path of length n
string path = "";
path += ch;                        // creates new string object every time via +=
path = path.substr(0, path.length()-1); // undo is O(n)

// FAST — O(1) amortized append/delete
string sb = "";
sb += ch;
sb.pop_back(); // O(1) undo
```

Rule: If you're building a string incrementally in a recursion, always use `string` with `pop_back()` for undo instead of `substr`.

---

## Vector Path Copy (not Reference)

```cpp
#include <bits/stdc++.h>
using namespace std;

// NOTE: In C++, push_back already copies the vector — no pointer aliasing bug
result.push_back(path); // safe: copies the current state

// Explicit copy (equivalent, sometimes clearer):
result.push_back(vector<int>(path));
```

---

## Sorting Before Backtracking

Always sort the input before backtracking when:
1. You need to skip duplicates (`i > start && nums[i] == nums[i-1]`)
2. You want to prune with `if (nums[i] > remaining) break`

```cpp
#include <bits/stdc++.h>
using namespace std;

sort(nums.begin(), nums.end()); // O(n log n) one time — enables O(1) pruning per node
```

---

## INT_MIN in Pow(x, n)

```cpp
#include <bits/stdc++.h>
using namespace std;

// OVERFLOW: -INT_MIN = INT_MIN (still negative!)
int n = INT_MIN;
n = -n; // BUG: still INT_MIN

// FIX: cast to long before negating
long N = (long) n;
N = -N; // correct: 2147483648
```

---

## In-Place Grid Marking (Word Search)

Instead of a separate `vector<vector<bool>> visited` array:

```cpp
char tmp = board[r][c];
board[r][c] = '#';  // mark — any non-letter sentinel
dfs(board, r+1, c, word, k+1);
board[r][c] = tmp;  // restore
```

Saves O(m×n) extra space.

---

## `used[i-1]` Duplicate Trick in Permutations II

```cpp
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
