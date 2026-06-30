# Complexity Analysis — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `complexity` `big-o` `time-space` `analysis`

---

## Table of Contents

1. [How to State Complexity](#how-to-state-complexity)
2. [Pattern-to-Complexity Quick Reference](#pattern-to-complexity-quick-reference)
3. [Amortized Complexity](#amortized-complexity)
4. [Common Complexity Traps](#common-complexity-traps)
5. [Space Complexity Nuances](#space-complexity-nuances)
6. [Related Files](#related-files)

---

## How to State Complexity

**Template to use in interviews:**

> "This solution has **O(n log n)** time complexity due to sorting, and **O(1)** additional space since I'm modifying the input array in-place. If the interviewer requires no mutation, it would be O(n) space for a copy."

Key points:
- Always state **both** time and space
- Explain the **dominant term** — not just the answer
- Mention **best case / worst case** if they differ significantly
- Address space for **output** separately from **auxiliary** space

---

## Pattern-to-Complexity Quick Reference

| Pattern | Time | Auxiliary Space | Notes |
|---------|------|----------------|-------|
| [Prefix Sum](../Patterns/Prefix%20Sum.md) — build | O(n) | O(n) | One-time cost |
| [Prefix Sum](../Patterns/Prefix%20Sum.md) — query | O(1) | — | After build |
| [Prefix Sum + HashMap](../Patterns/Prefix%20Sum.md) | O(n) | O(n) | Count subarrays |
| [Sliding Window](../Patterns/Sliding%20Window.md) — fixed | O(n) | O(1) | One pass |
| [Sliding Window](../Patterns/Sliding%20Window.md) — variable | O(n) | O(k) for freq map | Amortized O(n) |
| [Two Pointers](../Patterns/Two%20Pointers.md) — one array | O(n) | O(1) | After optional sort |
| [Two Pointers](../Patterns/Two%20Pointers.md) — 3Sum | O(n²) | O(1) | Outer loop × inner scan |
| [Kadane's Algorithm](../Patterns/Kadane's%20Algorithm.md) | O(n) | O(1) | Single pass |
| [Dutch National Flag](../Patterns/Dutch%20National%20Flag.md) | O(n) | O(1) | Single pass |
| [Moore's Voting](../Patterns/Moore's%20Voting.md) | O(n) | O(1) | Two passes |
| [Cyclic Sort](../Patterns/Cyclic%20Sort.md) | O(n) | O(1) | Amortized |
| [Merge Intervals](../Patterns/Merge%20Intervals.md) | O(n log n) | O(n) | Sort dominates |
| Sort + scan | O(n log n) | O(1) or O(log n) | Stack space for sort |
| HashSet/HashMap lookup | O(1) average | O(n) | Not O(1) worst case |

---

## Amortized Complexity

### Cyclic Sort — Why O(n)?

Each element is swapped **at most once** into its correct position.
Total swaps ≤ n, even though the while loop looks like O(n²).

```
Proof: i only stays at same position if a swap happened.
Each swap moves at least one element to its final position.
→ Total swaps ≤ n → O(n) total
```

### Sliding Window — Why O(n)?

`l` and `r` each move **at most n times** — they never go backwards.
Even though the inner `while` loop shrinks the window, total shrink operations = total expand operations ≤ n.

```
Total iterations = (times r advances) + (times l advances) ≤ 2n = O(n)
```

---

## Common Complexity Traps

| Trap | Wrong Answer | Correct Answer |
|------|-------------|----------------|
| "HashSet.contains is O(1)" | Always O(1) | O(1) **average**, O(n) worst case (hash collisions) |
| "Sorting is free" | — | O(n log n) — always mention if you sort |
| "Output space doesn't count" | O(1) space | O(n) auxiliary + O(n) output = O(n) total; state both |
| "Two nested for-loops = O(n²)" | Always O(n²) | Not if inner loop is amortized — e.g., sliding window is O(n) |
| "Recursion is O(1) space" | — | Recursion depth = O(log n) for divide-and-conquer, O(n) for linear recursion |
| "Arrays.sort is O(n log n)" | Always | For `int[]` it's dual-pivot quicksort: O(n log n) avg, O(n²) worst |

---

## Space Complexity Nuances

| Scenario | Space Analysis |
|----------|----------------|
| In-place algorithm | O(1) auxiliary (don't count input array) |
| Prefix sum array | O(n) auxiliary |
| HashMap for seen elements | O(n) in worst case |
| Output array (e.g., product except self) | O(n) — but interviewer may exclude output from "extra space" |
| Recursion stack | O(depth) — state explicitly |
| Result list (e.g., merge intervals) | O(n) — state even if "obvious" |

---

## Interview Complexity Script

When the interviewer asks "What's the complexity?":

1. **Time:** "This is O(n) — we do a single pass over the array with the sliding window. The inner while loop is amortized O(1) per element because `l` advances at most n times total."
2. **Space:** "This uses O(1) extra space — I'm only tracking a few integer variables. The output array is O(n) but that's required by the problem."
3. **Follow-up:** "Can we do better?" → Analyze if lower bounds apply (e.g., comparison-based sort can't be better than O(n log n)).

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Optimization Tips](./Optimization.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
