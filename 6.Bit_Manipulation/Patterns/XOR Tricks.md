# XOR Tricks

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Single Number I/II/III, Missing Number, XOR Subarray Queries

---

## XOR Properties (Memorize These)

| Property | Expression | Meaning |
|----------|-----------|---------|
| Self-inverse | `a ^ a = 0` | Any number XORed with itself = 0 |
| Identity | `a ^ 0 = a` | XOR with 0 = no change |
| Commutative | `a ^ b = b ^ a` | Order doesn't matter |
| Associative | `(a^b)^c = a^(b^c)` | Grouping doesn't matter |
| Cancellation | `a ^ b ^ a = b` | a's cancel |
| Swap | `a ^= b; b ^= a; a ^= b` | Swaps without temp variable |

**Core use:** XOR all elements in a set — duplicates cancel (→ 0), unique element survives.

---

## Template 1 — Single Number (LC 136)

Every number appears twice except one. Find the unique one.

```cpp
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (auto n : nums) result ^= n;
    return result; // all pairs cancel; unique remains
}
```

**Time:** O(n). **Space:** O(1). No `std::unordered_set` needed.

---

## Template 2 — Missing Number (LC 268)

Array of n numbers in range [0, n], one missing. XOR all indices 0..n with all values:

```cpp
int missingNumber(vector<int>& nums) {
    int result = nums.size(); // start with n
    for (int i = 0; i < (int)nums.size(); i++) {
        result ^= i ^ nums[i]; // XOR index and value cancel if equal; missing value survives
    }
    return result;
}
```

**Alternative:** `sum(0..n) - sum(nums)` = `n*(n+1)/2 - accumulate(nums.begin(), nums.end(), 0)`. Both O(n) O(1).

---

## Template 3 — Single Number II (LC 137)

Every number appears **three times** except one. Cannot use XOR directly (triples don't cancel).

**Bit counting approach:** Count bits mod 3. Any bit that appears 3k times cancels; the remaining bit belongs to the unique number.

```cpp
int singleNumber(vector<int>& nums) {
    int ones = 0, twos = 0;
    for (auto n : nums) {
        ones = (ones ^ n) & ~twos; // bits seen once (not in twos)
        twos = (twos ^ n) & ~ones; // bits seen twice (not in ones)
        // bits seen three times fall out of both
    }
    return ones;
}
```

**State machine:** Each bit cycles through states 0 → 1 → 2 → 0. `ones` tracks bits at state 1, `twos` at state 2.

**Bit-by-bit approach (clearer but O(32)):**
```cpp
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int bit = 0; bit < 32; bit++) {
        int sum = 0;
        for (auto n : nums) sum += (n >> bit) & 1;
        result |= (sum % 3) << bit;
    }
    return result;
}
```

---

## Template 4 — Single Number III (LC 260)

Two numbers appear once; all others appear twice. Find both.

**Key insight:** `xorVal = a ^ b` (the XOR of the two unique numbers). Any set bit in `xorVal` is a bit where `a` and `b` differ — use it to partition the array.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> singleNumber(vector<int>& nums) {
    int xorVal = 0;
    for (auto n : nums) xorVal ^= n;

    // Find any differing bit (e.g., lowest set bit)
    int diff = xorVal & (-xorVal); // isolate lowest set bit

    int a = 0;
    for (auto n : nums) {
        if ((n & diff) != 0) a ^= n; // group 1: numbers with this bit set
    }
    return {a, xorVal ^ a}; // b = xorVal ^ a
}
```

**Why partition works:** `diff` bit is 1 in `a` and 0 in `b` (or vice versa). So they end up in different groups. Duplicates still cancel within each group. Each group's XOR reveals one unique number.

---

## Template 5 — XOR Queries of a Subarray (LC 1310)

Prefix XOR — analogous to prefix sum but with XOR.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
    int n = arr.size();
    vector<int> prefix(n + 1); // prefix[i] = XOR of arr[0..i-1]
    for (int i = 0; i < n; i++) prefix[i + 1] = prefix[i] ^ arr[i];

    vector<int> result(queries.size());
    for (int i = 0; i < (int)queries.size(); i++) {
        int l = queries[i][0], r = queries[i][1];
        result[i] = prefix[r + 1] ^ prefix[l]; // XOR[l..r] = prefix[r+1] ^ prefix[l]
    }
    return result;
}
```

**Why prefix XOR works:** `prefix[r+1] ^ prefix[l]` = XOR of all elements in `arr[0..r]` XOR'd with XOR of `arr[0..l-1]`. The first `l` elements cancel (each appears twice), leaving `arr[l..r]`.

---

## Template 6 — Find XOR of Numbers from L to R

`XOR(L, R) = XOR(0, L-1) ^ XOR(0, R)`

XOR from 0 to n has a 4-cycle:
```cpp
int xorUpTo(int n) {
    switch (n % 4) {
        case 0: return n;
        case 1: return 1;
        case 2: return n + 1;
        case 3: return 0;
        default: return -1; // unreachable
    }
}
int xorRange(int l, int r) { return xorUpTo(r) ^ xorUpTo(l - 1); }
```

---

## Common XOR Patterns Summary

| Problem Type | Technique |
|-------------|-----------|
| One duplicate, rest appear twice | `XOR all = unique` |
| One missing in [0..n] | `XOR all indices ^ all values` |
| Every element 3×, one 1× | State machine (`ones`, `twos`) or bit sum mod 3 |
| Two unique, rest 2× | `XOR all` → partition by any set bit |
| Range XOR query | Prefix XOR array |
| XOR of [1..n] | 4-cycle trick |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Single Number I | O(n) | O(1) |
| Single Number II (state machine) | O(n) | O(1) |
| Single Number III | O(n) | O(1) |
| Missing Number | O(n) | O(1) |
| XOR Queries | O(n + q) | O(n) |
| XOR(L, R) | O(1) | O(1) |

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — AND/OR/shift foundations
- [Counting Bits](./Counting%20Bits.md) — popcount and bit counting
- [Prefix Sum](../../1.Arrays/Patterns/Prefix%20Sum.md) — prefix XOR is analogous

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [Bit Basics](./Bit%20Basics.md) | **Next:** [Counting Bits](./Counting%20Bits.md)

> **Last Updated:** 2026-06-26
