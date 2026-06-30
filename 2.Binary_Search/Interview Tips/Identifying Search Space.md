# Identifying the Search Space — Binary Search on Answer

> **Topic:** [Binary Search](../README.md) · **Section:** Interview Tips
> **Tags:** `search-space` `monotone-predicate` `identification`

---

## The Three Questions

When you see a problem, ask:

1. **Is there a monotone property?**
   - "If I can do it with X units, I can also do it with X+1 units."
   - "If it's infeasible with X workers, it's also infeasible with X-1 workers."

2. **Can I verify a candidate answer in O(n) or O(n log n)?**
   - Given a fixed value `mid`, can I check if it's feasible cheaply?

3. **Can I define clear `lo` and `hi`?**
   - `lo` = minimum possible answer (often 1, or max of some sub-problem)
   - `hi` = maximum possible answer (often sum, or n × max)

If all three → **Binary Search on Answer**.

---

## Recognition Table

| Problem Type | lo | hi | Predicate |
|--------------|----|----|-----------|
| Min speed to complete in H time | 1 | max(values) | `totalTime(speed) <= H` |
| Min capacity for D days | max(arr) | sum(arr) | `daysNeeded(cap) <= D` |
| Min max subarray sum for K parts | max(arr) | sum(arr) | `parts(maxSum) <= K` |
| Max min distance (aggressive cows) | 1 | max-min | `cows(minDist) >= K` |
| Kth missing positive | 1 | k + n | `missing(mid) < k` |
| Sqrt(x) integer | 1 | x | `mid * mid <= x` |

---

## The Monotone Predicate Test

Draw a mental number line:

```
Answer space:  1  2  3  4  5  6  7  8  9  10
isFeasible:    F  F  F  F  T  T  T  T  T   T
                            ↑ find this threshold
```

If the predicate flips from F to T exactly once → binary search works perfectly.

---

## Common Pitfall: Non-monotone Predicates

Binary search on answer only works for **unimodal** predicates. Some predicates are not monotone:

```
// WRONG use case: "find value where sum is exactly K"
// Feasibility: sum == K — this is NOT monotone (can be F, T, F, T...)
// → Use prefix sum + hashmap instead
```

---

## Related Files

- [Binary Search on Answer Pattern](../Patterns/Binary%20Search%20on%20Answer.md)
- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Binary Search README](../README.md)

> **Last Updated:** 2026-06-26
