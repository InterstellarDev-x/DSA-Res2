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
| NOT | `~` | `!5` = `!00000101` | `11111010` = -6 |
| Left Shift | `<<` | `1 << 3` | `1000` = 8 |
| Arithmetic Right | `>>` | `-8 >> 1` | `-4` (sign-extends) |
| Logical Right | `n as u32 >> k` | `-8i32 as u32 >> 1` | `2147483644` (fills with 0) |

**Note:** `!n == -(n+1)` in two's complement. `!0i32 == -1`, `!(-1i32) == 0`.

---

## Core Operations

```rust
fn get_bit(n: i32, k: i32) -> i32    { (n >> k) & 1 }
fn set_bit(n: i32, k: i32) -> i32    { n | (1 << k) }
fn clear_bit(n: i32, k: i32) -> i32  { n & !(1 << k) }
fn toggle_bit(n: i32, k: i32) -> i32 { n ^ (1 << k) }
fn is_power_of_2(n: i32) -> bool     { n > 0 && (n & (n - 1)) == 0 }
fn lowest_set_bit(n: i32) -> i32     { n & n.wrapping_neg() }  // isolate
fn clear_lowest(n: i32) -> i32       { n & (n - 1) }           // clear lowest
```

---

## Template 1 — Reverse Bits (LC 190)

Process 32 bits one at a time: extract LSB from `n`, put into MSB of result.

```rust
fn reverse_bits(mut n: u32) -> u32 {
    let mut result: u32 = 0;
    for _ in 0..32 {
        result = (result << 1) | (n & 1); // shift result left, put current LSB
        n >>= 1; // logical shift for u32 — fills with 0
    }
    result
}
```

**Note:** In Rust, `>>` on `u32` is already a logical (zero-filling) right shift, so no cast is needed. Using `u32` for this function avoids sign-extension issues entirely.

---

## Template 2 — Power of Two (LC 231)

A power of 2 has exactly one set bit. `n & (n-1)` clears the lowest set bit.

```rust
fn is_power_of_two(n: i32) -> bool {
    n > 0 && (n & (n - 1)) == 0
}
```

**Generalize:** Power of 4 has its single bit at an even position: `n > 0 && (n & (n-1)) == 0 && (n & 0xAAAAAAAAu32 as i32) == 0` (0xAAAAAAAA = all even-position bits set).

---

## Template 3 — Sum of Two Integers (no + or −) (LC 371)

Simulate binary addition with carry:

```rust
fn get_sum(mut a: i32, mut b: i32) -> i32 {
    while b != 0 {
        let carry = (a & b).wrapping_shl(1); // carry: positions where both bits are 1
        a = a ^ b;                             // sum without carry
        b = carry;
    }
    a
}
```

**Loop invariant:** At each iteration, `a` holds the partial sum (no carry), `b` holds the carry to be added. When `b == 0`, no more carries — `a` is the answer.

**Note:** In Rust, `i32` is always 32-bit. `wrapping_shl` is used to avoid debug-mode overflow panics on the carry shift. In Python, integers are unbounded — need `& 0xFFFFFFFF` masking.

---

## Template 4 — Divide Two Integers (LC 29)

Use bit shifts to find the largest multiple of divisor ≤ dividend:

```rust
fn divide(dividend: i32, divisor: i32) -> i32 {
    if dividend == i32::MIN && divisor == -1 {
        return i32::MAX; // overflow
    }

    let mut dvd = (dividend as i64).abs();
    let dvs = (divisor as i64).abs();
    let sign: i64 = if (dividend > 0) == (divisor > 0) { 1 } else { -1 };
    let mut result: i64 = 0;

    while dvd >= dvs {
        let mut tmp = dvs;
        let mut multiple: i64 = 1;
        while dvd >= (tmp << 1) {
            tmp <<= 1;
            multiple <<= 1;
        }
        dvd -= tmp;
        result += multiple;
    }
    (sign * result) as i32
}
```

**Key:** `tmp << 1` doubles the divisor — find the largest `2^k × divisor ≤ dividend` in O(log² n).

---

## Template 5 — Bitwise AND of Numbers Range (LC 201)

All numbers in [m, n] AND'd together. Any differing bit in the range will have both 0 and 1 → AND = 0.

The result is the **common prefix** of m and n in binary.

```rust
fn range_bitwise_and(mut m: i32, mut n: i32) -> i32 {
    let mut shift: u32 = 0;
    while m != n {
        m >>= 1;
        n >>= 1;
        shift += 1;
    }
    m << shift // common prefix shifted back
}
```

**Alternative — Brian Kernighan:** Keep clearing lowest set bit of `n` until `n <= m`.
```rust
while n > m { n &= n - 1; }
n
```

---

## Template 6 — Concatenation of Consecutive Binary Numbers (LC 1680)

Numbers 1, 2, ..., n concatenated in binary. Result mod 10^9+7.

```rust
fn concatenated_binary(n: i32) -> i32 {
    let mut result: i64 = 0;
    const MOD: i64 = 1_000_000_007;
    for i in 1..=n {
        let bits = (i as f64).log2() as i32 + 1; // bit length of i
        // OR: bits = 32 - (i as u32).leading_zeros() as i32
        result = ((result << bits) | i as i64) % MOD;
    }
    result as i32
}
```

---

## Shift Operator Precedence Trap

```rust
// BUG: + has higher precedence than <<
// 1 << n + 1  // = 1 << (n+1), not (1 << n) + 1  -- same rules apply in Rust!

// SAFE: always parenthesize shifts
(1 << n) + 1
```

---

## Two's Complement Reminders

- `-n == !n + 1`
- `n & n.wrapping_neg()` = lowest set bit (from two's complement)
- `i32::MIN = -2^31` has no positive counterpart in `i32` → use `i64`
- `!i32::MIN == i32::MAX`

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Get/Set/Clear/Toggle bit | O(1) | O(1) |
| Reverse Bits | O(32) = O(1) | O(1) |
| Power of Two | O(1) | O(1) |
| Sum (get_sum) | O(32) = O(1) | O(1) |
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
