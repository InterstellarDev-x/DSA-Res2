# Subsets & Combinations

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** Subsets I/II, Combination Sum I/II/III, Letter Combinations

---

## Core Decision Tree

At each element, make a binary decision: **include** or **exclude**. This generates 2^n leaves for n elements.

```
                   []
              /         \
          [1]            []
         /   \          /   \
      [1,2]  [1]      [2]   []
```

**Key control variables:**
- `start` — index from which to pick next element (prevents reuse and re-ordering)
- `remaining` / `target` — for combination sum problems
- Sort input first when handling duplicates

---

## Template 1 — Subsets (LC 78, no duplicates)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> subsets(vector<int>& nums) {
    vector<vector<int>> result;
    vector<int> path;
    backtrack(nums, 0, path, result);
    return result;
}

void backtrack(vector<int>& nums, int start, vector<int>& path, vector<vector<int>>& res) {
    res.push_back(path); // add at EVERY node, not just leaves
    for (int i = start; i < (int)nums.size(); i++) {
        path.push_back(nums[i]);
        backtrack(nums, i + 1, path, res); // i+1: each element used at most once
        path.pop_back();
    }
}
```

---

## Template 2 — Subsets II (LC 90, with duplicates)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> subsetsWithDup(vector<int>& nums) {
    sort(nums.begin(), nums.end()); // MUST sort to group duplicates
    vector<vector<int>> result;
    vector<int> path;
    backtrack(nums, 0, path, result);
    return result;
}

void backtrack(vector<int>& nums, int start, vector<int>& path, vector<vector<int>>& res) {
    res.push_back(path);
    for (int i = start; i < (int)nums.size(); i++) {
        // Skip duplicates at the SAME recursion level (not within a branch)
        if (i > start && nums[i] == nums[i - 1]) continue;
        path.push_back(nums[i]);
        backtrack(nums, i + 1, path, res);
        path.pop_back();
    }
}
```

**Why `i > start` not `i > 0`?**
- `i > 0`: skips the duplicate even at the first position in a new branch — loses valid subsets
- `i > start`: only skips if we're at the same level as a previous duplicate pick

---

## Template 3 — Combination Sum (LC 39, reuse allowed)

Same element can be used unlimited times → pass `i` not `i+1` on recursion.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end()); // enables pruning
    vector<vector<int>> result;
    vector<int> path;
    backtrack(candidates, 0, target, path, result);
    return result;
}

void backtrack(vector<int>& cands, int start, int remaining, vector<int>& path, vector<vector<int>>& res) {
    if (remaining == 0) { res.push_back(path); return; }
    for (int i = start; i < (int)cands.size(); i++) {
        if (cands[i] > remaining) break; // pruning: sorted, so rest also > remaining
        path.push_back(cands[i]);
        backtrack(cands, i, remaining - cands[i], path, res); // i, NOT i+1 (reuse OK)
        path.pop_back();
    }
}
```

---

## Template 4 — Combination Sum II (LC 40, no reuse, duplicates in input)

Each element used at most once; input may have duplicates; no duplicate combinations.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> combinationSum2(vector<int>& candidates, int target) {
    sort(candidates.begin(), candidates.end());
    vector<vector<int>> result;
    vector<int> path;
    backtrack(candidates, 0, target, path, result);
    return result;
}

void backtrack(vector<int>& cands, int start, int remaining, vector<int>& path, vector<vector<int>>& res) {
    if (remaining == 0) { res.push_back(path); return; }
    for (int i = start; i < (int)cands.size(); i++) {
        if (cands[i] > remaining) break;
        if (i > start && cands[i] == cands[i - 1]) continue; // skip duplicate at same level
        path.push_back(cands[i]);
        backtrack(cands, i + 1, remaining - cands[i], path, res); // i+1: no reuse
        path.pop_back();
    }
}
```

---

## Template 5 — Combination Sum III (LC 216)

Exactly k numbers from 1–9 summing to n.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> combinationSum3(int k, int n) {
    vector<vector<int>> result;
    vector<int> path;
    backtrack(1, k, n, path, result);
    return result;
}

void backtrack(int start, int k, int remaining, vector<int>& path, vector<vector<int>>& res) {
    if ((int)path.size() == k && remaining == 0) { res.push_back(path); return; }
    if ((int)path.size() == k || remaining <= 0) return; // prune
    for (int i = start; i <= 9; i++) {
        if (i > remaining) break; // prune
        path.push_back(i);
        backtrack(i + 1, k, remaining - i, path, res);
        path.pop_back();
    }
}
```

---

## Template 6 — Letter Combinations of Phone Number (LC 17)

```cpp
#include <bits/stdc++.h>
using namespace std;

static const string MAP[] = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

vector<string> letterCombinations(string digits) {
    vector<string> result;
    if (digits.empty()) return result;
    string path;
    backtrack(digits, 0, path, result);
    return result;
}

void backtrack(const string& digits, int i, string& path, vector<string>& res) {
    if (i == (int)digits.size()) { res.push_back(path); return; }
    for (char ch : MAP[digits[i] - '0']) {
        path += ch;
        backtrack(digits, i + 1, path, res);
        path.pop_back();
    }
}
```

---

## Decision Table: i vs i+1 vs 0

| Problem | Recursive Call | Reason |
|---------|---------------|--------|
| Subsets | `i + 1` | No reuse; forward only |
| Combination Sum I | `i` | Reuse allowed |
| Combination Sum II | `i + 1` | No reuse (but duplicates in input) |
| Permutations | `0` (with `used[]`) | Any unvisited index |

---

## Duplicate Skipping Rule (Memorize This)

```cpp
if (i > start && nums[i] == nums[i - 1]) continue;
```

- **Sort first** — duplicates are adjacent
- **`i > start`** — only skip at the same level; don't skip the first pick at this depth
- **Never `i > 0`** — that would skip valid first picks in sub-branches

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not copying `path` on add | `vector<int>(path)` — stores a snapshot (or just `path` when passed by value) |
| Adding to result at leaf only (subset) | Add at every node for subsets, at leaf only for combinations |
| Reuse: passing `i+1` instead of `i` | Combination Sum I requires `i` |
| Duplicate skip: `i > 0` instead of `i > start` | `i > start` preserves first pick at each level |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Subsets | O(n × 2^n) | O(n) stack |
| Subsets II | O(n × 2^n) | O(n) stack |
| Combination Sum I | O(2^(target/min)) — exponential | O(target/min) stack |
| Combination Sum II | O(2^n) | O(n) stack |
| Letter Combinations | O(4^n × n) | O(n) |

---

## Related Patterns

- [Permutations](./Permutations.md) — ordered selection (no `start` index)
- [Backtracking](./Backtracking.md) — generalized try/recurse/undo
- [Dynamic Programming](../../14.Dynamic_Programming/README.md) — when only count needed (no enumeration)

---

**Back:** [Recursion README](../README.md) | **Prev:** [Backtracking](./Backtracking.md) | **Next:** [Permutations](./Permutations.md)

> **Last Updated:** 2026-06-26
