# Bit Search & Binary Trie

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Maximum XOR of Two Numbers, XOR queries on array, Prefix XOR with Trie

---

## Core Idea

A **Binary Trie** stores integers bit by bit from the most significant bit (MSB) down. Each node has two children: 0 and 1. Inserting n into the trie takes O(32) time. Querying for the maximum XOR partner of a number x takes O(32) time — at each level, try the opposite bit of x's current bit to maximize XOR.

---

## Binary Trie Structure

```rust
// Index-based Binary Trie (avoids Rc<RefCell<>> complexity)
#[derive(Debug, Clone)]
struct TrieNode {
    children: [Option<usize>; 2], // children[0] = '0' bit, children[1] = '1' bit
}

struct Trie {
    nodes: Vec<TrieNode>,
}

impl Trie {
    fn new() -> Self {
        Trie {
            nodes: vec![TrieNode { children: [None, None] }],
        }
    }
}
```

**Insert n into trie (MSB first, 32 bits):**
```rust
// Insert n into trie (MSB first, 32 bits) — method on Trie
impl Trie {
    fn insert(&mut self, n: i32) {
        let mut curr = 0usize;
        for bit in (0..32).rev() {
            let b = ((n >> bit) & 1) as usize;
            if self.nodes[curr].children[b].is_none() {
                self.nodes.push(TrieNode { children: [None, None] });
                let new_idx = self.nodes.len() - 1;
                self.nodes[curr].children[b] = Some(new_idx);
            }
            curr = self.nodes[curr].children[b].unwrap();
        }
    }
}
```

**Query: find number in trie that maximizes XOR with x:**
```rust
// Query: find number in trie that maximizes XOR with x — method on Trie
impl Trie {
    fn max_xor(&self, x: i32) -> i32 {
        let mut curr = 0usize;
        let mut result = 0i32;
        for bit in (0..32).rev() {
            let b = ((x >> bit) & 1) as usize;
            let want = 1 - b; // prefer opposite bit to maximize XOR
            if self.nodes[curr].children[want].is_some() {
                result |= 1 << bit; // got the XOR bit = 1
                curr = self.nodes[curr].children[want].unwrap();
            } else {
                curr = self.nodes[curr].children[b].unwrap(); // forced to take same bit, XOR = 0
            }
        }
        result
    }
}
```

---

## Template 1 — Maximum XOR of Two Numbers (LC 421)

```rust
fn find_maximum_xor(nums: &[i32]) -> i32 {
    let mut trie = Trie::new();
    for &n in nums {
        trie.insert(n);
    }

    let mut max_val = 0i32;
    for &n in nums {
        max_val = max_val.max(trie.max_xor(n));
    }
    max_val
}
```

**Time:** O(32n) = O(n). **Space:** O(32n) = O(n) nodes.

---

## Template 2 — Greedy Bit-by-Bit (no Trie, using HashSet)

For Maximum XOR, build the answer greedily from MSB to LSB using a `HashSet` of prefixes:

```rust
use std::collections::HashSet;

fn find_maximum_xor(nums: &[i32]) -> i32 {
    let mut max_val = 0i32;
    let mut mask = 0i32;
    for bit in (0..32).rev() {
        mask |= 1 << bit;
        let prefixes: HashSet<i32> = nums.iter().map(|&n| n & mask).collect(); // prefix up to current bit

        let candidate = max_val | (1 << bit); // try setting this bit in the answer
        for &prefix in &prefixes {
            // If candidate ^ prefix is also a prefix, then two numbers XOR to candidate
            if prefixes.contains(&(candidate ^ prefix)) {
                max_val = candidate;
                break;
            }
        }
    }
    max_val
}
```

**Time:** O(32n). **Space:** O(n) for `HashSet`.

---

## Template 3 — Maximum XOR with a Number in Range (LC 1707)

Queries: for query `[x, m]`, find max XOR of x with any element ≤ m. Sort queries by m; process in order, inserting elements into trie as their value ≤ current m.

```rust
fn maximize_xor(nums: &mut Vec<i32>, queries: &Vec<Vec<i32>>) -> Vec<i32> {
    nums.sort();
    let n = queries.len();
    let mut idx: Vec<usize> = (0..n).collect();
    idx.sort_by(|&a, &b| queries[a][1].cmp(&queries[b][1])); // sort by m

    let mut result = vec![0i32; n];
    let mut trie = Trie::new();
    let mut j = 0usize;

    for i in idx {
        let x = queries[i][0];
        let m = queries[i][1];
        while j < nums.len() && nums[j] <= m {
            trie.insert(nums[j]);
            j += 1;
        }
        result[i] = if j == 0 { -1 } else { trie.max_xor(x) };
    }
    result
}
```

---

## Template 4 — Count Pairs with XOR in Range

Count pairs `(i, j)` with `i < j` such that `l ≤ nums[i] XOR nums[j] ≤ r`.

```rust
fn count_pairs(nums: &Vec<i32>, low: i32, high: i32) -> i32 {
    count_at_most(nums, high) - count_at_most(nums, low - 1)
}

// Count pairs with XOR <= limit
fn count_at_most(nums: &Vec<i32>, limit: i32) -> i32 {
    let mut trie = Trie::new();
    let mut count = 0i32;
    for &n in nums {
        count += query_at_most(&trie, n, limit);
        trie.insert(n);
    }
    count
}
```

---

## Why MSB-First?

For maximizing XOR, the highest bit contributes `2^31` vs `2^0` — greedy from MSB is correct. A set bit at position 31 is worth more than all lower 31 bits combined.

---

## Trie vs HashSet Approach

| Approach | Time | Space | When to Use |
|----------|------|-------|-------------|
| Binary Trie | O(32n) | O(32n) | When querying multiple times or with range constraints |
| HashSet prefix | O(32n) | O(n) | Single maximum XOR with no range constraint |
| Brute force | O(n²) | O(1) | n ≤ 1000 |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Max XOR Two Numbers (Trie) | O(n) | O(n) |
| Max XOR Two Numbers (HashSet) | O(n) | O(n) |
| Max XOR with Range Query | O((n+q) log n) | O(n) |

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — bit extraction
- [XOR Tricks](./XOR%20Tricks.md) — XOR identities
- [Tries](../../15.Tries/README.md) — character-based Trie (same structure, different alphabet)

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [Bitmask DP](./Bitmask%20DP.md)

> **Last Updated:** 2026-06-26
