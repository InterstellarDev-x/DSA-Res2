# Complexity Analysis — Recursion

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Pattern-to-Complexity Quick Reference

| Problem | Time | Space (Stack) | Notes |
|---------|------|--------------|-------|
| [Basic Recursion](../Patterns/Basic%20Recursion.md) — factorial | O(n) | O(n) | Linear recursion |
| Fibonacci (naive) | O(2^n) | O(n) | Exponential recomputation |
| Fibonacci (memoized) | O(n) | O(n) | Each fib(k) once |
| [Pow(x, n)](../Patterns/Divide%20and%20Conquer.md) | O(log n) | O(log n) → O(1) iterative | Halves per level |
| [Subsets](../Patterns/Subsets%20and%20Combinations.md) | O(n × 2^n) | O(n) | 2^n subsets × O(n) copy |
| [Combination Sum](../Patterns/Subsets%20and%20Combinations.md) | O(2^(target/min)) | O(target/min) | Worst case: all 1s, target = T |
| [Permutations](../Patterns/Permutations.md) | O(n × n!) | O(n) | n! perms × O(n) copy |
| [Generate Parentheses](../Patterns/Backtracking.md) | O(4^n / √n) | O(n) | Catalan(n) leaves × O(n) each |
| [N-Queens](../Patterns/Backtracking.md) | O(n!) pruned | O(n) | At row k: k choices pruned |
| [Sudoku Solver](../Patterns/Backtracking.md) | O(9^(empty cells)) | O(1) extra | Upper bound; pruning helps greatly |
| [Word Search](../Patterns/Backtracking.md) | O(m × n × 4^L) | O(L) | L = word length |
| [Merge Sort](../Patterns/Divide%20and%20Conquer.md) | O(n log n) | O(n) aux + O(log n) stack | |
| [Quick Sort avg](../Patterns/Divide%20and%20Conquer.md) | O(n log n) | O(log n) | Random pivot |
| [Quick Sort worst](../Patterns/Divide%20and%20Conquer.md) | O(n²) | O(n) | Sorted input + bad pivot |
| [Palindrome Partitioning](../Patterns/Backtracking.md) | O(n × 2^n) | O(n) | At each pos: split or not |
| [Regex Match (memoized)](../Patterns/Divide%20and%20Conquer.md) | O(m × n) | O(m × n) | m=pattern, n=string |

---

## Master Theorem Reference

For `T(n) = aT(n/b) + f(n)`:

| Case | Condition | Result |
|------|-----------|--------|
| 1 | f(n) = O(n^(log_b a - ε)) | T(n) = Θ(n^log_b(a)) |
| 2 | f(n) = Θ(n^log_b(a)) | T(n) = Θ(n^log_b(a) × log n) |
| 3 | f(n) = Ω(n^(log_b a + ε)) and regularity | T(n) = Θ(f(n)) |

Common applications:
- Merge Sort: a=2, b=2, f(n)=O(n) → Case 2 → O(n log n)
- Binary Search: a=1, b=2, f(n)=O(1) → Case 2 → O(log n)
- Pow(x,n): a=1, b=2, f(n)=O(1) → Case 2 → O(log n)

---

## How to Explain Backtracking Complexity

**N-Queens:**
> "At row 0, we have n choices. At row 1, at most n-1 (one column blocked), but also diagonals reduce it further. In the worst case, we explore O(n!) branches. Each branch requires O(n) work at each row. So the upper bound is O(n × n!), but in practice pruning reduces this significantly."

**Generate Parentheses:**
> "The number of valid strings is the nth Catalan number: `C(n) = C(2n, n) / (n+1)`. For large n, this grows as `4^n / (n^(3/2) × √π)`. Building each string takes O(n). So total time is O(4^n / √n)."

**Subsets:**
> "There are exactly 2^n subsets. For each, we create a copy of size up to n. So total time is O(n × 2^n), space is O(n) for the stack."

---

## Space: Stack vs Aux

| Component | What It Is | When It Matters |
|-----------|-----------|----------------|
| Call stack | One frame per level of recursion | Always O(depth) |
| `path` / `used[]` | Current partial solution | O(n) for most |
| Results list | All solutions | O(solutions × avg size) |
| Aux array (merge sort) | Temp buffer | O(n) for merge sort |
| Memo table | Cached subproblem results | O(states) |

Results space is often **not** counted in space complexity (output doesn't count). Check what the interviewer expects.

---

## Memoization Converts Exponential to Polynomial

| Problem | Without Memo | With Memo |
|---------|-------------|-----------|
| Fibonacci | O(2^n) | O(n) |
| Regex Match | O(2^(m+n)) | O(m × n) |
| Different Ways (LC 241) | O(n × 4^n) | O(n^3) |
| Word Break II | O(2^n) | O(n^2 × output) |

**Rule:** If you see overlapping subproblems in a recursion tree → memoize → this becomes top-down DP.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Recursion Tree Visualization](./Recursion%20Tree%20Visualization.md)

> **Last Updated:** 2026-06-26
