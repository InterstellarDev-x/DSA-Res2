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

```java
public int getSum(int a, int b) {
    while (b != 0) {
        int carry = (a & b) << 1; // AND gives carry positions; left shift moves them up
        a = a ^ b;                 // XOR gives sum without carry
        b = carry;                 // carry becomes the new "b" to add
    }
    return a;
}
```

**Trace for a=2, b=3:**
- Iter 1: carry = (010 & 011) << 1 = 010 << 1 = 100. a = 010 ^ 011 = 001. b = 100
- Iter 2: carry = (001 & 100) << 1 = 000. a = 001 ^ 100 = 101 = 5. b = 0
- Done. Result = 5 ✅

### Amazon Follow-ups

**Q: What about subtraction?**
`a - b = a + (~b + 1) = getSum(a, getSum(~b, 1))`

**Q: What about overflow?**
In Java, `int` is 32-bit with overflow defined behavior (wraps around). No special handling needed — the carry eventually shifts out of 32 bits.

---

## Deep Dive: Single Number II

**Method 1 — State machine (O(1) space):**
```java
int ones = 0, twos = 0;
for (int n : nums) {
    ones = (ones ^ n) & ~twos;
    twos = (twos ^ n) & ~ones;
}
return ones;
```

**Method 2 — Bit counting (more explainable in interviews):**
```java
int result = 0;
for (int bit = 0; bit < 32; bit++) {
    int bitSum = 0;
    for (int n : nums) bitSum += (n >> bit) & 1;
    result |= (bitSum % 3) << bit;
}
return result;
```

Method 2 is easier to explain: "Count how many numbers have each bit set. Divide by 3. Remainder = bit value of the unique number." Amazon interviewers appreciate the clear explanation.

---

## Amazon LP Alignment

| Problem | LP Principle |
|---------|-------------|
| Single Number | Invent and Simplify — O(1) XOR vs O(n) HashSet |
| Sum of Two Integers | Dive Deep — carry simulation requires bit-level reasoning |
| Single Number II | Are Right, A Lot — both methods work; explain tradeoffs |

---

## Related Files

- [Amazon OA-Qns](../OA-Qns/Amazon.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
