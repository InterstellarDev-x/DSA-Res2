# Java Bit Operators Reference

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Integer / Long Utility Methods

| Method | Returns | Example |
|--------|---------|---------|
| `Integer.bitCount(n)` | Number of 1-bits | `bitCount(7)` = 3 |
| `Integer.highestOneBit(n)` | `n` with only MSB kept | `highestOneBit(12)` = 8 |
| `Integer.lowestOneBit(n)` | `n` with only LSB kept | `lowestOneBit(12)` = 4 |
| `Integer.numberOfLeadingZeros(n)` | Leading zeros count | `nlz(8)` = 28 |
| `Integer.numberOfTrailingZeros(n)` | Trailing zeros count | `ntz(8)` = 3 |
| `Integer.reverse(n)` | Bits reversed | `reverse(1)` = -2147483648 |
| `Integer.reverseBytes(n)` | Bytes reversed | byte-order swap |
| `Integer.toBinaryString(n)` | Binary string, no padding | `"1010"` for 10 |
| `Integer.toHexString(n)` | Hex string | `"a"` for 10 |
| `Integer.signum(n)` | -1, 0, or 1 | sign of n |
| `Integer.MAX_VALUE` | 2^31 - 1 = 2147483647 | |
| `Integer.MIN_VALUE` | -2^31 = -2147483648 | |
| `Long.bitCount(n)` | Same for `long` | |

---

## Bitwise Assignment Operators

```java
n &= mask;   // n = n & mask
n |= mask;   // n = n | mask
n ^= mask;   // n = n ^ mask
n <<= k;     // n = n << k
n >>= k;     // n = n >> k  (arithmetic)
n >>>= k;    // n = n >>> k (logical)
```

---

## Bitmask Constants

```java
0xFF        // = 255, lowest 8 bits set
0xFFFF      // = 65535, lowest 16 bits
0xFFFFFFFF  // = -1 as int (all 32 bits set)
0xFFFFFFFFL // = 4294967295L as long (unsigned 32-bit max)
0x80000000  // = Integer.MIN_VALUE (only bit 31 set)
0x7FFFFFFF  // = Integer.MAX_VALUE (bits 0-30 set)
0xAAAAAAAA  // = bits 1,3,5,... (even positions from 0) = 2863311530
0x55555555  // = bits 0,2,4,... (odd positions from 0) = 1431655765
```

---

## Shift Rules

| Shift | Symbol | Effect | Sign-extends? |
|-------|--------|--------|--------------|
| Left | `<<` | × 2^k | N/A — zeros fill right |
| Arithmetic right | `>>` | ÷ 2^k (floor) | Yes — sign bit fills left |
| Logical right | `>>>` | ÷ 2^k (unsigned) | No — zeros fill left |

**Shift by ≥ 32 (int) or ≥ 64 (long):** Java takes `k % width`. So `1 << 32 == 1 << 0 == 1`.

---

## Two's Complement Quick Conversions

```java
-n      = ~n + 1       // negate
~n      = -n - 1       // bitwise NOT
n & -n  = lowest set bit of n
n | -n  = all bits set from lowest set bit upward
```

---

## Checking Individual Bits

```java
// Is bit k set in n?
boolean isSet = ((n >> k) & 1) == 1;
// OR equivalently:
boolean isSet = (n & (1 << k)) != 0;
```

---

## Java `java.util.BitSet`

```java
BitSet bs = new BitSet(100);  // 100-bit set, all 0
bs.set(5);           // set bit 5
bs.clear(5);         // clear bit 5
bs.flip(5);          // toggle bit 5
bs.get(5);           // true if bit 5 is set
bs.cardinality();    // number of set bits
bs.and(other);       // AND in place
bs.or(other);        // OR in place
bs.xor(other);       // XOR in place
bs.nextSetBit(from); // next set bit >= from
bs.size();           // allocated size (rounded to 64)
bs.length();         // index of highest set bit + 1
```

**Note:** `java.util.BitSet` is dynamically sized and uses `long[]` internally.

---

## Precedence Order (High → Low)

```
~  (bitwise NOT)
<< >> >>>  (shifts)
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
