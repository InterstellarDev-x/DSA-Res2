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

```rust
fn permute(nums: &[i32]) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    let mut used = vec![false; nums.len()];
    let mut path = Vec::new();
    backtrack(nums, &mut used, &mut path, &mut result);
    result
}

fn backtrack(nums: &[i32], used: &mut Vec<bool>, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    if path.len() == nums.len() { res.push(path.clone()); return; }
    for i in 0..nums.len() {
        if used[i] { continue; }
        used[i] = true;
        path.push(nums[i]);
        backtrack(nums, used, path, res);
        path.pop();
        used[i] = false;
    }
}
```

---

## Template 2 — Permutations via Swap (LC 46, O(1) extra space)

```rust
fn permute(mut nums: Vec<i32>) -> Vec<Vec<i32>> {
    let mut result = Vec::new();
    permute_helper(&mut nums, 0, &mut result);
    result
}

fn permute_helper(nums: &mut Vec<i32>, start: usize, res: &mut Vec<Vec<i32>>) {
    if start == nums.len() {
        // Collect current array state as a permutation
        res.push(nums.clone());
        return;
    }
    for i in start..nums.len() {
        nums.swap(start, i);          // put nums[i] at position start
        permute_helper(nums, start + 1, res);
        nums.swap(start, i);          // restore
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

```rust
fn permute_unique(mut nums: Vec<i32>) -> Vec<Vec<i32>> {
    nums.sort();
    let mut result = Vec::new();
    let mut used = vec![false; nums.len()];
    let mut path = Vec::new();
    backtrack(&nums, &mut used, &mut path, &mut result);
    result
}

fn backtrack(nums: &[i32], used: &mut Vec<bool>, path: &mut Vec<i32>, res: &mut Vec<Vec<i32>>) {
    if path.len() == nums.len() { res.push(path.clone()); return; }
    for i in 0..nums.len() {
        if used[i] { continue; }
        // Skip: same value AND previous duplicate not used
        // Ensures duplicates are placed in order (prevents same-level reuse)
        if i > 0 && nums[i] == nums[i - 1] && !used[i - 1] { continue; }
        used[i] = true;
        path.push(nums[i]);
        backtrack(nums, used, path, res);
        path.pop();
        used[i] = false;
    }
}
```

**The `!used[i-1]` condition explained:**
If `nums[i-1] == nums[i]` and `used[i-1]` is false, it means `nums[i-1]` was previously tried and backtracked at this same level. Allowing `nums[i]` (same value) here would generate a duplicate. Skipping enforces that identical values are always placed in left-to-right order.

---

## Template 4 — String Permutations (all permutations of a string)

```rust
fn permutations(s: &str) -> Vec<String> {
    let mut chars: Vec<char> = s.chars().collect();
    chars.sort();
    let mut result = Vec::new();
    let mut used = vec![false; chars.len()];
    let mut current = String::new();
    permute_str(&chars, &mut used, &mut current, &mut result);
    result
}

fn permute_str(s: &[char], used: &mut Vec<bool>, current: &mut String, res: &mut Vec<String>) {
    if current.chars().count() == s.len() { res.push(current.clone()); return; }
    for i in 0..s.len() {
        if used[i] { continue; }
        if i > 0 && s[i] == s[i - 1] && !used[i - 1] { continue; } // skip dup
        used[i] = true;
        current.push(s[i]);
        permute_str(s, used, current, res);
        current.pop();
        used[i] = false;
    }
}
```

---

## Next Permutation (LC 31) — Iterative, O(n) O(1)

Finds the lexicographically next permutation in-place:

```rust
fn next_permutation(nums: &mut Vec<i32>) {
    let n = nums.len();
    // Step 1: find largest i where nums[i] < nums[i+1] (rightmost "dip")
    let mut i = n as i32 - 2;
    while i >= 0 && nums[i as usize] >= nums[i as usize + 1] {
        i -= 1;
    }

    if i >= 0 {
        // Step 2: find smallest j > i where nums[j] > nums[i]
        let mut j = n as i32 - 1;
        while nums[j as usize] <= nums[i as usize] {
            j -= 1;
        }
        nums.swap(i as usize, j as usize);
    }
    // Step 3: reverse suffix from i+1 to end
    nums[i as usize + 1..].reverse();
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
| In swap approach: not restoring (forgetting second swap) | Both `nums.swap(start, i)` calls are required |
| Storing reference to `path` | `res.push(path.clone())` copies by value at the leaf |
| Swap approach: collecting result without copying | Copy `nums` explicitly: `res.push(nums.clone())` |

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
