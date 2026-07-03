> **Topic:** [Tries](../README.md) · **Pattern 3 of 3**

# Bitwise Trie (XOR)

## Core Concept

A **bitwise trie** (a.k.a. binary trie) stores integers as fixed-width binary strings instead of letter strings. Each node has exactly **two children**: bit `0` and bit `1`. We insert the bits from the **most significant bit (MSB) first**, because XOR maximization is a greedy decision that must start from the highest-value bit.

The classic application is **Maximum XOR of two numbers**: for each number, we want a partner whose bits differ as much as possible at the high end. A binary trie lets us, for each number, greedily walk toward the *opposite* bit at every level — if that child exists, this bit of the XOR is `1` (the best possible); otherwise we're forced down the same-bit child.

**Recognition signals:** "maximum XOR", "maximize/minimize XOR of a pair", "bit-by-bit greedy over an array of integers", queries involving XOR against a set.

This is the same structure documented in the Bit Manipulation topic — see [Bit Search and Trie](../6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md).

---

## Problem 6 — Maximum XOR of Two Numbers in an Array · LC 421 · Medium

Given `nums`, find `max(nums[i] XOR nums[j])`. Constraints fit in 32-bit ints; for LeetCode's range we use **31 bits** (`0 .. 2^31 - 1`, so bit indices `30 .. 0`). Using 32 bits also works as long as you're consistent.

```rust
const HIGH_BIT: i32 = 30; // 31-bit numbers: MSB index 30

struct Trie {
    // Each node has two children indexed by bit (0 or 1)
    // Index-based arena: root is at index 0, None means no child
    children: Vec<[Option<usize>; 2]>,
}

impl Trie {
    fn new() -> Self {
        Trie {
            children: vec![[None, None]], // root node at index 0
        }
    }

    // Insert a number MSB-first as a path of 31 bits.
    fn insert(&mut self, num: i32) {
        let mut node = 0;
        for i in (0..=HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;
            if self.children[node][bit].is_none() {
                let new_idx = self.children.len();
                self.children.push([None, None]);
                self.children[node][bit] = Some(new_idx);
            }
            node = self.children[node][bit].unwrap();
        }
    }

    // For `num`, greedily prefer the opposite bit at each level to maximize XOR.
    fn max_xor_with(&self, num: i32) -> i32 {
        let mut node = 0;
        let mut xor_val = 0;
        for i in (0..=HIGH_BIT).rev() {
            let bit = ((num >> i) & 1) as usize;
            let want = bit ^ 1; // we want the opposite bit
            if let Some(next) = self.children[node][want] {
                xor_val |= 1 << i; // this bit of XOR is 1
                node = next;
            } else {
                node = self.children[node][bit].unwrap(); // forced to take the same bit
            }
        }
        xor_val
    }
}

fn find_maximum_xor(nums: &[i32]) -> i32 {
    let mut trie = Trie::new();
    for &num in nums {
        trie.insert(num);
    }

    let mut best = 0;
    for &num in nums {
        best = best.max(trie.max_xor_with(num));
    }
    best
}
```

**Why MSB first?** XOR's value is dominated by its highest set bit. To maximize, we must secure the highest bit as `1` if at all possible, then the next, and so on — a greedy decision that only makes sense top-down. Inserting LSB-first would make this greedy walk impossible (a classic bug; see [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)).

**Complexity:** **O(31 · N)** ≈ **O(N)** time (each insert and each query is 31 steps). Space is **O(31 · N)** for the trie in the worst case.

---

## Dry Run — nums = [3, 10, 5, 25, 2, 8]

Binary (showing low bits; high bits are 0):
```
 3 = 00011
10 = 01010
 5 = 00101
25 = 11001
 2 = 00010
 8 = 01000
```

The maximum XOR is `5 XOR 25 = 28`:
```
 5 = 00101
25 = 11001
XOR = 11100 = 28
```

**Tracing `max_xor_with(root, 5)` after all numbers are inserted** (we only show the bits where it matters; ignore the all-zero high bits):

- `5  = ...00101`. Walking MSB-first, at the bit where `25 = ...11001` diverges:
  - Top meaningful bit: `5` has `0`, we *want* `1`. `25` and `8` and `10` provide a `1` child → take it → XOR bit set. (`25` is on this branch.)
  - Next bit: `5` has `0`, want `1`; `25` has `1` here → take it → XOR bit set.
  - Next bit: `5` has `1`, want `0`; `25` has `0` here → take it → XOR bit set.
  - Lower bits resolve to `0 XOR 1` / `1 XOR 0` giving the final `11100`.
- Result: `max_xor_with(root, 5) = 28`.

Every other number's greedy walk yields a value `<= 28`, so `find_maximum_xor` returns **28**.

The greedy guarantee: at each level, if *any* inserted number offers the opposite bit, we grab it and lock in a `1` at that position — no later choice can beat a higher-position `1`.

---

## Variants & Extensions

- **Minimum XOR of a pair:** same trie, but greedily prefer the *same* bit (`want = bit`) to keep XOR bits `0`.
- **Maximum XOR with a constraint (LC 1707):** sort queries and numbers, insert numbers below the limit lazily before each query.
- **Fixed 32 vs 31 bits:** be consistent. For non-negative ints up to `10^9`, 31 bits (indices 30..0) suffice. Off-by-one between `31` and `32` is a common bug — see [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md).

---

## Recognition Signals Recap

- Pairwise XOR maximization/minimization over an array → binary trie, MSB-first.
- Bit extraction idiom: `(num >> i) & 1`.
- Greedy "take the opposite bit if available" → classic max-XOR trie walk.

Cross-reference: [Bit Manipulation · Bit Search and Trie](../6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md).

---

> **Last Updated:** 2026-06-26
