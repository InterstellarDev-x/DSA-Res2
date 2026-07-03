# Bit Basics

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Reverse Bits, Power of Two, Sum without +, Divide without /, AND of Range

---

## C++ Bitwise Operators

| Operator | Symbol | Example | Result |
|----------|--------|---------|--------|
| AND | `&` | `5 & 3` = `101 & 011` | `001` = 1 |
| OR | `\|` | `5 \| 3` = `101 \| 011` | `111` = 7 |
| XOR | `^` | `5 ^ 3` = `101 ^ 011` | `110` = 6 |
| NOT | `~` | `~5` = `~00000101` | `11111010` = -6 |
| Left Shift | `<<` | `1 << 3` | `1000` = 8 |
| Arithmetic Right | `>>` | `-8 >> 1` | `-4` (sign-extends) |
| Logical Right | `(unsigned)n >> k` | `(unsigned)-8 >> 1` | `2147483644` (fills with 0) |

**Note:** `~n = -(n+1)` in two's complement. `~0 = -1`, `~(-1) = 0`.

---

## Core Operations

```cpp
#include <bits/stdc++.h>
using namespace std;

int getBit(int n, int k)    { return (n >> k) & 1; }
int setBit(int n, int k)    { return n | (1 << k); }
int clearBit(int n, int k)  { return n & ~(1 << k); }
int toggleBit(int n, int k) { return n ^ (1 << k); }
bool isPowerOf2(int n)      { return n > 0 && (n & (n - 1)) == 0; }
int lowestSetBit(int n)     { return n & (-n); }         // isolate
int clearLowest(int n)      { return n & (n - 1); }       // clear lowest
```

---

## Template 1 — Reverse Bits (LC 190)

Process 32 bits one at a time: extract LSB from `n`, put into MSB of result.

```cpp
#include <bits/stdc++.h>
using namespace std;

int reverseBits(int n) {
    int result = 0;
    for (int i = 0; i < 32; i++) {
        result = (result << 1) | (n & 1); // shift result left, put current LSB
        n = (int)((unsigned int)n >> 1); // logical shift (unsigned) — fill with 0
    }
    return result;
}
```

**Note:** Must use `(unsigned int)n >> 1` (logical), not `n >> 1` (arithmetic). For negative `n`, arithmetic `>>` fills with 1s and gives wrong answer.

---

## Template 2 — Power of Two (LC 231)

A power of 2 has exactly one set bit. `n & (n-1)` clears the lowest set bit.

```cpp
bool isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

**Generalize:** Power of 4 has its single bit at an even position: `n > 0 && (n & (n-1)) == 0 && (n & 0xAAAAAAAA) == 0` (0xAAAAAAAA = all even-position bits set).

---

## Template 3 — Sum of Two Integers (no + or −) (LC 371)

Simulate binary addition with carry:

```cpp
int getSum(int a, int b) {
    while (b != 0) {
        int carry = (a & b) << 1; // carry: positions where both bits are 1
        a = a ^ b;                 // sum without carry
        b = carry;
    }
    return a;
}
```

**Loop invariant:** At each iteration, `a` holds the partial sum (no carry), `b` holds the carry to be added. When `b == 0`, no more carries — `a` is the answer.

**Note:** In C++, `int` is typically 32-bit on modern platforms. This terminates. In Python, integers are unbounded — need `& 0xFFFFFFFF` masking.

---

## Template 4 — Divide Two Integers (LC 29)

Use bit shifts to find the largest multiple of divisor ≤ dividend:

```cpp
#include <bits/stdc++.h>
using namespace std;

int divide(int dividend, int divisor) {
    if (dividend == INT_MIN && divisor == -1) return INT_MAX; // overflow

    long dvd = abs((long) dividend);
    long dvs = abs((long) divisor);
    int sign = (dividend > 0) == (divisor > 0) ? 1 : -1;
    long result = 0;

    while (dvd >= dvs) {
        long tmp = dvs, multiple = 1;
        while (dvd >= (tmp << 1)) {
            tmp <<= 1;
            multiple <<= 1;
        }
        dvd -= tmp;
        result += multiple;
    }
    return (int) (sign * result);
}
```

**Key:** `tmp << 1` doubles the divisor — find the largest `2^k × divisor ≤ dividend` in O(log² n).

---

## Template 5 — Bitwise AND of Numbers Range (LC 201)

All numbers in [m, n] AND'd together. Any differing bit in the range will have both 0 and 1 → AND = 0.

The result is the **common prefix** of m and n in binary.

```cpp
int rangeBitwiseAnd(int m, int n) {
    int shift = 0;
    while (m != n) {
        m >>= 1;
        n >>= 1;
        shift++;
    }
    return m << shift; // common prefix shifted back
}
```

**Alternative — Brian Kernighan:** Keep clearing lowest set bit of `n` until `n <= m`.
```cpp
while (n > m) n &= (n - 1);
return n;
```

---

## Template 6 — Concatenation of Consecutive Binary Numbers (LC 1680)

Numbers 1, 2, ..., n concatenated in binary. Result mod 10^9+7.

```cpp
#include <bits/stdc++.h>
using namespace std;

int concatenatedBinary(int n) {
    long result = 0;
    const int MOD = (int)1e9 + 7;
    for (int i = 1; i <= n; i++) {
        int bits = (int)(log(i) / log(2)) + 1; // bit length of i
        // OR: bits = 32 - __builtin_clz(i)
        result = ((result << bits) | i) % MOD;
    }
    return (int) result;
}
```

---

## Shift Operator Precedence Trap

```cpp
// BUG: + has higher precedence than <<
1 << n + 1  // = 1 << (n+1), not (1 << n) + 1

// SAFE: always parenthesize shifts
(1 << n) + 1
```

---

## Two's Complement Reminders

- `-n = ~n + 1`
- `n & (-n)` = lowest set bit (from two's complement)
- `INT_MIN = -2^31` has no positive counterpart in `int` → use `long`
- `~INT_MIN = INT_MAX`

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Get/Set/Clear/Toggle bit | O(1) | O(1) |
| Reverse Bits | O(32) = O(1) | O(1) |
| Power of Two | O(1) | O(1) |
| Sum (getSum) | O(32) = O(1) | O(1) |
| Divide (bit shift) | O(log² n) | O(1) |
| AND of Range | O(32) = O(1) | O(1) |

---

## Related Patterns

- [XOR Tricks](./XOR%20Tricks.md) — XOR-specific identities
- [Counting Bits](./Counting%20Bits.md) — Brian Kernighan popcount
- [Bitmask DP](./Bitmask%20DP.md) — subsets as integers

---

**Back:** [Bit Manipulation README](../README.md) | **Next:** [XOR Tricks](./XOR%20Tricks.md)

> **Last Updated:** 2026-06-26
