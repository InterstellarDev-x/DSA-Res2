# Google Interview Problems — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Maximum XOR of Two Numbers | L4–L5 | Medium | ⭐⭐⭐⭐⭐ | [Bit Search / Trie](../Patterns/Bit%20Search%20and%20Trie.md) | [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |
| 2 | Single Number III | L4 | Medium | ⭐⭐⭐⭐ | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [260](https://leetcode.com/problems/single-number-iii/) |
| 3 | Minimum XOR Sum of Two Arrays | L5 | Hard | ⭐⭐⭐ | [Bitmask DP](../Patterns/Bitmask%20DP.md) | [1879](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/) |
| 4 | Total Hamming Distance | L4 | Hard | ⭐⭐⭐ | [Counting Bits](../Patterns/Counting%20Bits.md) | [477](https://leetcode.com/problems/total-hamming-distance/) |

---

## Deep Dive: Maximum XOR — Two Approaches

### Approach 1: Binary Trie (O(n) time, O(n) space)

```rust
fn find_maximum_xor(nums: &[i32]) -> i32 {
    let mut root = TrieNode::new();
    for &n in nums { insert(&mut root, n); }
    let mut max_val = 0;
    for &n in nums { max_val = max_val.max(max_xor(&root, n)); }
    max_val
}
```

### Approach 2: Greedy with `HashSet` (O(n) time, O(n) space, simpler)

```rust
use std::collections::HashSet;

fn find_maximum_xor(nums: &[i32]) -> i32 {
    let mut max_xor = 0i32;
    let mut mask = 0i32;
    for bit in (0..32).rev() {
        mask |= 1 << bit;
        let prefixes: HashSet<i32> = nums.iter().map(|&n| n & mask).collect();

        let candidate = max_xor | (1 << bit);
        for &p in &prefixes {
            if prefixes.contains(&(candidate ^ p)) { max_xor = candidate; break; }
        }
    }
    max_xor
}
```

**Why greedy works:** At each bit position, we greedily try to set that bit in the answer to 1. If two prefixes XOR to have that bit set, it's achievable.

### Google Follow-ups

**Q: How would you handle this for a stream of numbers (online)?**
The Trie approach naturally extends — insert each new number, query against all previously inserted numbers. Amortized O(32) per operation.

**Q: XOR queries with range constraint [min_val, max_val] on the second number?**
Sort + offline processing. See [Bit Search & Trie](../Patterns/Bit%20Search%20and%20Trie.md) for LC 1707.

---

## Deep Dive: Total Hamming Distance

### Key Insight

Pairwise hamming distance = for each bit position, count pairs that differ.
If `k` numbers have bit `i` set: those `k` pair with the `n-k` that don't → `k × (n-k)` differing pairs.

```rust
fn total_hamming_distance(nums: &[i32]) -> i32 {
    let mut total = 0i32;
    let n = nums.len() as i32;
    for bit in 0..32 {
        let ones: i32 = nums.iter().map(|&num| (num >> bit) & 1).sum();
        total += ones * (n - ones);
    }
    total
}
```

**Google follow-up:** "What if the array can have 10^5 elements and we need to support point updates?"
→ Maintain a 32-element array `bitCount[i]` = number of elements with bit `i` set. On update, adjust `bitCount` and recompute contribution.

---

## Related Files

- [Google OA-Qns](../OA-Qns/Google.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
