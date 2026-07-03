# Rust Bit Operators Reference

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Integer / Long Utility Methods

| Method | Returns | Example |
|--------|---------|---------|
| `n.count_ones()` | Number of 1-bits | `7u32.count_ones()` = 3 |
| `1u32 << (31 - n.leading_zeros())` | `n` with only MSB kept | result for 12 = 8 |
| `n & n.wrapping_neg()` | `n` with only LSB kept | result for 12 = 4 |
| `n.leading_zeros()` | Leading zeros count | `8u32.leading_zeros()` = 28 |
| `n.trailing_zeros()` | Trailing zeros count | `8u32.trailing_zeros()` = 3 |
| `n.reverse_bits()` | Bits reversed | built-in method on all integer types |
| `n.swap_bytes()` | Bytes reversed | byte-order swap |
| `format!("{:b}", n)` | Binary string, no padding | `"1010"` for 10 |
| `format!("{:x}", n)` | Hex string | `"a"` for 10 |
| `n.signum()` | -1, 0, or 1 | sign of n |
| `i32::MAX` | 2^31 - 1 = 2147483647 | |
| `i32::MIN` | -2^31 = -2147483648 | |
| `n.count_ones()` (on `i64`/`u64`) | Number of 1-bits for `i64` | |

---

## Bitwise Assignment Operators

```rust
n &= mask;              // n = n & mask
n |= mask;              // n = n | mask
n ^= mask;              // n = n ^ mask
n <<= k;                // n = n << k
n >>= k;                // n = n >> k  (arithmetic, sign-extends for signed types)
n = (n as u32) >> k;    // logical right shift (cast to unsigned for zero-fill)
```

---

## Bitmask Constants

```rust
0xFF_u32         // = 255, lowest 8 bits set
0xFFFF_u32       // = 65535, lowest 16 bits
0xFFFF_FFFFu32   // = 4294967295 as u32 (all 32 bits set); -1i32 in two's complement
0xFFFF_FFFFu32   // = u32::MAX (unsigned 32-bit max)
0x8000_0000u32   // = i32::MIN as bits (only bit 31 set)
0x7FFF_FFFFu32   // = i32::MAX as bits (bits 0-30 set)
0xAAAA_AAAAu32   // = bits 1,3,5,... (even positions from 0) = 2863311530
0x5555_5555u32   // = bits 0,2,4,... (odd positions from 0) = 1431655765
```

---

## Shift Rules

| Shift | Symbol | Effect | Sign-extends? |
|-------|--------|--------|--------------|
| Left | `<<` | × 2^k | N/A — zeros fill right |
| Arithmetic right | `>>` | ÷ 2^k (floor) | Yes — sign bit fills left (for signed types) |
| Logical right | `(n as u32) >> k` | ÷ 2^k (unsigned) | No — zeros fill left |

**Shift by >= 32 (`i32`) or >= 64 (`i64`):** In Rust, shifting by >= the bit width is a **panic in debug mode** and **undefined behavior in release mode** (wraps). Use `checked_shl`/`checked_shr` or ensure `0 <= k < 32` (or 64 for `i64`). Rust does NOT take `k % width` automatically.

---

## Two's Complement Quick Conversions

```rust
// negate (wrapping)
let neg_n = n.wrapping_neg();     // equivalent to ~n + 1
// bitwise NOT
let not_n = !n;                   // equivalent to -n - 1
// lowest set bit
let lsb = n & n.wrapping_neg();
// all bits set from lowest set bit upward
let upper = n | n.wrapping_neg();
```

---

## Checking Individual Bits

```rust
// Is bit k set in n?
let is_set: bool = ((n >> k) & 1) == 1;
// OR equivalently:
let is_set: bool = (n & (1 << k)) != 0;
```

---

## Rust Bit Manipulation (no `std::bitset` — use integers or `Vec<u64>`)

Rust's standard library has no direct `bitset<N>` type. For small fixed-size bit sets use integer primitives; for large dynamic bit sets use `Vec<u64>` manually or the `bitvec` crate.

```rust
// Fixed-size bitset using u128 (up to 128 bits)
let mut bs: u128 = 0;            // all 0 bits

// set bit 5
bs |= 1u128 << 5;

// clear bit 5
bs &= !(1u128 << 5);

// toggle bit 5
bs ^= 1u128 << 5;

// test bit 5
let is_set = (bs >> 5) & 1 == 1;

// count set bits
let count = bs.count_ones();

// AND / OR / XOR in place
bs &= other;
bs |= other;
bs ^= other;

// next set bit after position `from`  (manual scan)
fn next_set_bit(bs: u128, from: u32) -> Option<u32> {
    let shifted = bs >> (from + 1);
    if shifted == 0 { None } else { Some(from + 1 + shifted.trailing_zeros()) }
}

// fixed size
let size: u32 = u128::BITS; // 128

// index of first set bit
let first = bs.trailing_zeros(); // u128::BITS (128) if none set
```

**Note:** For truly dynamic or very large bit sets, use `Vec<u64>` with manual indexing (`word = i / 64`, `bit = i % 64`) or the `bitvec` crate. Unlike C++'s `std::bitset<N>`, Rust integer types require the width to fit in a primitive; `Vec<bool>` is an alternative but does not pack bits.

---

## Precedence Order (High → Low)

```
!  (bitwise NOT)
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
