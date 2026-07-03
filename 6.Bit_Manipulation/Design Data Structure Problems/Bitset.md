# Bitset

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [2166 — Design Bitset](https://leetcode.com/problems/design-bitset/)

---

## Problem

Design a `Bitset` with a fixed size. Supports:
- `fix(idx)` — set bit idx to 1
- `unfix(idx)` — set bit idx to 0
- `flip()` — invert all bits
- `all()` — true if all bits are 1
- `one()` — true if at least one bit is 1
- `count()` — number of 1-bits
- `toString()` — binary string representation

All operations O(1) except `toString()` O(n).

---

## Design

Use a `Vec<i64>` array — each `i64` holds 64 bits. Key challenge: `flip()` must be O(1), not O(n).

**Lazy flip:** Maintain a `flipped` boolean. Flipping twice = no-op. On any read, XOR the actual bit with `flipped` to get the logical value. Maintain a separate `set_count` that adjusts on flip: `set_count = size - set_count`.

```rust
struct Bitset {
    words: Vec<i64>,
    size: i32,
    flipped: bool,
    set_count: i32,
}

impl Bitset {
    fn new(size: i32) -> Self {
        Bitset {
            words: vec![0i64; ((size + 63) / 64) as usize],
            size,
            flipped: false,
            set_count: 0,
        }
    }

    fn fix(&mut self, idx: i32) {
        let wi = (idx / 64) as usize;
        let bi = idx % 64;
        let is_set = ((self.words[wi] >> bi) & 1) == 1;
        // Logical value = is_set XOR flipped
        if !(is_set ^ self.flipped) { // currently logically 0
            if !self.flipped {
                self.words[wi] |= 1i64 << bi;  // set physical bit
            } else {
                self.words[wi] &= !(1i64 << bi); // physical 1 means logical 0 when flipped; clear it
            }
            self.set_count += 1;
        }
    }

    fn unfix(&mut self, idx: i32) {
        let wi = (idx / 64) as usize;
        let bi = idx % 64;
        let is_set = ((self.words[wi] >> bi) & 1) == 1;
        if is_set ^ self.flipped { // currently logically 1
            if !self.flipped {
                self.words[wi] &= !(1i64 << bi);
            } else {
                self.words[wi] |= 1i64 << bi;
            }
            self.set_count -= 1;
        }
    }

    fn flip(&mut self) {
        self.flipped = !self.flipped;
        self.set_count = self.size - self.set_count;
    }

    fn all(&self) -> bool  { self.set_count == self.size }
    fn one(&self) -> bool  { self.set_count > 0 }
    fn count(&self) -> i32 { self.set_count }

    fn to_string(&self) -> String {
        let mut result = String::with_capacity(self.size as usize);
        for i in 0..self.size {
            let wi = (i / 64) as usize;
            let bi = i % 64;
            let physical = ((self.words[wi] >> bi) & 1) as i32;
            let logical = physical ^ (if self.flipped { 1 } else { 0 });
            result.push(if logical == 1 { '1' } else { '0' });
        }
        result
    }
}
```

---

## Simpler Implementation (using Vec\<bool\>)

If a simpler approach is acceptable, use Rust's `Vec<bool>`:

```rust
struct Bitset {
    bs: Vec<bool>,
    flipped_bs: Vec<bool>,
    sz: usize,
}

impl Bitset {
    fn new(size: usize) -> Self {
        Bitset {
            bs: vec![false; size],
            flipped_bs: vec![true; size],
            sz: size,
        }
    }

    fn fix(&mut self, idx: usize) {
        self.bs[idx] = true;
        self.flipped_bs[idx] = false;
    }

    fn unfix(&mut self, idx: usize) {
        self.bs[idx] = false;
        self.flipped_bs[idx] = true;
    }

    fn flip(&mut self) {
        let tmp = self.bs.clone();
        for i in 0..self.sz {
            self.bs[i] = self.bs[i] ^ self.flipped_bs[i];
        }
        for i in 0..self.sz {
            self.flipped_bs[i] = self.flipped_bs[i] ^ tmp[i];
        }
    }

    fn all(&self) -> bool {
        self.bs.iter().all(|&b| b)
    }

    fn one(&self) -> bool {
        self.bs.iter().any(|&b| b)
    }

    fn count(&self) -> usize {
        self.bs.iter().filter(|&&b| b).count()
    }

    fn to_string(&self) -> String {
        self.bs.iter().map(|&b| if b { '1' } else { '0' }).collect()
    }
}
```

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Storage | `Vec<i64>` or `Vec<bool>` | Compact 64x space saving vs bool[] |
| `flip()` O(1) | Lazy `flipped` flag | Avoids O(n) bit inversion |
| `set_count` maintenance | Updated on fix/unfix/flip | O(1) `all()`/`one()`/`count()` |
| `to_string()` | O(n) always | Must read each bit |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `fix`, `unfix` | O(1) | — |
| `flip` | O(1) | — |
| `all`, `one`, `count` | O(1) | — |
| `to_string` | O(n) | O(n) |
| Total space | — | O(n/64) = O(n) |

---

## Related Files

- [XOR Linked List](./XOR%20Linked%20List.md)
- [Bit Basics](../Patterns/Bit%20Basics.md)

---

**Back:** [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
