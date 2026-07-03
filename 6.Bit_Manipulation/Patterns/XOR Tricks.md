# XOR Tricks

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Single Number I/II/III, Missing Number, XOR Subarray Queries

---

## XOR Properties (Memorize These)

| Property | Expression | Meaning |
|----------|-----------|---------|
| Self-inverse | `a ^ a = 0` | Any number XORed with itself = 0 |
| Identity | `a ^ 0 = a` | XOR with 0 = no change |
| Commutative | `a ^ b = b ^ a` | Order doesn't matter |
| Associative | `(a^b)^c = a^(b^c)` | Grouping doesn't matter |
| Cancellation | `a ^ b ^ a = b` | a's cancel |
| Swap | `a ^= b; b ^= a; a ^= b` | Swaps without temp variable |

**Core use:** XOR all elements in a set — duplicates cancel (→ 0), unique element survives.

---

## Template 1 — Single Number (LC 136)

Every number appears twice except one. Find the unique one.

```rust
fn single_number(nums: &[i32]) -> i32 {
    let mut result = 0;
    for &n in nums {
        result ^= n;
    }
    result // all pairs cancel; unique remains
}
```

**Time:** O(n). **Space:** O(1). No `HashSet` needed.

---

## Template 2 — Missing Number (LC 268)

Array of n numbers in range [0, n], one missing. XOR all indices 0..n with all values:

```rust
fn missing_number(nums: &[i32]) -> i32 {
    let mut result = nums.len() as i32; // start with n
    for (i, &n) in nums.iter().enumerate() {
        result ^= i as i32 ^ n; // XOR index and value cancel if equal; missing value survives
    }
    result
}
```

**Alternative:** `sum(0..n) - sum(nums)` = `n*(n+1)/2 - nums.iter().sum::<i32>()`. Both O(n) O(1).

---

## Template 3 — Single Number II (LC 137)

Every number appears **three times** except one. Cannot use XOR directly (triples don't cancel).

**Bit counting approach:** Count bits mod 3. Any bit that appears 3k times cancels; the remaining bit belongs to the unique number.

```rust
fn single_number(nums: &[i32]) -> i32 {
    let mut ones = 0i32;
    let mut twos = 0i32;
    for &n in nums {
        ones = (ones ^ n) & !twos; // bits seen once (not in twos)
        twos = (twos ^ n) & !ones; // bits seen twice (not in ones)
        // bits seen three times fall out of both
    }
    ones
}
```

**State machine:** Each bit cycles through states 0 → 1 → 2 → 0. `ones` tracks bits at state 1, `twos` at state 2.

**Bit-by-bit approach (clearer but O(32)):**
```rust
fn single_number(nums: &[i32]) -> i32 {
    let mut result = 0i32;
    for bit in 0..32 {
        let mut sum = 0i32;
        for &n in nums {
            sum += (n >> bit) & 1;
        }
        result |= (sum % 3) << bit;
    }
    result
}
```

---

## Template 4 — Single Number III (LC 260)

Two numbers appear once; all others appear twice. Find both.

**Key insight:** `xor_val = a ^ b` (the XOR of the two unique numbers). Any set bit in `xor_val` is a bit where `a` and `b` differ — use it to partition the array.

```rust
fn single_number(nums: &[i32]) -> Vec<i32> {
    let mut xor_val = 0i32;
    for &n in nums {
        xor_val ^= n;
    }

    // Find any differing bit (e.g., lowest set bit)
    let diff = xor_val & (-xor_val); // isolate lowest set bit

    let mut a = 0i32;
    for &n in nums {
        if (n & diff) != 0 {
            a ^= n; // group 1: numbers with this bit set
        }
    }
    vec![a, xor_val ^ a] // b = xor_val ^ a
}
```

**Why partition works:** `diff` bit is 1 in `a` and 0 in `b` (or vice versa). So they end up in different groups. Duplicates still cancel within each group. Each group's XOR reveals one unique number.

---

## Template 5 — XOR Queries of a Subarray (LC 1310)

Prefix XOR — analogous to prefix sum but with XOR.

```rust
fn xor_queries(arr: &[i32], queries: &[Vec<i32>]) -> Vec<i32> {
    let n = arr.len();
    let mut prefix = vec![0i32; n + 1]; // prefix[i] = XOR of arr[0..i-1]
    for i in 0..n {
        prefix[i + 1] = prefix[i] ^ arr[i];
    }

    let mut result = vec![0i32; queries.len()];
    for (i, q) in queries.iter().enumerate() {
        let l = q[0] as usize;
        let r = q[1] as usize;
        result[i] = prefix[r + 1] ^ prefix[l]; // XOR[l..r] = prefix[r+1] ^ prefix[l]
    }
    result
}
```

**Why prefix XOR works:** `prefix[r+1] ^ prefix[l]` = XOR of all elements in `arr[0..r]` XOR'd with XOR of `arr[0..l-1]`. The first `l` elements cancel (each appears twice), leaving `arr[l..r]`.

---

## Template 6 — Find XOR of Numbers from L to R

`XOR(L, R) = XOR(0, L-1) ^ XOR(0, R)`

XOR from 0 to n has a 4-cycle:
```rust
fn xor_up_to(n: i32) -> i32 {
    match n % 4 {
        0 => n,
        1 => 1,
        2 => n + 1,
        3 => 0,
        _ => -1, // unreachable
    }
}
fn xor_range(l: i32, r: i32) -> i32 { xor_up_to(r) ^ xor_up_to(l - 1) }
```

---

## Common XOR Patterns Summary

| Problem Type | Technique |
|-------------|-----------|
| One duplicate, rest appear twice | `XOR all = unique` |
| One missing in [0..n] | `XOR all indices ^ all values` |
| Every element 3×, one 1× | State machine (`ones`, `twos`) or bit sum mod 3 |
| Two unique, rest 2× | `XOR all` → partition by any set bit |
| Range XOR query | Prefix XOR array |
| XOR of [1..n] | 4-cycle trick |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Single Number I | O(n) | O(1) |
| Single Number II (state machine) | O(n) | O(1) |
| Single Number III | O(n) | O(1) |
| Missing Number | O(n) | O(1) |
| XOR Queries | O(n + q) | O(n) |
| XOR(L, R) | O(1) | O(1) |

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — AND/OR/shift foundations
- [Counting Bits](./Counting%20Bits.md) — popcount and bit counting
- [Prefix Sum](../../1.Arrays/Patterns/Prefix%20Sum.md) — prefix XOR is analogous

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [Bit Basics](./Bit%20Basics.md) | **Next:** [Counting Bits](./Counting%20Bits.md)

> **Last Updated:** 2026-06-26
