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

```rust
fn subsets(nums: &[i32]) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(nums, 0, &mut path, &mut result);
    result
}

fn backtrack(nums: &[i32], start: usize, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    res.push(path.clone()); // add at EVERY node, not just leaves
    for i in start..nums.len() {
        path.push(nums[i]);
        backtrack(nums, i + 1, path, res); // i+1: each element used at most once
        path.pop();
    }
}
```

---

## Template 2 — Subsets II (LC 90, with duplicates)

```rust
fn subsets_with_dup(nums: &mut Vec<i32>) -> Vec<Vec<i32>> {
    nums.sort(); // MUST sort to group duplicates
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(nums, 0, &mut path, &mut result);
    result
}

fn backtrack(nums: &[i32], start: usize, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    res.push(path.clone());
    for i in start..nums.len() {
        // Skip duplicates at the SAME recursion level (not within a branch)
        if i > start && nums[i] == nums[i - 1] { continue; }
        path.push(nums[i]);
        backtrack(nums, i + 1, path, res);
        path.pop();
    }
}
```

**Why `i > start` not `i > 0`?**
- `i > 0`: skips the duplicate even at the first position in a new branch — loses valid subsets
- `i > start`: only skips if we're at the same level as a previous duplicate pick

---

## Template 3 — Combination Sum (LC 39, reuse allowed)

Same element can be used unlimited times → pass `i` not `i+1` on recursion.

```rust
fn combination_sum(candidates: &mut Vec<i32>, target: i32) -> Vec<Vec<i32>> {
    candidates.sort(); // enables pruning
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(candidates, 0, target, &mut path, &mut result);
    result
}

fn backtrack(cands: &[i32], start: usize, remaining: i32, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    if remaining == 0 { res.push(path.clone()); return; }
    for i in start..cands.len() {
        if cands[i] > remaining { break; } // pruning: sorted, so rest also > remaining
        path.push(cands[i]);
        backtrack(cands, i, remaining - cands[i], path, res); // i, NOT i+1 (reuse OK)
        path.pop();
    }
}
```

---

## Template 4 — Combination Sum II (LC 40, no reuse, duplicates in input)

Each element used at most once; input may have duplicates; no duplicate combinations.

```rust
fn combination_sum2(candidates: &mut Vec<i32>, target: i32) -> Vec<Vec<i32>> {
    candidates.sort();
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(candidates, 0, target, &mut path, &mut result);
    result
}

fn backtrack(cands: &[i32], start: usize, remaining: i32, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    if remaining == 0 { res.push(path.clone()); return; }
    for i in start..cands.len() {
        if cands[i] > remaining { break; }
        if i > start && cands[i] == cands[i - 1] { continue; } // skip duplicate at same level
        path.push(cands[i]);
        backtrack(cands, i + 1, remaining - cands[i], path, res); // i+1: no reuse
        path.pop();
    }
}
```

---

## Template 5 — Combination Sum III (LC 216)

Exactly k numbers from 1–9 summing to n.

```rust
fn combination_sum3(k: usize, n: i32) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    let mut path = Vec::new();
    backtrack(1, k, n, &mut path, &mut result);
    result
}

fn backtrack(start: i32, k: usize, remaining: i32, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    if path.len() == k && remaining == 0 { res.push(path.clone()); return; }
    if path.len() == k || remaining <= 0 { return; } // prune
    for i in start..=9 {
        if i > remaining { break; } // prune
        path.push(i);
        backtrack(i + 1, k, remaining - i, path, res);
        path.pop();
    }
}
```

---

## Template 6 — Letter Combinations of Phone Number (LC 17)

```rust
fn letter_combinations(digits: &str) -> Vec<String> {
    const MAP: [&str; 10] = ["", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"];
    let mut result = Vec::new();
    if digits.is_empty() { return result; }
    let mut path = String::new();
    backtrack(digits.as_bytes(), 0, &mut path, &mut result, &MAP);
    result
}

fn backtrack(digits: &[u8], i: usize, path: &mut String, res: &mut Vec<String>, map: &[&str; 10]) {
    if i == digits.len() { res.push(path.clone()); return; }
    for ch in map[(digits[i] - b'0') as usize].chars() {
        path.push(ch);
        backtrack(digits, i + 1, path, res, map);
        path.pop();
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

```rust
if i > start && nums[i] == nums[i - 1] { continue; }
```

- **Sort first** — duplicates are adjacent
- **`i > start`** — only skip at the same level; don't skip the first pick at this depth
- **Never `i > 0`** — that would skip valid first picks in sub-branches

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not copying `path` on add | `path.clone()` — stores a snapshot (or just `path` when passed by value) |
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
