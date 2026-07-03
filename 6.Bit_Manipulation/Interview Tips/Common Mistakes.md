# Common Mistakes — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Operator Precedence: Missing Parentheses

```cpp
// BUG: == binds tighter than &
if (n & 1 == 0) // parsed as n & (1 == 0) = n & 0 = always 0

// FIX
if ((n & 1) == 0)
```

---

## 2. Arithmetic vs Logical Right Shift

```cpp
// BUG: >> sign-extends; for negative n, fills with 1s
int n = -8; // binary: 11111000
n >> 1; // = -4 (11111100) — fills with 1

// FIX: use (unsigned) >> for unsigned (fill-with-0) shift
(unsigned)n >> 1; // = 2147483644 (01111100)
```

---

## 3. INT_MIN Negation Overflow

```cpp
#include <bits/stdc++.h>
using namespace std;

// BUG
int n = INT_MIN;
int pos = -n; // still INT_MIN! overflow

// FIX: cast to long long
long long pos = -(long long) n; // = 2147483648LL
```

---

## 4. `1 << k` vs `1LL << k` for k >= 31

```cpp
// BUG: 1 is int; 1 << 31 = INT_MIN (negative!)
long long mask = 1 << 31; // = -2147483648LL (promoted after shift)

// FIX
long long mask = 1LL << 31; // = 2147483648LL
```

---

## 5. XOR Swap When a == b (Same Reference)

```cpp
// BUG: if i == j, both become 0
nums[i] ^= nums[j];
nums[j] ^= nums[i];
nums[i] ^= nums[j];
// if i == j: nums[i] = 0

// FIX: guard first
if (i != j) { ... swap ... }
```

---

## 6. `n & (n-1)` vs `n & (~n+1)` Confusion

```cpp
n & (n - 1)  // clears lowest set bit of n
n & (-n)     // isolates lowest set bit of n (since -n = ~n+1)
```

Don't mix these up — one removes the bit, the other keeps only that bit.

---

## 7. Single Number II — Wrong State Machine Ordering

```cpp
// BUG: updating ones before twos with the same n causes interference
ones = (ones ^ n) & ~twos;
twos = (twos ^ n) & ~ones; // ones already updated — uses new ones (actually correct order)
```

The correct order is: update `ones` first (using old `twos`), then `twos` (using new `ones`). This IS the correct order above. Reversing it breaks the state machine.

---

## 8. Bitmask Overflow for n >= 32

```cpp
// BUG: 1 << 32 in C++ = undefined behavior (shift >= width of int)
int fullMask = (1 << 32) - 1; // = undefined / 0 — WRONG for n=32

// FIX: use -1 for all-bits mask
int fullMask = -1; // 0xFFFFFFFF = all 32 bits set
// Or for n < 32:
int fullMask = (1 << n) - 1; // only safe for n < 32
```

---

## 9. NOT in AND — Missing Parentheses

```cpp
// BUG: ~ has lower precedence than you think
n & ~1 << k    // = n & (~(1 << k)) — actually works here due to shift precedence

// SAFE: always parenthesize NOT
n & ~(1 << k)
```

---

## 10. Divide Two Integers — Inner Loop Overflow

```cpp
// BUG: tmp << 1 can overflow long long if tmp is near LLONG_MAX/2
while (dvd >= (tmp << 1)) { tmp <<= 1; ... }

// FIX: guard against overflow
while ((tmp << 1) > 0 && dvd >= (tmp << 1)) { tmp <<= 1; ... }
// OR: use dvd >= tmp && dvd - tmp >= tmp (avoid overflow entirely)
```

---

## Quick Checklist

- [ ] All bitwise conditions parenthesized: `(n & k) == 0` not `n & k == 0`?
- [ ] Using `(unsigned) >>` where sign extension would corrupt result?
- [ ] `(long long)` cast before negating/abs if input could be `INT_MIN`?
- [ ] `1LL <<` instead of `1 <<` for shifts ≥ 31?
- [ ] XOR swap guarded by `i != j`?
- [ ] Bitmask for n >= 32 uses `-1` not `(1 << 32) - 1`?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Java Bit Operators](./Java%20Bit%20Operators.md)

> **Last Updated:** 2026-06-26
