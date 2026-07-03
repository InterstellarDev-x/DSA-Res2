# Common Mistakes — Heaps

> **Topic:** [Heaps](../README.md) · **Section:** Interview Tips

---

## Mistake 1: Subtraction in Comparator — Integer Overflow

```rust
use std::collections::BinaryHeap;

// WRONG — overflow for i32::MIN / i32::MAX with subtraction-based comparator
// The equivalent wrong approach would be comparing with subtraction:
// |a: &i32, b: &i32| b - a  // overflow risk if a = i32::MIN, b = 1
// Example: a = i32::MIN, b = 1 → b - a = 1 - (-2147483648) overflows

// CORRECT — use explicit comparison
// BinaryHeap in Rust is a max-heap by default, uses Ord trait (no subtraction)
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // default max-heap in Rust
```

---

## Mistake 2: Wrong Heap Type for Top K

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — max-heap for "k largest": peek gives largest, not kth largest
let mut max_heap: BinaryHeap<i32> = BinaryHeap::new(); // default max-heap
for &n in &nums {
    max_heap.push(n);
    if max_heap.len() > k { max_heap.pop(); }  // removes largest — wrong!
}

// CORRECT — min-heap for "k largest": peek = kth largest (smallest of k largest)
let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
for &n in &nums {
    min_heap.push(Reverse(n));
    if min_heap.len() > k { min_heap.pop(); }  // removes smallest — keeps k largest
}
min_heap.peek().unwrap().0  // kth largest
```

---

## Mistake 3: Forgetting Rebalance in Two Heaps

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — no rebalance; sizes diverge
fn add_num_wrong(lo: &mut BinaryHeap<i32>, hi: &mut BinaryHeap<Reverse<i32>>, num: i32) {
    if lo.is_empty() || num <= *lo.peek().unwrap() { lo.push(num); }
    else { hi.push(Reverse(num)); }
    // MISSING: rebalance!
}

// CORRECT
fn add_num(lo: &mut BinaryHeap<i32>, hi: &mut BinaryHeap<Reverse<i32>>, num: i32) {
    if lo.is_empty() || num <= *lo.peek().unwrap() { lo.push(num); }
    else { hi.push(Reverse(num)); }
    if lo.len() > hi.len() + 1 { hi.push(Reverse(lo.pop().unwrap())); }
    else if hi.len() > lo.len() { lo.push(hi.pop().unwrap().0); }
}
```

Without rebalance, `find_median` returns wrong results after several inserts.

---

## Mistake 4: Median Heap — Accessing Empty `hi` When Sizes Equal

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — hi might be empty initially
fn find_median_wrong(lo: &BinaryHeap<i32>, hi: &BinaryHeap<Reverse<i32>>) -> f64 {
    if lo.len() == hi.len() {
        return (*lo.peek().unwrap() + hi.peek().unwrap().0) as f64 / 2.0; // panic if hi is empty
    }
    *lo.peek().unwrap() as f64
}

// CORRECT — guard empty
fn find_median(lo: &BinaryHeap<i32>, hi: &BinaryHeap<Reverse<i32>>) -> f64 {
    if lo.len() > hi.len() { return *lo.peek().unwrap() as f64; }
    (*lo.peek().unwrap() + hi.peek().unwrap().0) as f64 / 2.0  // lo.len == hi.len, both non-empty
}
```

After adding the first element, `lo.len()=1, hi.len()=0` — they're not equal, so this path is safe. But if the size-balance invariant breaks, calling `.peek().unwrap()` on an empty heap causes a panic.

---

## Mistake 5: K-Way Merge — Storing Value Only (Missing Index)

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — can't advance to next element in the list
heap.push(Reverse((value,)));

// CORRECT — store (value, list_index, element_index)
heap.push(Reverse((value, list_index, element_index)));
// On pop: advance element from same list
if let Some(Reverse((_, li, ei))) = heap.pop() {
    if ei + 1 < lists[li].len() {
        heap.push(Reverse((lists[li][ei + 1], li, ei + 1)));
    }
}
```

---

## Mistake 6: Merge K Lists — Using `a.val - b.val` in Comparator

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

#[derive(Debug, Clone, PartialEq)]
pub struct ListNode {
    pub val: i32,
    pub next: Option<Box<ListNode>>,
}

impl ListNode {
    pub fn new(val: i32) -> Self {
        ListNode { val, next: None }
    }
}

// WRONG — overflow risk with subtraction
// val_a - val_b can overflow for large values (e.g. i32::MIN - i32::MAX)

// CORRECT — use Reverse for min-heap ordering by val (explicit comparison, no subtraction)
// Store (val, node) so BinaryHeap orders by val ascending (min at top)
let mut heap: BinaryHeap<Reverse<(i32, Box<ListNode>)>> = BinaryHeap::new();
```

Even though `ListNode` values are usually small, the safe habit prevents bugs with large values.

---

## Mistake 7: Task Scheduler — Forgetting `tasks.len()` in `max`

```rust
// WRONG — when no idle time is needed, answer = tasks.len()
return (max_freq - 1) * (n + 1) + max_count;

// CORRECT
return ((max_freq - 1) * (n + 1) + max_count).max(tasks.len() as i32);
```

For input `tasks = [A,A,B,B,C,C]`, `n = 0`: formula = `(2-1)*(1)+2 = 3`, but `tasks.len() = 6`. The `max` ensures we never return fewer slots than actual tasks.

---

## Mistake 8: Reorganize String — Missing Feasibility Check

```rust
// WRONG — constructs invalid string without checking
// If 'a' appears 5 times in "aaabc" (n=5), no valid arrangement exists

// CORRECT — check early
let max_freq = freq.iter().copied().max().unwrap_or(0);
if max_freq > (s.len() as i32 + 1) / 2 { return String::new(); }
```

Without this check, the heap approach may produce an invalid string or loop incorrectly when one element dominates.

---

## Mistake 9: K Closest — Using `sqrt` for Distance

```rust
// WRONG — floating point imprecision; sqrt is unnecessary
let dist = ((x * x + y * y) as f64).sqrt();

// CORRECT — compare squared distances (always integer, no precision loss)
let dist_sq = x * x + y * y;
// Use in comparator: |a: &Vec<i32>, b: &Vec<i32>| (b[0]*b[0]+b[1]*b[1]).cmp(&(a[0]*a[0]+a[1]*a[1]))
```

For ranking, `sqrt` is monotonic — if `d1 < d2` then `sqrt(d1) < sqrt(d2)`. Skip the sqrt; compare squared distances directly.

---

## Mistake 10: IPO — Not Breaking When No Affordable Project

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// WRONG — always tries to pop from available, even if empty
for _ in 0..k {
    while !locked.is_empty() && locked.peek().unwrap().0 <= w {
        let item = locked.pop().unwrap();
        available.push(item);
    }
    w += available.pop().unwrap(); // panic if available is empty!
}

// CORRECT
for _ in 0..k {
    while !locked.is_empty() && locked.peek().unwrap().0 <= w {
        let item = locked.pop().unwrap();
        available.push(item);
    }
    if available.is_empty() { break; }  // no affordable projects
    w += available.pop().unwrap();
}
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Heap Patterns](./Heap%20Patterns.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
