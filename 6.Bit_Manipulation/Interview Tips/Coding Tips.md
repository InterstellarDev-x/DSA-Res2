# Coding Tips — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## The Six Core Operations (Memorize)

```java
int getBit(int n, int k)    { return (n >> k) & 1; }
int setBit(int n, int k)    { return n | (1 << k); }
int clearBit(int n, int k)  { return n & ~(1 << k); }
int toggleBit(int n, int k) { return n ^ (1 << k); }
int lowestSetBit(int n)     { return n & (-n); }
int clearLowest(int n)      { return n & (n - 1); }
```

---

## `>>>` vs `>>` in Java

```java
-1 >> 1   // = -1  (arithmetic: sign-extends, fills with 1)
-1 >>> 1  // = Integer.MAX_VALUE (logical: fills with 0)
```

**Rule:** Use `>>>` when dealing with bit patterns that should not be sign-extended (e.g., Reverse Bits, hash functions, byte manipulation).

---

## 1L vs 1 for Bit Shifts

```java
int n = 32;
1 << n      // BUG: 1 << 32 = 1 (undefined behavior for shift ≥ width in Java — actually 1 << (32%32) = 1<<0)
1 << 31     // OK: = Integer.MIN_VALUE = -2^31
1L << 31    // = 2147483648 (positive long)
1L << 32    // = 4294967296 (safe with long)
```

**Rule:** For shifts ≥ 31, use `1L` (long literal).

---

## Operator Precedence

Bitwise operators have lower precedence than arithmetic and comparison:

```java
// BUG: == has higher precedence than &
if (n & 1 == 0)   // parsed as: n & (1 == 0) = n & false = n & 0 = 0

// FIX: always parenthesize bitwise operations
if ((n & 1) == 0)
```

Common traps:
```java
n & n - 1    // = n & (n-1) ✅ (subtraction before &)
n | 1 << k   // = n | (1 << k) ✅ (shift before |)
~n + 1       // = (~n) + 1 ✅ (NOT before +)
```

---

## Integer.MIN_VALUE Invariants

```java
Integer.MIN_VALUE          // = -2147483648 = -2^31 = 0x80000000
Integer.MAX_VALUE          // = 2147483647  = 2^31 - 1 = 0x7FFFFFFF
-Integer.MIN_VALUE         // OVERFLOW: still Integer.MIN_VALUE
Math.abs(Integer.MIN_VALUE)// OVERFLOW: still Integer.MIN_VALUE
(long)Integer.MIN_VALUE    // = -2147483648L (safe)
```

**Rule:** Whenever negating or taking abs of a value that might be `Integer.MIN_VALUE`, cast to `long` first.

---

## XOR for In-Place Swap (No Temp)

```java
// Only works if a and b are DIFFERENT variables/addresses
a ^= b; // a = a^b
b ^= a; // b = b^(a^b) = a
a ^= b; // a = (a^b)^a = b
```

**Caution:** If `a` and `b` point to the same memory location (same index), this sets them both to 0.

---

## Bitmask Idioms

```java
int fullMask = (1 << n) - 1;  // all n bits set: 0b111...1 (n ones)
int highBit  = 1 << (n - 1);  // only highest bit set
int evenBits = 0xAAAAAAAA;     // bits 1,3,5,... (even positions from 0)
int oddBits  = 0x55555555;     // bits 0,2,4,... (odd positions from 0)
```

---

## Counting Set Bits — Three Ways

```java
// 1. Brian Kernighan (fastest for sparse bits)
while (n != 0) { n &= (n-1); count++; }

// 2. Java built-in (fastest overall)
Integer.bitCount(n);

// 3. Lookup table (for repeated calls, O(1) per call after O(256) build)
int[] lookup = new int[256];
for (int i = 1; i < 256; i++) lookup[i] = (i & 1) + lookup[i >> 1];
// count bits in n: lookup[n&0xFF] + lookup[(n>>8)&0xFF] + ...
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Java Bit Operators](./Java%20Bit%20Operators.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
