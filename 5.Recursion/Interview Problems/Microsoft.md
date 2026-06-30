# Microsoft Interview Problems — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Generate Parentheses | SDE I | Medium | ⭐⭐⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [22](https://leetcode.com/problems/generate-parentheses/) |
| 2 | Permutations | SDE I | Medium | ⭐⭐⭐⭐ | [Permutations](../Patterns/Permutations.md) | [46](https://leetcode.com/problems/permutations/) |
| 3 | Sudoku Solver | SDE II | Hard | ⭐⭐⭐ | [Backtracking](../Patterns/Backtracking.md) | [37](https://leetcode.com/problems/sudoku-solver/) |
| 4 | Combination Sum II | SDE I | Medium | ⭐⭐⭐ | [Subsets & Combos](../Patterns/Subsets%20and%20Combinations.md) | [40](https://leetcode.com/problems/combination-sum-ii/) |

---

## Deep Dive: Permutations — Two Approaches

Microsoft commonly asks which approach you prefer and why:

**`used[]` approach:** Preferred when duplicates are present (Permutations II) or when you need to maintain the original order (sorted output). More readable.

**Swap approach:** Preferred when O(1) extra space matters. Can be surprising to trace if interviewer is unfamiliar — explain it clearly.

```java
// At position `start`, try placing each unplaced element
for (int i = start; i < nums.length; i++) {
    swap(nums, start, i);         // element i is now at position start
    permuteHelper(nums, start+1); // fix positions start+1 ... n-1
    swap(nums, start, i);         // restore — element i back to its original spot
}
```

**Microsoft follow-up:** "Does the swap approach work for Permutations II?"
Answer: Not directly — it doesn't handle duplicates cleanly. Easier to sort + `used[]` for duplicates.

---

## Related Files

- [Microsoft OA-Qns](../OA-Qns/Microsoft.md)
- [Recursion README](../README.md)

> **Last Updated:** 2026-06-26
