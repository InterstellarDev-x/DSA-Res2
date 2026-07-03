# Common Mistakes — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## 1. Fast/Slow Pointer — Wrong Loop Termination

```rust
// BUG: only checks fast
while fast.is_some() {
    slow = slow.as_ref().unwrap().next.as_deref();
    fast = fast.as_ref().unwrap().next.as_ref().unwrap().next.as_deref(); // panic if fast.next is None
}

// FIX
while fast.is_some() && fast.as_ref().unwrap().next.is_some() { /* ... */ }
```

---

## 2. Reverse — Forgetting to Save Next

```rust
// BUG: node.next is lost after relinking (Rust borrow checker helps catch this)
node.next = prev;
prev = Some(node);
// curr is now pointing at prev, not the original next!

// FIX: save before relinking using .take()
let nxt = node.next.take(); // save next as Option<Box<ListNode>>
node.next = prev;           // relink
prev = Some(node);          // advance prev
curr = nxt;                 // advance curr to saved next
```

---

## 3. K-Group Reverse — Wrong groupPrev Advancement

```rust
// BUG: advancing group_prev to kth (new head of group) instead of old head (new tail)
group_prev = kth; // wrong — skips to new head

// FIX: tmp = group_prev.next (old first = new tail of reversed group)
let tmp = group_prev.as_mut().unwrap().next.take();
group_prev.as_mut().unwrap().next = kth; // kth is new head
group_prev = tmp;                        // advance to new tail
```

---

## 4. Partition List — Not Null-Terminating Greater List

```rust
// BUG: greater.next may still point into the original list → cycle
less.next = greater_head.as_mut().unwrap().next.take();
// ... return less_head.next

// FIX: terminate before connecting (.take() sets to None automatically)
greater.next = None;                                     // MUST do this first
less.next = greater_head.as_mut().unwrap().next.take();
// ... return less_head.next
```

---

## 5. Remove Nth from End — Off-by-One on Gap

```rust
// BUG: advance fast n times → slow lands ON the target, not before it
for _ in 0..n { fast = fast.as_ref().unwrap().next.as_deref(); }

// FIX: advance n+1 times → slow lands one before target
for _ in 0..=n { fast = fast.as_ref().unwrap().next.as_deref(); }
// Also: use dummy head so removing the first node works uniformly
```

---

## 6. LRU Cache — Not Storing Key in Node

```rust
// BUG: can't evict LRU from HashMap without knowing its key
let lru = tail_prev_idx; // don't know the key!
map.remove(&???);        // unknown key

// FIX: store key in Node
struct Node { key: i32, val: i32, /* prev/next indices */ }
map.remove(&nodes[lru].key); // works
```

---

## 7. Floyd's Phase 2 — Moving Fast at 2x

```rust
// BUG: in phase 2, fast still moves 2x
slow = head;
while !std::ptr::eq(slow, fast) {
    slow = (*slow).next;
    fast = (*(*fast).next).next; // wrong — phase 2 needs 1x speed
}

// FIX: both move 1x in phase 2
while !std::ptr::eq(slow, fast) {
    slow = (*slow).next;
    fast = (*fast).next;
}
```

---

## 8. Intersecting Lists — Wrong Head Assignment on Restart

```rust
// BUG: a restarts to head_a, b restarts to head_b (no progress)
a = if a.is_some() { a.as_ref().unwrap().next.as_deref() } else { head_a };
b = if b.is_some() { b.as_ref().unwrap().next.as_deref() } else { head_b };

// FIX: each restarts to the OTHER list's head
a = if a.is_some() { a.as_ref().unwrap().next.as_deref() } else { head_b };
b = if b.is_some() { b.as_ref().unwrap().next.as_deref() } else { head_a };
```

---

## 9. Cycle Detection — Checking Before Advancing

```rust
// BUG: slow == fast is true at start (both at head) → false positive no-cycle
if std::ptr::eq(slow, fast) { return true; } // before any movement!
while fast.is_some() && fast.as_ref().unwrap().next.is_some() { /* ... */ }

// FIX: check AFTER advancing
while fast.is_some() && fast.as_ref().unwrap().next.is_some() {
    slow = slow.as_ref().unwrap().next.as_deref();
    fast = fast.as_ref().unwrap().next.as_ref().unwrap().next.as_deref();
    if std::ptr::eq(slow, fast) { return true; } // check after movement
}
```

---

## 10. Recursion Stack Overflow

Recursive reverse / recursive DFS on large inputs (n = 10^5) will cause a stack overflow.

```rust
// RISKY for large input
fn reverse_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    match head {
        None => None,
        Some(ref node) if node.next.is_none() => head,
        Some(mut node) => {
            let rest = reverse_list(node.next.take()); // O(n) stack depth
            // ...
            rest
        }
    }
}

// SAFE
fn reverse_list(mut head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut prev: Option<Box<ListNode>> = None; // iterative O(1) space
    while let Some(mut node) = head {
        head = node.next.take();
        node.next = prev;
        prev = Some(node);
    }
    prev
}
```

Default system stack size is typically 1–8 MB. A list of length 10^5 with deep recursion will overflow.

---

## 11. Heap Comparator Integer Overflow

```rust
// BUG: a.val - b.val overflows for large negative/positive pairs
// In Rust, integer overflow panics in debug mode — avoid subtraction-based comparisons

// FIX: use proper ordering with BinaryHeap<Reverse<...>> for min-heap by val
use std::cmp::Reverse;
use std::collections::BinaryHeap;

// Min-heap by val: wrap value in Reverse so smallest val has highest priority
let mut pq: BinaryHeap<(Reverse<i32>, i32)> = BinaryHeap::new();
// pq.push((Reverse(node.val), node.key));
```

---

## Quick Checklist Before Submitting

- [ ] Dummy head used where head might change?
- [ ] `fast.is_some() && fast.as_ref().unwrap().next.is_some()` (both conditions)?
- [ ] `nxt` saved before `node.next = prev` (use `.take()`)?
- [ ] Greater list `None`-terminated in partition?
- [ ] Phase 2 of Floyd's uses 1x speed?
- [ ] Key stored in LRU/LFU node?

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)

> **Last Updated:** 2026-06-26
