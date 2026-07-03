# Coding Tips — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## 1. Never Use Subtraction in Heap Comparators

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// DANGEROUS — subtraction overflows for i32::MIN/i32::MAX — avoid this!
// let bad_cmp = |a: i32, b: i32| (b - a) > 0; // overflow-prone, DON'T do this

// SAFE — BinaryHeap<T> is already a max-heap in Rust
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // max-heap (largest on top)

// For min-heap: wrap with Reverse
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
// push: min_heap.push(Reverse(val));
// peek: min_heap.peek().map(|Reverse(v)| v)
```

For `Vec<i32>` entries in the heap (common for K-way merge entries):
```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// SAFE — compare elements explicitly (no subtraction)
// Min-heap by first element: use Reverse on a tuple (value, list_idx, elem_idx)
let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
// heap.push(Reverse((val, list_index, elem_index)));
// Tuples compare lexicographically, so minimum first element rises to top
```

---

## 2. Top K Pattern — Min-Heap for K Largest, Max-Heap for K Smallest

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// k LARGEST elements → min-heap size k (root = kth largest)
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
for &n in &nums {
    min_heap.push(Reverse(n));
    if min_heap.len() > k {
        min_heap.pop();
    }
}
// min_heap.peek() = Some(Reverse(kth_largest))

// k SMALLEST elements → max-heap size k (root = kth smallest)
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // default max-heap
for &n in &nums {
    max_heap.push(n);
    if max_heap.len() > k {
        max_heap.pop();
    }
}
// max_heap.peek() = Some(kth_smallest)
```

Mnemonic: "Opposite heap, size k, root is the answer."

---

## 3. K-Way Merge — Always Store (Value, ListIndex, ElementIndex)

```rust
use std::cmp::Reverse;

// Standard K-way merge heap entry: (value, list_index, elem_index)
// Use Reverse for min-heap ordering
heap.push(Reverse((value, list_index, elem_index)));
// When popping: advance elem_index in the same list
let Reverse((val, li, ei)) = heap.pop().unwrap();
if ei + 1 < lists[li].len() {
    heap.push(Reverse((lists[li][ei + 1], li, ei + 1)));
}
```

The three-element entry is the canonical pattern — never store value alone.

---

## 4. Two Heaps — Always Rebalance After Every Operation

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// lo: max-heap (lower half), hi: min-heap (upper half)
// CORRECT — rebalance every time
fn add_num(lo: &mut BinaryHeap<i32>, hi: &mut BinaryHeap<Reverse<i32>>, num: i32) {
    if lo.is_empty() || num <= *lo.peek().unwrap() {
        lo.push(num);
    } else {
        hi.push(Reverse(num));
    }
    // ALWAYS rebalance
    if lo.len() > hi.len() + 1 {
        let top = lo.pop().unwrap();
        hi.push(Reverse(top));
    } else if hi.len() > lo.len() {
        let Reverse(top) = hi.pop().unwrap();
        lo.push(top);
    }
}
```

It's easy to forget the rebalance in a time-pressured interview. Write it as a `rebalance()` helper function to make it obvious and testable.

---

## 5. Task Scheduler Formula — Memorize the Derivation

```
Frames:  [A _ _] [A _ _] [A]
         ↑n+1↑   ↑n+1↑   ↑maxCount tasks
Number of full frames: maxFreq - 1
Total slots in frames: (maxFreq - 1) * (n + 1)
Final tail tasks: maxCount (tasks tied for max frequency)
Formula: max((maxFreq - 1) * (n + 1) + maxCount, tasks.length)
```

`tasks.length` handles the case where tasks fill all slots with no idle time needed.

---

## 6. Median Two Heaps — `lo_top / 2.0 + hi_top / 2.0` for Overflow Safety

```rust
// Potential overflow if lo.peek() and hi.peek() are both near i32::MAX
// let median = (lo.peek().unwrap() + hi_top) as f64 / 2.0; // can overflow before cast

// Safe version — cast to f64 first, then divide
let lo_top = *lo.peek().unwrap() as f64;
let Reverse(hi_raw) = *hi.peek().unwrap();
let hi_top = hi_raw as f64;
return lo_top / 2.0 + hi_top / 2.0;
```

Both are fine for typical LeetCode inputs, but the safe version is correct for all 32-bit integers.

---

## 7. Element Removal from `BinaryHeap` Is O(n) — Avoid in Loops

```rust
use std::collections::BTreeMap;

// SLOW — BinaryHeap has no direct remove(x); a workaround is O(n)
// Avoid manual element removal inside loops

// Use BTreeMap<i32, usize> (value → count) when O(log n) removal is needed
let mut ms: BTreeMap<i32, usize> = BTreeMap::new();
// Insert one occurrence:
*ms.entry(value).or_insert(0) += 1;
// Remove one occurrence (O(log n)):
if let Some(count) = ms.get_mut(&outgoing_element) {
    *count -= 1;
    if *count == 0 {
        ms.remove(&outgoing_element);
    }
}

// Or use lazy deletion: mark elements as deleted, skip on pop
```

For sliding window median, use `BTreeMap<i32, usize>` instead of `BinaryHeap` to get O(log n) removal.

---

## 8. Reorganize String — Early Exit Check

```rust
let max_freq = *freq.iter().max().unwrap_or(&0);
if max_freq > (s.len() as i32 + 1) / 2 {
    return String::new();
}
```

Always check feasibility before attempting construction. Integer division: `(n+1)/2` correctly computes the ceiling of `n/2`.

---

## 9. Pop from Empty Heap Is Undefined Behavior — Check Before Using

```rust
// SAFE pattern — pop() returns Option<T>, no undefined behavior
if let Some(val) = heap.pop() {
    // val is guaranteed to exist
}

// Or peek first (read only)
let val = heap.peek().copied().unwrap_or(-1);
```

Unlike Java's `BinaryHeap::peek()/pop()` returns `Option<T>` in Rust — no undefined behavior, always safe to call.

---

## Quick Reference

| Goal | Heap Type | Size Constraint | Root After |
|------|----------|----------------|-----------|
| k largest | Min-heap | k | kth largest |
| k smallest | Max-heap | k | kth smallest |
| Median — lower half | Max-heap | ⌈n/2⌉ | lower median |
| Median — upper half | Min-heap | ⌊n/2⌋ | upper median |
| k-way merge | Min-heap | k | current minimum |

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Heap Patterns](./Heap%20Patterns.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
