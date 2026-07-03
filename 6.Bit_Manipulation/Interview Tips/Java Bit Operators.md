# C++ Bit Operators Reference

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Integer / Long Utility Methods

| Method | Returns | Example |
|--------|---------|---------|
| `__builtin_popcount(n)` | Number of 1-bits | `__builtin_popcount(7)` = 3 |
| `1u << (31 - __builtin_clz(n))` | `n` with only MSB kept | result for 12 = 8 |
| `n & (-n)` | `n` with only LSB kept | result for 12 = 4 |
| `__builtin_clz(n)` | Leading zeros count | `__builtin_clz(8)` = 28 |
| `__builtin_ctz(n)` | Trailing zeros count | `__builtin_ctz(8)` = 3 |
| (no standard equivalent) | Bits reversed | use manual loop or `__builtin_bitreverse32` (non-standard) |
| `__builtin_bswap32(n)` | Bytes reversed | byte-order swap |
| `bitset<32>(n).to_string()` | Binary string, no padding | `"1010"` for 10 |
| `printf("%x", n)` / `stringstream` with `hex` | Hex string | `"a"` for 10 |
| `(n > 0) - (n < 0)` | -1, 0, or 1 | sign of n |
| `INT_MAX` | 2^31 - 1 = 2147483647 | |
| `INT_MIN` | -2^31 = -2147483648 | |
| `__builtin_popcountll(n)` | Number of 1-bits for `long long` | |

---

## Bitwise Assignment Operators

```cpp
n &= mask;             // n = n & mask
n |= mask;             // n = n | mask
n ^= mask;             // n = n ^ mask
n <<= k;               // n = n << k
n >>= k;               // n = n >> k  (arithmetic, sign-extends for signed types)
n = (unsigned)n >> k;  // logical right shift (no >>>= in C++; cast to unsigned first)
```

---

## Bitmask Constants

```cpp
0xFF         // = 255, lowest 8 bits set
0xFFFF       // = 65535, lowest 16 bits
0xFFFFFFFF   // = -1 as int (all 32 bits set)
0xFFFFFFFFUL // = 4294967295UL as unsigned long (unsigned 32-bit max)
0x80000000   // = INT_MIN (only bit 31 set)
0x7FFFFFFF   // = INT_MAX (bits 0-30 set)
0xAAAAAAAA   // = bits 1,3,5,... (even positions from 0) = 2863311530
0x55555555   // = bits 0,2,4,... (odd positions from 0) = 1431655765
```

---

## Shift Rules

| Shift | Symbol | Effect | Sign-extends? |
|-------|--------|--------|--------------|
| Left | `<<` | × 2^k | N/A — zeros fill right |
| Arithmetic right | `>>` | ÷ 2^k (floor) | Yes — sign bit fills left (for signed types) |
| Logical right | `(unsigned)n >> k` | ÷ 2^k (unsigned) | No — zeros fill left |

**Shift by ≥ 32 (int) or ≥ 64 (long long):** In C++, shifting by >= the bit width is **undefined behavior**. Always ensure `0 <= k < 32` (or 64 for `long long`). Unlike Java, C++ does NOT take `k % width` automatically.

---

## Two's Complement Quick Conversions

```cpp
-n      = ~n + 1       // negate
~n      = -n - 1       // bitwise NOT
n & -n  = lowest set bit of n
n | -n  = all bits set from lowest set bit upward
```

---

## Checking Individual Bits

```cpp
// Is bit k set in n?
bool isSet = ((n >> k) & 1) == 1;
// OR equivalently:
bool isSet = (n & (1 << k)) != 0;
```

---

## C++ `std::bitset`

```cpp
#include <bits/stdc++.h>
using namespace std;

bitset<100> bs;          // 100-bit set, all 0 (size must be compile-time constant)
bs.set(5);               // set bit 5
bs.reset(5);             // clear bit 5
bs.flip(5);              // toggle bit 5
bs.test(5);              // true if bit 5 is set
bs.count();              // number of set bits
bs &= other;             // AND in place
bs |= other;             // OR in place
bs ^= other;             // XOR in place
bs._Find_next(from);     // next set bit > from (GCC extension)
bs.size();               // fixed size (100 in this case)
bs._Find_first();        // index of first set bit (GCC extension)
```

**Note:** `std::bitset<N>` requires a compile-time constant size and is fixed-width. For dynamic bit sets, use `vector<bool>` or a manual bitmask with `vector<uint64_t>`.

---

## Precedence Order (High → Low)

```
~  (bitwise NOT)
<< >>  (shifts)
&  (bitwise AND)
^  (bitwise XOR)
|  (bitwise OR)
```

Comparison (`==`, `!=`, `<`, etc.) has **higher** precedence than `&`, `^`, `|`. Always parenthesize.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
