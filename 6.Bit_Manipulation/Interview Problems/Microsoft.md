# Microsoft Interview Problems — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Divide Two Integers | SDE II | Medium | ⭐⭐⭐⭐ | [Bit Basics](../Patterns/Bit%20Basics.md) | [29](https://leetcode.com/problems/divide-two-integers/) |
| 2 | Reverse Bits | SDE I | Easy | ⭐⭐⭐⭐ | [Bit Basics](../Patterns/Bit%20Basics.md) | [190](https://leetcode.com/problems/reverse-bits/) |
| 3 | Single Number | SDE I | Easy | ⭐⭐⭐ | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [136](https://leetcode.com/problems/single-number/) |
| 4 | Counting Bits | SDE I | Easy | ⭐⭐⭐ | [Counting Bits](../Patterns/Counting%20Bits.md) | [338](https://leetcode.com/problems/counting-bits/) |

---

## Deep Dive: Divide Two Integers

### Key Considerations

1. **No `*`, `/`, `%`** — use bit shifts
2. **Overflow:** `i32::MIN / -1 = 2^31` — exceeds `i32::MAX`, return `i32::MAX`
3. **Signs:** Compute absolute values; apply sign at end
4. **Use `i64`** — `abs(i32::MIN)` overflows `i32`

```rust
fn divide(dividend: i32, divisor: i32) -> i32 {
    if dividend == i32::MIN && divisor == -1 {
        return i32::MAX;
    }
    let mut dvd = (dividend as i64).abs();
    let dvs = (divisor as i64).abs();
    let sign: i64 = if (dividend > 0) == (divisor > 0) { 1 } else { -1 };
    let mut result: i64 = 0;

    while dvd >= dvs {
        let mut tmp = dvs;
        let mut multiple: i64 = 1;
        while dvd >= (tmp << 1) && (tmp << 1) > 0 {
            tmp <<= 1;
            multiple <<= 1;
        }
        dvd -= tmp;
        result += multiple;
    }
    (sign * result) as i32
}
```

**Complexity:** O(log² n) — outer loop runs log(n) times, inner loop runs log(n) times.

### Microsoft Follow-ups

**Q: What if you could only use shifts, not multiplication in the inner loop check?**
`tmp << 1` IS a shift — this is already compliant.

**Q: Handle when dvd = i64::MAX?**
`tmp << 1` could overflow `i64`. Guard: `dvd >= (tmp << 1) && (tmp << 1) > 0`.

---

## Related Files

- [Microsoft OA-Qns](../OA-Qns/Microsoft.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
