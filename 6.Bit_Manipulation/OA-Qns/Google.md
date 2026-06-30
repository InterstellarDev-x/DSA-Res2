# Google OA — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** OA-Qns
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Difficulty | Frequency | Year | Pattern | LeetCode |
|---|---------|-----------|----------|------|---------|---------|
| 1 | Maximum XOR of Two Numbers | Medium | ⭐⭐⭐⭐⭐ | 2023–2026 | [Bit Search / Trie](../Patterns/Bit%20Search%20and%20Trie.md) | [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |
| 2 | Single Number III | Medium | ⭐⭐⭐⭐ | 2024–2025 | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [260](https://leetcode.com/problems/single-number-iii/) |
| 3 | Bitwise AND of Range | Medium | ⭐⭐⭐ | 2023–2025 | [Bit Basics](../Patterns/Bit%20Basics.md) | [201](https://leetcode.com/problems/bitwise-and-of-numbers-range/) |
| 4 | Minimum XOR Sum of Two Arrays | Hard | ⭐⭐⭐ | 2024–2025 | [Bitmask DP](../Patterns/Bitmask%20DP.md) | [1879](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) |
| 5 | Shortest Path Visiting All Nodes | Hard | ⭐⭐ | 2024 | [Bitmask DP](../Patterns/Bitmask%20DP.md) | [847](https://leetcode.com/problems/shortest-path-visiting-all-nodes/) |

---

## Google Focus

Google emphasizes **insight over brute force** for bit manipulation:
- Max XOR: explain why greedy MSB approach works, not just the Trie code
- Single Number III: explain the partition logic — why any set bit in `xor = a^b` works as a separator
- Bitmask DP: justify state space O(n × 2^n) and why bitmask represents visited set compactly

---

## Related Files

- [Google Interview Problems](../Interview%20Problems/Google.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
