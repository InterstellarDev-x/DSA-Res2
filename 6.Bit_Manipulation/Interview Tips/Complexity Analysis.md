# Complexity Analysis — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Pattern-to-Complexity Quick Reference

| Problem / Operation | Time | Space | Notes |
|---------------------|------|-------|-------|
| Get/Set/Clear/Toggle bit | O(1) | O(1) | Single instruction |
| [Reverse Bits](../Patterns/Bit%20Basics.md) | O(32) = O(1) | O(1) | Fixed 32-bit input |
| [Power of Two](../Patterns/Bit%20Basics.md) | O(1) | O(1) | `n & (n-1)` |
| [Sum of Two Integers](../Patterns/Bit%20Basics.md) | O(32) = O(1) | O(1) | ≤32 carry propagation steps |
| [Divide Two Integers](../Patterns/Bit%20Basics.md) | O(log² n) | O(1) | log n outer × log n inner |
| [AND of Range](../Patterns/Bit%20Basics.md) | O(32) = O(1) | O(1) | Right-shift until equal |
| [Single Number I](../Patterns/XOR%20Tricks.md) | O(n) | O(1) | XOR all |
| [Single Number II](../Patterns/XOR%20Tricks.md) | O(n) | O(1) | State machine; or O(32n)=O(n) bit count |
| [Single Number III](../Patterns/XOR%20Tricks.md) | O(n) | O(1) | XOR all, partition |
| [Missing Number](../Patterns/XOR%20Tricks.md) | O(n) | O(1) | |
| [XOR Subarray Queries](../Patterns/XOR%20Tricks.md) | O(n + q) | O(n) | Prefix XOR build + queries |
| XOR(L, R) | O(1) | O(1) | 4-cycle formula |
| [Hamming Weight (Kernighan)](../Patterns/Counting%20Bits.md) | O(popcount) | O(1) | ≤32 iterations |
| `Integer.bitCount` | O(1) | O(1) | Hardware instruction (POPCNT) |
| [Counting Bits 0..n (DP)](../Patterns/Counting%20Bits.md) | O(n) | O(n) | DP recurrence |
| [Hamming Distance](../Patterns/Counting%20Bits.md) | O(1) | O(1) | `bitCount(x ^ y)` |
| [Total Hamming Distance](../Patterns/Counting%20Bits.md) | O(32n) = O(n) | O(1) | Per-bit contribution |
| [Subsets (bitmask)](../Patterns/Bitmask%20DP.md) | O(n × 2^n) | O(n × 2^n) | All subsets × copy |
| [Min XOR Sum (Bitmask DP)](../Patterns/Bitmask%20DP.md) | O(n² × 2^n) | O(2^n) | Assignment DP |
| [Shortest Path All Nodes](../Patterns/Bitmask%20DP.md) | O(n² × 2^n) | O(n × 2^n) | BFS with bitmask |
| [Max XOR Two Numbers (Trie)](../Patterns/Bit%20Search%20and%20Trie.md) | O(32n) = O(n) | O(32n) = O(n) | Insert + query each |
| [Max XOR (HashSet)](../Patterns/Bit%20Search%20and%20Trie.md) | O(32n) = O(n) | O(n) | Prefix sets per bit |

---

## Why Bit Operations Are "O(1)"

For 32-bit integers: all bit operations process a fixed 32 bits regardless of input value. Even loops over all 32 bits run exactly 32 iterations — constant time. For 64-bit (`long`), it's O(64) = O(1).

Only when iterating over n elements (`for (int n : nums)`) does the time become O(n).

---

## Bitmask DP Feasibility

| n | 2^n | Feasible? |
|---|-----|-----------|
| 10 | 1,024 | ✅ Easy |
| 15 | 32,768 | ✅ OK |
| 20 | 1,048,576 | ✅ Borderline (with O(n × 2^n)) |
| 25 | 33,554,432 | ⚠️ Tight — depends on constant |
| 30 | 1,073,741,824 | ❌ TLE |

Rule of thumb: **n ≤ 20** for bitmask DP, **n ≤ 25** for just enumeration.

---

## Submask Enumeration Is O(3^n)

Total iterations across all masks:

```
∑_{mask=0}^{2^n - 1} 2^|mask| = ∑_{k=0}^{n} C(n,k) × 2^k = (1+2)^n = 3^n
```

Binomial identity: `∑ C(n,k) × 2^k = 3^n`.

For n=20: 3^20 ≈ 3.5 billion — **too slow**. For n=15: 3^15 ≈ 14 million — **OK**.

---

## Space Complexity Traps

| Expression | Appears O(1) | Actually |
|------------|-------------|---------|
| `new int[1 << n]` | — | O(2^n) — exponential if n is large |
| `new boolean[n][1 << n]` | — | O(n × 2^n) |
| XOR prefix array | — | O(n) |
| Binary Trie (32 bits, n numbers) | — | O(32n) = O(n) nodes |

---

## How to Explain Bitmask DP Time

> "There are 2^n possible subsets. For each subset, we iterate over at most n elements to try adding one. So total transitions are n × 2^n. Each transition is O(1). Total time: O(n × 2^n)."

> For submask: "Every element can be in the outer mask, in the inner sub-mask, or in neither. By the inclusion-exclusion analogy, total iterations = 3^n — one factor of 3 per element."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Java Bit Operators](./Java%20Bit%20Operators.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
