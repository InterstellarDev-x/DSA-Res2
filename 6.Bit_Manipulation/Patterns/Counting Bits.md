# Counting Bits

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Hamming Weight, Counting Bits DP, Hamming Distance, Total Hamming Distance

---

## Brian Kernighan's Algorithm

`n & (n-1)` clears the **lowest set bit** of n. Iterate until n = 0, counting iterations = popcount.

```cpp
int hammingWeight(int n) {
    int count = 0;
    while (n != 0) {
        n &= (n - 1); // clear lowest set bit
        count++;
    }
    return count;
}
```

**Why it works:** `n-1` flips all bits from the lowest set bit downward. AND with `n` clears that bit and all below it. Each iteration eliminates exactly one set bit.

**Example: n = 12 (1100)**
- `12 & 11 = 1100 & 1011 = 1000` → cleared bit 2, count=1
- `8 & 7 = 1000 & 0111 = 0000` → cleared bit 3, count=2
- Done. Result = 2 ✅

---

## C++ Built-in popcount

```cpp
__builtin_popcount(n);              // number of 1-bits in int n
__builtin_popcountll(n);            // for long long
__builtin_clz(n);                   // leading zeros
__builtin_ctz(n);                   // trailing zeros = position of lowest set bit
(1 << (31 - __builtin_clz(n)));     // keeps only the highest set bit
(n & (-n));                          // keeps only the lowest set bit
```

---

## Template 1 — Counting Bits for 0..n (LC 338)

Return array where `result[i]` = number of 1 bits in i, for all i in [0, n].

**Naive:** O(n log n) — call `hammingWeight` for each.

**DP recurrence: O(n) time, O(n) space**

Key insight: `bits(n) = bits(n >> 1) + (n & 1)` — right-shift removes the last bit; `n & 1` adds it back if it was 1.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> countBits(int n) {
    vector<int> dp(n + 1, 0);
    for (int i = 1; i <= n; i++) {
        dp[i] = dp[i >> 1] + (i & 1);
    }
    return dp;
}
```

**Alternative DP using `n & (n-1)`:**
```cpp
dp[i] = dp[i & (i - 1)] + 1; // i has one more bit than i-with-lowest-cleared
```

---

## Template 2 — Hamming Distance (LC 461)

Number of bit positions where two integers differ = popcount of their XOR.

```cpp
int hammingDistance(int x, int y) {
    return __builtin_popcount(x ^ y);
}
```

---

## Template 3 — Total Hamming Distance (LC 477)

Sum of hamming distances between all pairs in an array. Naive O(n²) is too slow.

**Key insight:** Consider each bit position independently. If `k` numbers have bit `i` set and `n-k` do not, that bit contributes `k × (n-k)` to the total hamming distance.

```cpp
#include <bits/stdc++.h>
using namespace std;

int totalHammingDistance(vector<int>& nums) {
    int total = 0, n = nums.size();
    for (int bit = 0; bit < 32; bit++) {
        int ones = 0;
        for (auto& num : nums) ones += (num >> bit) & 1;
        total += ones * (n - ones); // pairs where this bit differs
    }
    return total;
}
```

**Time:** O(32n) = O(n). **Space:** O(1).

---

## Template 4 — Number of Steps to Reduce to Zero (LC 1342)

At each step: if even, divide by 2 (right shift); if odd, subtract 1 (clear last bit).

```cpp
int numberOfSteps(int num) {
    return __builtin_popcount(num) + (32 - __builtin_clz(num)) - 1;
    // = (number of 1-bits) + (bit-length - 1)
    // = ones take 1 step each to subtract; zeros take 1 step each to shift
}
```

**Proof:** Each 1-bit contributes 2 steps (subtract then shift), except the highest bit contributes 1 (only subtract). Each 0-bit contributes 1 step (just shift). Total = 2×ones + zeros - 1 = ones + bitLength - 1.

---

## Counting Bits DP Table (first 16 numbers)

| n | binary | popcount | dp[n>>1] + (n&1) |
|---|--------|----------|-----------------|
| 0 | 0000 | 0 | — |
| 1 | 0001 | 1 | dp[0]+1 = 1 |
| 2 | 0010 | 1 | dp[1]+0 = 1 |
| 3 | 0011 | 2 | dp[1]+1 = 2 |
| 4 | 0100 | 1 | dp[2]+0 = 1 |
| 5 | 0101 | 2 | dp[2]+1 = 2 |
| 6 | 0110 | 2 | dp[3]+0 = 2 |
| 7 | 0111 | 3 | dp[3]+1 = 3 |
| 8 | 1000 | 1 | dp[4]+0 = 1 |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Hamming Weight (Kernighan) | O(popcount) | O(1) |
| Hamming Weight (C++ built-in) | O(1) | O(1) |
| Counting Bits 0..n (DP) | O(n) | O(n) |
| Hamming Distance | O(1) | O(1) |
| Total Hamming Distance | O(32n) = O(n) | O(1) |

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — `n & (n-1)` foundation
- [XOR Tricks](./XOR%20Tricks.md) — popcount of XOR = hamming distance
- [Bitmask DP](./Bitmask%20DP.md) — iterate subsets using bit operations

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [XOR Tricks](./XOR%20Tricks.md) | **Next:** [Bitmask DP](./Bitmask%20DP.md)

> **Last Updated:** 2026-06-26
