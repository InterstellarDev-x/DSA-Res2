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

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximumXOR(vector<int>& nums) {
    TrieNode* root = new TrieNode();
    for (int n : nums) insert(root, n);
    int maxVal = 0;
    for (int n : nums) maxVal = max(maxVal, maxXOR(root, n));
    return maxVal;
}
```

### Approach 2: Greedy with `std::unordered_set` (O(n) time, O(n) space, simpler)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximumXOR(vector<int>& nums) {
    int maxXor = 0, mask = 0;
    for (int bit = 31; bit >= 0; bit--) {
        mask |= (1 << bit);
        unordered_set<int> prefixes;
        for (int n : nums) prefixes.insert(n & mask);

        int candidate = maxXor | (1 << bit);
        for (int p : prefixes) {
            if (prefixes.count(candidate ^ p)) { maxXor = candidate; break; }
        }
    }
    return maxXor;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

int totalHammingDistance(vector<int>& nums) {
    int total = 0, n = nums.size();
    for (int bit = 0; bit < 32; bit++) {
        int ones = 0;
        for (int num : nums) ones += (num >> bit) & 1;
        total += ones * (n - ones);
    }
    return total;
}
```

**Google follow-up:** "What if the array can have 10^5 elements and we need to support point updates?"
→ Maintain a 32-element array `bitCount[i]` = number of elements with bit `i` set. On update, adjust `bitCount` and recompute contribution.

---

## Related Files

- [Google OA-Qns](../OA-Qns/Google.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
