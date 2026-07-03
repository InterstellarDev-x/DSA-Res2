# Permutations

> **Topic:** [Recursion](../README.md) · **Section:** Patterns
> **Applies to:** Permutations I/II, Next Permutation, String Permutations

---

## Core Idea

A permutation uses every element exactly once in a specific order. Unlike subsets (which use a `start` index), permutations can pick any unused element at each position.

Two implementation strategies:
1. **`used[]` boolean array** — check each element; clean to understand
2. **In-place swap** — no extra array; swap element into current position, recurse, swap back

---

## Template 1 — Permutations via used[] (LC 46)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> permute(vector<int>& nums) {
    vector<vector<int>> result;
    vector<bool> used(nums.size(), false);
    vector<int> path;
    backtrack(nums, used, path, result);
    return result;
}

void backtrack(vector<int>& nums, vector<bool>& used, vector<int>& path, vector<vector<int>>& res) {
    if (path.size() == nums.size()) { res.push_back(path); return; }
    for (int i = 0; i < nums.size(); i++) {
        if (used[i]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        backtrack(nums, used, path, res);
        path.pop_back();
        used[i] = false;
    }
}
```

---

## Template 2 — Permutations via Swap (LC 46, O(1) extra space)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> permute(vector<int> nums) {
    vector<vector<int>> result;
    permuteHelper(nums, 0, result);
    return result;
}

void permuteHelper(vector<int>& nums, int start, vector<vector<int>>& res) {
    if (start == nums.size()) {
        // Collect current array state as a permutation
        res.push_back(nums);
        return;
    }
    for (int i = start; i < nums.size(); i++) {
        swap(nums[start], nums[i]);          // put nums[i] at position start
        permuteHelper(nums, start + 1, res);
        swap(nums[start], nums[i]);          // restore
    }
}
```

**Visualization for [1,2,3] at start=0:**
```
i=0: [1,2,3] → recurse start=1
i=1: [2,1,3] → recurse start=1
i=2: [3,2,1] → recurse start=1
```

---

## Template 3 — Permutations II (LC 47, with duplicates)

Sort first. Skip if same value was already placed at the current position this level.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> permuteUnique(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> result;
    vector<bool> used(nums.size(), false);
    vector<int> path;
    backtrack(nums, used, path, result);
    return result;
}

void backtrack(vector<int>& nums, vector<bool>& used, vector<int>& path, vector<vector<int>>& res) {
    if (path.size() == nums.size()) { res.push_back(path); return; }
    for (int i = 0; i < nums.size(); i++) {
        if (used[i]) continue;
        // Skip: same value AND previous duplicate not used
        // Ensures duplicates are placed in order (prevents same-level reuse)
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
        used[i] = true;
        path.push_back(nums[i]);
        backtrack(nums, used, path, res);
        path.pop_back();
        used[i] = false;
    }
}
```

**The `!used[i-1]` condition explained:**
If `nums[i-1] == nums[i]` and `used[i-1]` is false, it means `nums[i-1]` was previously tried and backtracked at this same level. Allowing `nums[i]` (same value) here would generate a duplicate. Skipping enforces that identical values are always placed in left-to-right order.

---

## Template 4 — String Permutations (all permutations of a string)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> permutations(string s) {
    vector<string> result;
    sort(s.begin(), s.end());
    vector<bool> used(s.size(), false);
    string current;
    permuteStr(s, used, current, result);
    return result;
}

void permuteStr(const string& s, vector<bool>& used, string& current, vector<string>& res) {
    if (current.size() == s.size()) { res.push_back(current); return; }
    for (int i = 0; i < s.size(); i++) {
        if (used[i]) continue;
        if (i > 0 && s[i] == s[i - 1] && !used[i - 1]) continue; // skip dup
        used[i] = true;
        current += s[i];
        permuteStr(s, used, current, res);
        current.pop_back();
        used[i] = false;
    }
}
```

---

## Next Permutation (LC 31) — Iterative, O(n) O(1)

Finds the lexicographically next permutation in-place:

```cpp
#include <bits/stdc++.h>
using namespace std;

void nextPermutation(vector<int>& nums) {
    int n = nums.size();
    // Step 1: find largest i where nums[i] < nums[i+1] (rightmost "dip")
    int i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;

    if (i >= 0) {
        // Step 2: find smallest j > i where nums[j] > nums[i]
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;
        swap(nums[i], nums[j]);
    }
    // Step 3: reverse suffix from i+1 to end
    reverse(nums.begin() + i + 1, nums.end());
}
```

---

## Comparison: used[] vs Swap

| | `used[]` | Swap |
|--|----------|------|
| Extra space | O(n) for `used[]` | O(1) |
| Order preserved | Yes — generates in sorted order | No — depends on swap order |
| Duplicate handling | `!used[i-1]` trick | Harder with swap |
| Readability | Clearer | Lower |
| Preferred when | Duplicates present | Space-constrained |

---

## Counting Permutations

| Scenario | Count |
|----------|-------|
| n distinct elements | n! |
| n elements with duplicates: a copies of A, b of B... | n! / (a! × b! × ...) |
| Permutations of length k from n elements | n! / (n-k)! = P(n,k) |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| `i > 0 && !used[i-1]` — condition reversed | `nums[i] == nums[i-1] && !used[i-1]` — check value equality AND parent not used |
| In swap approach: not restoring (forgetting second swap) | Both `swap(nums[start], nums[i])` calls are required |
| Storing reference to `path` | `res.push_back(path)` copies by value at the leaf |
| Swap approach: collecting result without copying | Copy `nums` explicitly: `res.push_back(nums)` |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Permutations (distinct) | O(n × n!) | O(n) stack + used |
| Permutations II (duplicates) | O(n × n!) with pruning | O(n) |
| Next Permutation | O(n) | O(1) |

---

## Related Patterns

- [Subsets & Combinations](./Subsets%20and%20Combinations.md) — ordered vs unordered selection
- [Backtracking](./Backtracking.md) — try/recurse/undo framework
- [Bit Manipulation](../../6.Bit_Manipulation/README.md) — bitmask-based permutation enumeration

---

**Back:** [Recursion README](../README.md) | **Prev:** [Subsets & Combinations](./Subsets%20and%20Combinations.md) | **Next:** [Divide & Conquer](./Divide%20and%20Conquer.md)

> **Last Updated:** 2026-06-26
