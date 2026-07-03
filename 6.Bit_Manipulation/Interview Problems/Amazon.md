# Amazon Interview Problems — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Single Number | SDE I | Easy | ⭐⭐⭐⭐⭐ | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [136](https://leetcode.com/problems/single-number/) |
| 2 | Sum of Two Integers | SDE I | Medium | ⭐⭐⭐⭐ | [Bit Basics](../Patterns/Bit%20Basics.md) | [371](https://leetcode.com/problems/sum-of-two-integers/) |
| 3 | Single Number II | SDE II | Medium | ⭐⭐⭐ | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [137](https://leetcode.com/problems/single-number-ii/) |
| 4 | Maximum XOR of Two Numbers | SDE II | Medium | ⭐⭐⭐ | [Bit Search](../Patterns/Bit%20Search%20and%20Trie.md) | [421](https://leetcode.com/problems/maximum-xor-of-two-numbers-in-an-array/) |

---

## Deep Dive: Sum of Two Integers

### The Carry Simulation Loop

```rust
fn get_sum(a: i32, b: i32) -> i32 {
    let mut a = a as u32;
    let mut b = b as u32;
    while b != 0 {
        let carry = (a & b) << 1; // AND gives carry positions; left shift moves them up
        a = a ^ b;                 // XOR gives sum without carry
        b = carry;                 // carry becomes the new "b" to add
    }
    a as i32
}
```

**Trace for a=2, b=3:**
- Iter 1: carry = (010 & 011) << 1 = 010 << 1 = 100. a = 010 ^ 011 = 001. b = 100
- Iter 2: carry = (001 & 100) << 1 = 000. a = 001 ^ 100 = 101 = 5. b = 0
- Done. Result = 5 ✅

### Amazon Follow-ups

**Q: What about subtraction?**
`a - b = a + (~b + 1) = get_sum(a, get_sum(!b as i32, 1))`

**Q: What about overflow?**
In Rust, left-shifting a signed `i32` can panic in debug mode on overflow; cast to `u32` for safe wraparound. The carry eventually shifts out of 32 bits.

---

## Deep Dive: Single Number II

**Method 1 — State machine (O(1) space):**
```rust
let mut ones = 0i32;
let mut twos = 0i32;
for &n in &nums {
    ones = (ones ^ n) & !twos;
    twos = (twos ^ n) & !ones;
}
// ones holds the single number
```

**Method 2 — Bit counting (more explainable in interviews):**
```rust
let mut result = 0i32;
for bit in 0..32 {
    let mut bit_sum = 0i32;
    for &n in &nums { bit_sum += (n >> bit) & 1; }
    result |= (bit_sum % 3) << bit;
}
// result holds the single number
```

Method 2 is easier to explain: "Count how many numbers have each bit set. Divide by 3. Remainder = bit value of the unique number." Amazon interviewers appreciate the clear explanation.

---

## Amazon LP Alignment

| Problem | LP Principle |
|---------|-------------|
| Single Number | Invent and Simplify — O(1) XOR vs O(n) `HashSet` |
| Sum of Two Integers | Dive Deep — carry simulation requires bit-level reasoning |
| Single Number II | Are Right, A Lot — both methods work; explain tradeoffs |

---

## Related Files

- [Amazon OA-Qns](../OA-Qns/Amazon.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
