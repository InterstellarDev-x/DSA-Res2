# Design: Dynamic Array (ArrayList)

> **Topic:** [Arrays](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Medium · **Tags:** `design` `amortized` `resizing`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Brute Force Approach](#brute-force-approach)
4. [Optimal Approach](#optimal-approach)
5. [Rust Implementation](#rust-implementation)
6. [Complexity Analysis](#complexity-analysis)
7. [Edge Cases](#edge-cases)
8. [Similar Problems](#similar-problems)
9. [Follow-up Questions](#follow-up-questions)
10. [Company Tags](#company-tags)

---

## Problem Statement

Design a dynamic array (similar to `Vec`) that supports:

- `void add(int val)` — append element; resize if needed
- `int get(int index)` — return element at index
- `void set(int index, int val)` — update element at index
- `void remove(int index)` — remove element, shift left
- `int size()` — return current number of elements

The internal array should double in capacity when full and halve when less than 25% full (to avoid thrashing).

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core concept | Amortized O(1) append via doubling strategy |
| Key insight | Why doubling (not +1) gives O(1) amortized cost |
| Must explain | Amortized analysis — total cost over n operations = O(n) |
| Bonus | Shrinking policy and why threshold = 25% (not 50%) |

---

## Brute Force Approach

Create a new array of size `n+1` every time an element is added.

- **Time:** O(n) per `add` (copy everything)
- **Space:** O(n²) total over n insertions

---

## Optimal Approach

**Doubling strategy:**
- Start with capacity `c = 1`
- When `size == capacity`: allocate new array of size `2 * capacity`, copy all elements
- When `size < capacity / 4`: shrink to `capacity / 2`

**Why doubling works (amortized analysis):**
After `n` insertions into a fresh array, total copy work = 1 + 2 + 4 + ... + n = **2n** = O(n).
So amortized cost per insertion = O(1).

---

## Rust Implementation

```rust
pub struct DynamicArray {
    data: Box<[i32]>,
    sz: usize,
    capacity: usize,
}

impl DynamicArray {
    pub fn new() -> Self {
        let capacity = 2;
        DynamicArray {
            data: vec![0; capacity].into_boxed_slice(),
            sz: 0,
            capacity,
        }
    }

    pub fn add(&mut self, val: i32) {
        if self.sz == self.capacity {
            self.resize(self.capacity * 2);
        }
        self.data[self.sz] = val;
        self.sz += 1;
    }

    pub fn get(&self, index: usize) -> i32 {
        self.check_bounds(index);
        self.data[index]
    }

    pub fn set(&mut self, index: usize, val: i32) {
        self.check_bounds(index);
        self.data[index] = val;
    }

    pub fn remove(&mut self, index: usize) {
        self.check_bounds(index);
        for i in index..self.sz - 1 {
            self.data[i] = self.data[i + 1];
        }
        self.sz -= 1;
        // Shrink if using less than 25% capacity (avoid thrash: don't shrink at 50%)
        if self.sz > 0 && self.sz == self.capacity / 4 {
            self.resize(self.capacity / 2);
        }
    }

    pub fn size(&self) -> usize {
        self.sz
    }

    fn resize(&mut self, new_capacity: usize) {
        let mut new_data = vec![0; new_capacity].into_boxed_slice();
        new_data[..self.sz].copy_from_slice(&self.data[..self.sz]);
        self.data = new_data;
        self.capacity = new_capacity;
    }

    fn check_bounds(&self, index: usize) {
        if index >= self.sz {
            panic!("Index: {}, Size: {}", index, self.sz);
        }
    }
}
```

---

## Complexity Analysis

| Operation | Time (amortized) | Space |
|-----------|-----------------|-------|
| `add` | O(1) amortized, O(n) worst | O(1) amortized |
| `get` | O(1) | O(1) |
| `set` | O(1) | O(1) |
| `remove` (middle) | O(n) | O(1) |
| `remove` (end) | O(1) | O(1) |
| Overall space | — | O(n) |

---

## Edge Cases

- `get(usize::MAX)` or `get(size)` → panics in `check_bounds`
- Remove last element → no shift needed, just `self.sz -= 1`
- Repeated remove → verify shrinking doesn't go below capacity 1
- Add after remove → reuse existing space correctly
- Capacity can never go below 1 (guard the shrink)

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Design Circular Queue](./Circular%20Queue.md) | Fixed-size circular buffer |
| [Design HashMap](./HashMap.md) | Dynamic resizing with load factor |
| [Design Stack using Array](../../7.Stacks_and_Queues/Design%20Data%20Structure%20Problems/Stack.md) | Stack over dynamic array |

---

## Follow-up Questions

1. **Why shrink at 25% and not 50%?** — 50% causes thrashing: add one element (resize up), remove one (resize down), repeat. 25% gives a hysteresis buffer.
2. **How does `Vec` differ?** — It grows by a factor (typically 2x), never shrinks automatically; use `shrink_to_fit()` explicitly.
3. **Thread safety?** — Need a `Mutex` or use atomic operations for concurrent access.
4. **Generic version?** — Use a generic struct `DynamicArray<T>` with `Box<[T]>` as the internal storage.
5. **Memory fragmentation?** — Each resize allocates a new contiguous block; the old `Box<[i32]>` is freed automatically when dropped.

---

## Company Tags

`Amazon` `Google` `Microsoft` `Adobe`

---

## Navigation

| Related |
|---------|
| [Arrays README](../README.md) |
| [Interview Tips — Coding Tips](../Interview%20Tips/Coding%20Tips.md) |

> **Last Updated:** 2026-06-26
