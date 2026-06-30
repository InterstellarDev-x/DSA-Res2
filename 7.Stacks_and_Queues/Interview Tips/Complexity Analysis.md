# Complexity Analysis — Stacks & Queues

> **Topic:** [Stacks & Queues](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Pattern-to-Complexity Quick Reference

| Problem / Operation | Time | Space | Notes |
|---------------------|------|-------|-------|
| Stack push/pop/peek | O(1) amortized | O(n) | ArrayDeque; O(1) amortized due to array resizing |
| [Valid Parentheses](../Patterns/Stack%20Basics.md) | O(n) | O(n) | Stack at most n/2 deep |
| [Backspace String Compare (stack)](../Patterns/Stack%20Basics.md) | O(n+m) | O(n+m) | Builds both processed strings |
| [Backspace String Compare (2ptr)](../Patterns/Stack%20Basics.md) | O(n+m) | O(1) | Two pointers from right |
| [Simplify Path](../Patterns/Stack%20Basics.md) | O(n) | O(n) | Stack of path parts |
| [NGE I (HashMap)](../Patterns/Monotonic%20Stack.md) | O(n+m) | O(n) | n = nums2 size, m = nums1 size |
| [Daily Temperatures](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Each index enters/exits stack once |
| [NGE II (circular)](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Loop 2n; push only n times |
| [Online Stock Span](../Patterns/Monotonic%20Stack.md) | O(1) amortized | O(n) | Cumulative span; total pops ≤ total pushes |
| [132 Pattern](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Right-to-left monotonic stack |
| [Asteroid Collision](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Each asteroid pushed/popped at most once |
| [Remove K Digits](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Each digit pushed/popped at most once |
| [Trapping Rain Water (mono stack)](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | |
| [Trapping Rain Water (2ptr)](../Patterns/Monotonic%20Stack.md) | O(n) | O(1) | Optimal |
| [Sum of Subarray Mins](../Patterns/Monotonic%20Stack.md) | O(n) | O(n) | Two monotonic stack passes |
| [Remove Duplicate Letters](../Patterns/Monotonic%20Stack.md) | O(n) | O(1) | O(26) stack = O(1) |
| [Largest Rectangle (mono stack)](../Patterns/Histogram%20and%20Rectangle.md) | O(n) | O(n) | Sentinel flush |
| [Maximal Rectangle](../Patterns/Histogram%20and%20Rectangle.md) | O(rows×cols) | O(cols) | LRH per row |
| [Sliding Window Max](../Patterns/Queue%20and%20Deque.md) | O(n) | O(k) | Monotonic deque; k = window size |
| [Implement Queue via Stacks](../Patterns/Queue%20and%20Deque.md) | O(1) amortized | O(n) | Each element moves in→out once |
| [Implement Stack via Queue](../Patterns/Queue%20and%20Deque.md) | O(n) push, O(1) pop | O(n) | Rotate on push |
| [Min Stack](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | O(1) all ops | O(n) | Pair or aux stack |
| [Circular Queue](../Design%20Data%20Structure%20Problems/Min%20Stack.md) | O(1) all ops | O(k) | k = capacity |
| [Max Frequency Stack](../Design%20Data%20Structure%20Problems/Maximum%20Frequency%20Stack.md) | O(1) all ops | O(n) | Freq map + group map |
| [Evaluate RPN](../Patterns/Stack%20for%20Expressions.md) | O(n) | O(n) | |
| [Decode String](../Patterns/Stack%20for%20Expressions.md) | O(n×max_repeat) | O(n) | String building |
| [Basic Calculator I/II](../Patterns/Stack%20for%20Expressions.md) | O(n) | O(n) | Stack of terms |
| [Score of Parentheses (stack)](../Patterns/Stack%20for%20Expressions.md) | O(n) | O(n) | |
| [Score of Parentheses (depth)](../Patterns/Stack%20for%20Expressions.md) | O(n) | O(1) | 2^depth contribution |

---

## Amortized Analysis — Key Examples

### Monotonic Stack is O(n) Not O(n²)

Common mistake: seeing the nested `while` loop and thinking O(n²).

**Proof:** Each index is pushed exactly once and popped at most once. Total push operations = n. Total pop operations ≤ n. Each iteration of the inner `while` loop corresponds to one pop. Therefore total inner loop iterations ≤ n across all outer iterations.

Total work = O(n pushes + n pops) = **O(n)**.

### Queue via Two Stacks is O(1) Amortized

Each element follows this lifecycle: push to inStack (1 op) → transfer to outStack (1 op) → pop from outStack (1 op) = 3 ops total. Regardless of when the transfer happens, across N operations the total work is O(N). Per-operation amortized = O(1).

### Online Stock Span is O(1) Amortized

The cumulative span trick means each day is pushed exactly once. Total pops across all calls ≤ total pushes = number of calls. So total work across N calls = O(N), giving O(1) amortized.

---

## How to Explain "Each Element Enters/Exits Once" in an Interview

> "The key invariant is that each element is pushed to the stack exactly once. In the inner `while` loop, we only pop elements — we never add new ones. So the inner loop can run at most as many times as the outer loop pushed elements. Total iterations of the inner loop, summed across all outer loop iterations, is bounded by n. Combined with the n outer iterations, total time is O(n)."

This argument works for: monotonic stack (NGE, Daily Temps, Histogram), Asteroid Collision, Remove K Digits, and Sliding Window Max.

---

## Space Complexity Notes

| Structure | Space | Why |
|-----------|-------|-----|
| Stack | O(n) worst case | All elements pushed before any pop |
| Monotonic Stack | O(n) worst case | Strictly increasing input → stack fills |
| Deque (sliding window) | O(k) | At most k indices in window |
| Two-stack Queue | O(n) | All elements in one of the two stacks |
| Min Stack (pair) | O(n) | One pair per element |
| Max Freq Stack | O(n) | Each element in freq map + at least one group stack |

---

## How to Explain Stack Complexity at FAANG

**For monotonic stack:** "The algorithm is O(n) because each element is pushed exactly once and popped at most once. The inner while loop's total work across the entire outer loop is O(n), not O(n) per iteration."

**For Sliding Window Max:** "The deque holds at most k indices. Each index enters and exits the deque at most once across all n iterations. Total work = O(n)."

**For Queue via Two Stacks:** "Push is O(1). Pop is O(n) in the worst case but O(1) amortized — each element crosses from inStack to outStack exactly once in its lifetime. For N operations, total work is O(N)."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Stack vs Queue Usage](./Stack%20vs%20Queue%20Usage.md)
