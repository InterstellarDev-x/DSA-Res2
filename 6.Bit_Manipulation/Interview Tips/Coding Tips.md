# Coding Tips — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## The Six Core Operations (Memorize)

```cpp
int getBit(int n, int k)    { return (n >> k) & 1; }
int setBit(int n, int k)    { return n | (1 << k); }
int clearBit(int n, int k)  { return n & ~(1 << k); }
int toggleBit(int n, int k) { return n ^ (1 << k); }
int lowestSetBit(int n)     { return n & (-n); }
int clearLowest(int n)      { return n & (n - 1); }
```

---

## Arithmetic vs Logical Right Shift in C++

```cpp
-1 >> 1                      // = -1  (arithmetic: sign-extends, fills with 1 — implementation-defined for signed in C++)
(unsigned)(-1) >> 1          // = INT_MAX (logical: fills with 0, cast to unsigned first)
```

**Rule:** Use `(unsigned)n >> k` when dealing with bit patterns that should not be sign-extended (e.g., Reverse Bits, hash functions, byte manipulation).

---

## 1L vs 1 for Bit Shifts

```cpp
int n = 32;
1 << n      // BUG: 1 << 32 is undefined behavior for shift >= width in C++
1 << 31     // UB: signed integer overflow for 32-bit int
1L << 31    // = 2147483648 (positive long)
1L << 32    // = 4294967296 (safe with long)
```

**Rule:** For shifts >= 31, use `1L` (long literal).

---

## Operator Precedence

Bitwise operators have lower precedence than arithmetic and comparison:

```cpp
// BUG: == has higher precedence than &
if (n & 1 == 0)   // parsed as: n & (1 == 0) = n & false = n & 0 = 0

// FIX: always parenthesize bitwise operations
if ((n & 1) == 0)
```

Common traps:
```cpp
n & n - 1    // = n & (n-1) // (subtraction before &)
n | 1 << k   // = n | (1 << k) // (shift before |)
~n + 1       // = (~n) + 1 // (NOT before +)
```

---

## INT_MIN / INT_MAX Invariants

```cpp
#include <bits/stdc++.h>
using namespace std;

INT_MIN                // = -2147483648 = -2^31 = 0x80000000
INT_MAX                // = 2147483647  = 2^31 - 1 = 0x7FFFFFFF
-INT_MIN               // OVERFLOW: still INT_MIN (undefined behavior for signed)
abs(INT_MIN)           // OVERFLOW: still INT_MIN
(long)INT_MIN          // = -2147483648L (safe)
```

**Rule:** Whenever negating or taking abs of a value that might be `INT_MIN`, cast to `long` first.

---

## XOR for In-Place Swap (No Temp)

```cpp
// Only works if a and b are DIFFERENT variables/addresses
a ^= b; // a = a^b
b ^= a; // b = b^(a^b) = a
a ^= b; // a = (a^b)^a = b
```

**Caution:** If `a` and `b` point to the same memory location (same index), this sets them both to 0.

---

## Bitmask Idioms

```cpp
int fullMask = (1 << n) - 1;  // all n bits set: 0b111...1 (n ones)
int highBit  = 1 << (n - 1);  // only highest bit set
int evenBits = 0xAAAAAAAA;     // bits 1,3,5,... (even positions from 0)
int oddBits  = 0x55555555;     // bits 0,2,4,... (odd positions from 0)
```

---

## Counting Set Bits — Three Ways

```cpp
#include <bits/stdc++.h>
using namespace std;

// 1. Brian Kernighan (fastest for sparse bits)
while (n != 0) { n &= (n-1); count++; }

// 2. Built-in (fastest overall)
__builtin_popcount(n);

// 3. Lookup table (for repeated calls, O(1) per call after O(256) build)
vector<int> lookup(256);
for (int i = 1; i < 256; i++) lookup[i] = (i & 1) + lookup[i >> 1];
// count bits in n: lookup[n&0xFF] + lookup[(n>>8)&0xFF] + ...
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [C++ Bit Operators](./Java%20Bit%20Operators.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
