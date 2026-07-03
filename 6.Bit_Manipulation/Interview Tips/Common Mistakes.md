# Common Mistakes — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Operator Precedence: Missing Parentheses

```rust
// BUG: == binds tighter than &
// if n & 1 == 0 { }  // parsed as n & (1 == 0) — Rust: type error (bool vs int), won't compile

// FIX
if (n & 1) == 0 { }
```

---

## 2. Arithmetic vs Logical Right Shift

```rust
// BUG: >> sign-extends; for negative n, fills with 1s
let n: i32 = -8; // binary: 11111000
let _ = n >> 1; // = -4 (11111100) — fills with 1

// FIX: cast to u32 for logical (fill-with-0) right shift
let _ = (n as u32) >> 1; // = 2147483644 (01111100)
```

---

## 3. i32::MIN Negation Overflow

```rust
// BUG
let n: i32 = i32::MIN;
// let pos: i32 = -n; // panics in debug mode, wraps in release

// FIX: cast to i64
let pos: i64 = -(n as i64); // = 2147483648
```

---

## 4. `1 << k` vs `1_i64 << k` for k >= 31

```rust
// BUG: 1 is i32; 1 << 31 overflows i32 (panic in debug mode)
// let mask: i64 = 1 << 31; // 1 inferred as i32 — overflow!

// FIX: use i64 literal
let mask: i64 = 1_i64 << 31; // = 2147483648
```

---

## 5. XOR Swap When a == b (Same Reference)

```rust
// BUG: if i == j, both become 0
nums[i] ^= nums[j];
nums[j] ^= nums[i];
nums[i] ^= nums[j];
// if i == j: nums[i] = 0

// FIX: guard first
if i != j { /* swap */ }
```

---

## 6. `n & (n-1)` vs `n & (-n)` Confusion

```rust
n & (n - 1)  // clears lowest set bit of n
n & (-n)     // isolates lowest set bit of n (since -n = !n + 1)
```

Don't mix these up — one removes the bit, the other keeps only that bit.

---

## 7. Single Number II — Wrong State Machine Ordering

```rust
// BUG: updating ones before twos with the same n causes interference
ones = (ones ^ n) & !twos;
twos = (twos ^ n) & !ones; // ones already updated — uses new ones (actually correct order)
```

The correct order is: update `ones` first (using old `twos`), then `twos` (using new `ones`). This IS the correct order above. Reversing it breaks the state machine.

---

## 8. Bitmask Overflow for n >= 32

```rust
// BUG: 1 << 32 in Rust = panic (shift overflow in debug mode)
// let full_mask: i32 = (1 << 32) - 1; // panics or wraps — WRONG for n=32

// FIX: use -1 for all-bits mask
let full_mask: i32 = -1; // 0xFFFFFFFF = all 32 bits set (as i32)
let full_mask: u32 = u32::MAX; // preferred for unsigned all-bits mask
// Or for n < 32:
let full_mask: i32 = (1 << n) - 1; // only safe for n < 32
```

---

## 9. NOT in AND — Missing Parentheses

```rust
// BUG: ! has lower precedence than you think
// n & !1 << k    // = n & (!(1 << k)) — actually works here due to shift precedence

// SAFE: always parenthesize NOT
n & !(1 << k)
```

---

## 10. Divide Two Integers — Inner Loop Overflow

```rust
// BUG: tmp << 1 can overflow i64 if tmp is near i64::MAX / 2
// while dvd >= (tmp << 1) { tmp <<= 1; ... }  // panics on overflow in debug

// FIX: guard against overflow using checked_shl
while tmp.checked_shl(1).map_or(false, |t| dvd >= t) { tmp <<= 1; /* ... */ }
// OR: avoid overflow entirely
while dvd >= tmp && dvd - tmp >= tmp { tmp <<= 1; /* ... */ }
```

---

## Quick Checklist

- [ ] All bitwise conditions parenthesized: `(n & k) == 0` not `n & k == 0`?
- [ ] Using `as u32` cast for logical right shift where sign extension would corrupt result?
- [ ] `as i64` cast before negating/abs if input could be `i32::MIN`?
- [ ] `1_i64 <<` instead of `1 <<` for shifts ≥ 31?
- [ ] XOR swap guarded by `i != j`?
- [ ] Bitmask for n >= 32 uses `-1` not `(1 << 32) - 1`?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Java Bit Operators](./Java%20Bit%20Operators.md)

> **Last Updated:** 2026-06-26
