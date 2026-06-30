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

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> res) {
    res.add(new ArrayList<>(path)); // add at EVERY node, not just leaves
    for (int i = start; i < nums.length; i++) {
        path.add(nums[i]);
        backtrack(nums, i + 1, path, res); // i+1: each element used at most once
        path.remove(path.size() - 1);
    }
}
```

---

## Template 2 — Subsets II (LC 90, with duplicates)

```java
public List<List<Integer>> subsetsWithDup(int[] nums) {
    Arrays.sort(nums); // MUST sort to group duplicates
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, 0, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, int start, List<Integer> path, List<List<Integer>> res) {
    res.add(new ArrayList<>(path));
    for (int i = start; i < nums.length; i++) {
        // Skip duplicates at the SAME recursion level (not within a branch)
        if (i > start && nums[i] == nums[i - 1]) continue;
        path.add(nums[i]);
        backtrack(nums, i + 1, path, res);
        path.remove(path.size() - 1);
    }
}
```

**Why `i > start` not `i > 0`?**
- `i > 0`: skips the duplicate even at the first position in a new branch — loses valid subsets
- `i > start`: only skips if we're at the same level as a previous duplicate pick

---

## Template 3 — Combination Sum (LC 39, reuse allowed)

Same element can be used unlimited times → pass `i` not `i+1` on recursion.

```java
public List<List<Integer>> combinationSum(int[] candidates, int target) {
    Arrays.sort(candidates); // enables pruning
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, 0, target, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] cands, int start, int remaining, List<Integer> path, List<List<Integer>> res) {
    if (remaining == 0) { res.add(new ArrayList<>(path)); return; }
    for (int i = start; i < cands.length; i++) {
        if (cands[i] > remaining) break; // pruning: sorted, so rest also > remaining
        path.add(cands[i]);
        backtrack(cands, i, remaining - cands[i], path, res); // i, NOT i+1 (reuse OK)
        path.remove(path.size() - 1);
    }
}
```

---

## Template 4 — Combination Sum II (LC 40, no reuse, duplicates in input)

Each element used at most once; input may have duplicates; no duplicate combinations.

```java
public List<List<Integer>> combinationSum2(int[] candidates, int target) {
    Arrays.sort(candidates);
    List<List<Integer>> result = new ArrayList<>();
    backtrack(candidates, 0, target, new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] cands, int start, int remaining, List<Integer> path, List<List<Integer>> res) {
    if (remaining == 0) { res.add(new ArrayList<>(path)); return; }
    for (int i = start; i < cands.length; i++) {
        if (cands[i] > remaining) break;
        if (i > start && cands[i] == cands[i - 1]) continue; // skip duplicate at same level
        path.add(cands[i]);
        backtrack(cands, i + 1, remaining - cands[i], path, res); // i+1: no reuse
        path.remove(path.size() - 1);
    }
}
```

---

## Template 5 — Combination Sum III (LC 216)

Exactly k numbers from 1–9 summing to n.

```java
public List<List<Integer>> combinationSum3(int k, int n) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(1, k, n, new ArrayList<>(), result);
    return result;
}

private void backtrack(int start, int k, int remaining, List<Integer> path, List<List<Integer>> res) {
    if (path.size() == k && remaining == 0) { res.add(new ArrayList<>(path)); return; }
    if (path.size() == k || remaining <= 0) return; // prune
    for (int i = start; i <= 9; i++) {
        if (i > remaining) break; // prune
        path.add(i);
        backtrack(i + 1, k, remaining - i, path, res);
        path.remove(path.size() - 1);
    }
}
```

---

## Template 6 — Letter Combinations of Phone Number (LC 17)

```java
private static final String[] MAP = {"", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"};

public List<String> letterCombinations(String digits) {
    List<String> result = new ArrayList<>();
    if (digits.isEmpty()) return result;
    backtrack(digits, 0, new StringBuilder(), result);
    return result;
}

private void backtrack(String digits, int i, StringBuilder sb, List<String> res) {
    if (i == digits.length()) { res.add(sb.toString()); return; }
    for (char ch : MAP[digits.charAt(i) - '0'].toCharArray()) {
        sb.append(ch);
        backtrack(digits, i + 1, sb, res);
        sb.deleteCharAt(sb.length() - 1);
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

```java
if (i > start && nums[i] == nums[i - 1]) continue;
```

- **Sort first** — duplicates are adjacent
- **`i > start`** — only skip at the same level; don't skip the first pick at this depth
- **Never `i > 0`** — that would skip valid first picks in sub-branches

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not copying `path` on add | `new ArrayList<>(path)` — stores a snapshot |
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
