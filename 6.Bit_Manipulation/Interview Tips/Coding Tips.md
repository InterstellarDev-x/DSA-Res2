# Coding Tips — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## The Six Core Operations (Memorize)

```rust
fn get_bit(n: i32, k: i32) -> i32    { (n >> k) & 1 }
fn set_bit(n: i32, k: i32) -> i32    { n | (1 << k) }
fn clear_bit(n: i32, k: i32) -> i32  { n & !(1 << k) }
fn toggle_bit(n: i32, k: i32) -> i32 { n ^ (1 << k) }
fn lowest_set_bit(n: i32) -> i32     { n & (-n) }
fn clear_lowest(n: i32) -> i32       { n & (n - 1) }
```

---

## Arithmetic vs Logical Right Shift in Rust

```rust
(-1i32) >> 1                // = -1  (arithmetic: sign-extends, fills with 1 — well-defined for signed in Rust)
(-1i32 as u32) >> 1         // = i32::MAX as u32 (logical: fills with 0, cast to unsigned first)
```

**Rule:** Use `n as u32 >> k` when dealing with bit patterns that should not be sign-extended (e.g., Reverse Bits, hash functions, byte manipulation).

---

## 1i64 vs 1i32 for Bit Shifts

```rust
let n: i32 = 32;
1i32 << n      // BUG: panics in debug mode if shift >= 32
1i32 << 31     // panics in debug (overflow); wraps in release
1i64 << 31     // = 2147483648 (positive i64)
1i64 << 32     // = 4294967296 (safe with i64)
```

**Rule:** For shifts >= 31, use `1i64` (i64 literal).

---

## Operator Precedence

Bitwise operators have lower precedence than arithmetic, but higher than comparison in Rust:

```rust
// In Rust, & has HIGHER precedence than ==, so this is safe:
if n & 1 == 0 { }   // parsed as: (n & 1) == 0 (correct in Rust!)

// Still good practice to parenthesize for clarity:
if (n & 1) == 0 { }
```

Common traps:
```rust
n & n - 1    // = n & (n-1) // (subtraction before &)
n | 1 << k   // = n | (1 << k) // (shift before |)
!n + 1       // = (!n) + 1 // (bitwise NOT before +)
```

---

## i32::MIN / i32::MAX Invariants

```rust
i32::MIN                // = -2147483648 = -2^31 = 0x80000000
i32::MAX                // = 2147483647  = 2^31 - 1 = 0x7FFFFFFF
-i32::MIN               // panics in debug mode (overflow)
i32::MIN.abs()          // panics in debug mode (overflow)
i32::MIN as i64         // = -2147483648i64 (safe)
```

**Rule:** Whenever negating or taking abs of a value that might be `i32::MIN`, cast to `i64` first.

---

## XOR for In-Place Swap (No Temp)

```rust
// Only works if a and b are DIFFERENT variables/addresses
a ^= b; // a = a^b
b ^= a; // b = b^(a^b) = a
a ^= b; // a = (a^b)^a = b
// Note: prefer std::mem::swap(&mut a, &mut b) in idiomatic Rust
```

**Caution:** If `a` and `b` point to the same memory location (same index), this sets them both to 0.

---

## Bitmask Idioms

```rust
let full_mask: i32 = (1 << n) - 1;      // all n bits set: 0b111...1 (n ones)
let high_bit: i32  = 1 << (n - 1);      // only highest bit set
let even_bits: u32 = 0xAAAAAAAAu32;      // bits 1,3,5,... (even positions from 0)
let odd_bits: u32  = 0x55555555u32;      // bits 0,2,4,... (odd positions from 0)
```

---

## Counting Set Bits — Three Ways

```rust
// 1. Brian Kernighan (fastest for sparse bits)
while n != 0 { n &= n - 1; count += 1; }

// 2. Built-in (fastest overall)
n.count_ones();

// 3. Lookup table (for repeated calls, O(1) per call after O(256) build)
let mut lookup = vec![0i32; 256];
for i in 1..256 { lookup[i] = (i & 1) as i32 + lookup[i >> 1]; }
// count bits in n: lookup[n&0xFF] + lookup[(n>>8)&0xFF] + ...
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [C++ Bit Operators](./Java%20Bit%20Operators.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
