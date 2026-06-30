# Complexity Analysis — Binary Search

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Tips

---

## Quick Reference

| Pattern | Time | Space | Notes |
|---------|------|-------|-------|
| [Classic BS](../Patterns/Classic%20Binary%20Search.md) | O(log n) | O(1) | Each step halves search space |
| [Lower/Upper Bound](../Patterns/Lower%20and%20Upper%20Bound.md) | O(log n) | O(1) | Same structure |
| [BS on Answer](../Patterns/Binary%20Search%20on%20Answer.md) | O(n log R) | O(1) | R = answer range; n = feasibility check |
| [BS on 2D — flatten](../Patterns/Binary%20Search%20on%202D.md) | O(log(m×n)) | O(1) | Treat as 1D array |
| [Staircase (2D)](../Patterns/Binary%20Search%20on%202D.md) | O(m+n) | O(1) | Not true binary search |
| [Peak Finding](../Patterns/Peak%20Finding.md) | O(log n) | O(1) | |
| Median of 2 sorted arrays | O(log(min(m,n))) | O(1) | BS on shorter array |
| Kth smallest in sorted matrix | O((m+n) log(max-min)) | O(1) | |

---

## How to Derive O(n log R) for BS on Answer

```
Loop runs: log₂(hi - lo) = log₂(R) times       [R = answer range]
Each iteration: feasibility check costs O(n)
Total: O(n × log R)

For Koko: R = max(piles) ≤ 10^9, so log R ≈ 30
          n = number of piles
          Total = O(n × 30) = O(n log maxPile)

For Ship: R = sum(weights) ≤ n × max_weight
          Total = O(n log(n × max_weight))
```

---

## Space Complexity Notes

All iterative binary search variants are **O(1) auxiliary space**.
Recursive binary search is O(log n) due to call stack depth.

> **Interview Tip:** Always state "iterative implementation" to signal O(1) space.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Binary Search README](../README.md)

> **Last Updated:** 2026-06-26
