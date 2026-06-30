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

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> res) {
    if (path.size() == nums.length) { res.add(new ArrayList<>(path)); return; }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, res);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

---

## Template 2 — Permutations via Swap (LC 46, O(1) extra space)

```java
public List<List<Integer>> permute(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    permuteHelper(nums, 0, result);
    return result;
}

private void permuteHelper(int[] nums, int start, List<List<Integer>> res) {
    if (start == nums.length) {
        // Collect current array state as a permutation
        List<Integer> perm = new ArrayList<>();
        for (int n : nums) perm.add(n);
        res.add(perm);
        return;
    }
    for (int i = start; i < nums.length; i++) {
        swap(nums, start, i);          // put nums[i] at position start
        permuteHelper(nums, start + 1, res);
        swap(nums, start, i);          // restore
    }
}

private void swap(int[] a, int i, int j) { int tmp = a[i]; a[i] = a[j]; a[j] = tmp; }
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

```java
public List<List<Integer>> permuteUnique(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();
    backtrack(nums, new boolean[nums.length], new ArrayList<>(), result);
    return result;
}

private void backtrack(int[] nums, boolean[] used, List<Integer> path, List<List<Integer>> res) {
    if (path.size() == nums.length) { res.add(new ArrayList<>(path)); return; }
    for (int i = 0; i < nums.length; i++) {
        if (used[i]) continue;
        // Skip: same value AND previous duplicate not used
        // Ensures duplicates are placed in order (prevents same-level reuse)
        if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;
        used[i] = true;
        path.add(nums[i]);
        backtrack(nums, used, path, res);
        path.remove(path.size() - 1);
        used[i] = false;
    }
}
```

**The `!used[i-1]` condition explained:**
If `nums[i-1] == nums[i]` and `used[i-1]` is false, it means `nums[i-1]` was previously tried and backtracked at this same level. Allowing `nums[i]` (same value) here would generate a duplicate. Skipping enforces that identical values are always placed in left-to-right order.

---

## Template 4 — String Permutations (all permutations of a string)

```java
public List<String> permutations(String s) {
    List<String> result = new ArrayList<>();
    char[] arr = s.toCharArray();
    Arrays.sort(arr);
    boolean[] used = new boolean[arr.length];
    permuteStr(arr, used, new StringBuilder(), result);
    return result;
}

private void permuteStr(char[] arr, boolean[] used, StringBuilder sb, List<String> res) {
    if (sb.length() == arr.length) { res.add(sb.toString()); return; }
    for (int i = 0; i < arr.length; i++) {
        if (used[i]) continue;
        if (i > 0 && arr[i] == arr[i - 1] && !used[i - 1]) continue; // skip dup
        used[i] = true;
        sb.append(arr[i]);
        permuteStr(arr, used, sb, res);
        sb.deleteCharAt(sb.length() - 1);
        used[i] = false;
    }
}
```

---

## Next Permutation (LC 31) — Iterative, O(n) O(1)

Finds the lexicographically next permutation in-place:

```java
public void nextPermutation(int[] nums) {
    int n = nums.length;
    // Step 1: find largest i where nums[i] < nums[i+1] (rightmost "dip")
    int i = n - 2;
    while (i >= 0 && nums[i] >= nums[i + 1]) i--;

    if (i >= 0) {
        // Step 2: find smallest j > i where nums[j] > nums[i]
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;
        swap(nums, i, j);
    }
    // Step 3: reverse suffix from i+1 to end
    reverse(nums, i + 1, n - 1);
}
```

---

## Comparison: used[] vs Swap

| | `used[]` | Swap |
|--|----------|------|
| Extra space | O(n) for `used[]` | O(1) |
| Order preserved | Yes — generates in sorted order | No — depends on swap order |
| Duplicate handling | `!used[i-1]` trick | Harder with swap |
| Readability | ✅ Clearer | Lower |
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
| In swap approach: not restoring (forgetting second swap) | Both `swap(nums, start, i)` calls are required |
| Storing reference to `path` | `new ArrayList<>(path)` at the leaf |
| Swap approach: collecting result with `Arrays.asList(nums)` | That shares the backing array; copy explicitly |

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
