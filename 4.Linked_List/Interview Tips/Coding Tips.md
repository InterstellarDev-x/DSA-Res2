# Coding Tips — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Rust Node Definitions

```rust
// Singly Linked List
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

// Doubly Linked List (for LRU/LFU) — use index-based Vec approach
#[derive(Debug, Clone)]
pub struct Node {
    pub key: i32,
    pub val: i32,
    pub prev: Option<usize>, // index into node arena
    pub next: Option<usize>, // index into node arena
}

impl Node {
    pub fn new(key: i32, val: i32) -> Self {
        Node { key, val, prev: None, next: None }
    }
}
```

---

## The Dummy Node Idiom

Always use a dummy head when the result list's head might change:

```rust
// BAD — head can change, requires is_none() checks everywhere
let mut head: Option<Box<ListNode>> = None;
// ... complex conditional checks needed at every append

// GOOD — dummy head never changes; result is always dummy.next
// Use raw pointer for mutable tail tracking (required in Rust for this pattern)
let mut dummy = Box::new(ListNode::new(0));
let mut tail: *mut ListNode = dummy.as_mut();
unsafe {
    (*tail).next = Some(Box::new(ListNode::new(val)));
    tail = (*tail).next.as_mut().unwrap().as_mut();
}
// result: dummy.next
```

Use dummy head for: merge, partition, remove nodes, reverse sublist.

---

## Pointer Manipulation Order

When relinking pointers, always save references **before** overwriting:

```rust
// Reverse step — take ownership before relinking (with Option<Box<ListNode>>)
fn reverse_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut prev: Option<Box<ListNode>> = None;
    let mut curr = head;
    while let Some(mut node) = curr {
        let nxt = node.next.take(); // 1. save (disconnects next, sets to None)
        node.next = prev;           // 2. relink
        prev = Some(node);          // 3. advance prev
        curr = nxt;                 // 4. advance curr using saved reference
    }
    prev
}
```

The order `save → relink → advance` prevents losing nodes.

---

## Linked List Length

Count once and store — don't recount:

```rust
let mut length: usize = 0;
let mut curr = &head; // head: Option<Box<ListNode>>
while let Some(node) = curr {
    length += 1;
    curr = &node.next;
}
// Reuse length, don't traverse again
```

---

## Avoid Null Pointer in Fast Pointer

Fast pointer skips 2 nodes per step — always check both `fast` and `fast.next`:

```rust
// Using shared references for read-only traversal
let mut slow: Option<&ListNode> = head.as_deref();
let mut fast: Option<&ListNode> = head.as_deref();
while fast.is_some() && fast.unwrap().next.is_some() {
    slow = slow.unwrap().next.as_deref();
    fast = fast.unwrap().next.as_ref().and_then(|n| n.next.as_deref());
    // safe because fast.next.is_some() checked above
}
```

If you check only `fast.is_some()`, accessing `fast.next.next` without checking causes a panic.

---

## Even vs Odd Length Bias in Mid-Finding

The init of `fast` controls which mid you get for even-length lists:

```rust
// fast = head → second middle (default for most problems)
let mut slow: Option<&ListNode> = head.as_deref();
let mut fast: Option<&ListNode> = head.as_deref();

// fast = head.next → first middle (for palindrome: split after first half)
let mut slow: Option<&ListNode> = head.as_deref();
let mut fast: Option<&ListNode> = head.as_deref().and_then(|n| n.next.as_deref());
```

For `[1, 2, 3, 4]`:
- `fast = head`: slow ends at 3 (second middle)
- `fast = head.next`: slow ends at 2 (first middle)

---

## In-Place Tricks

**Delete Node without head reference (LC 237):**
```rust
// Copy next node's value, then skip next node
let next_val = node.next.as_ref().unwrap().val;
let next_next = node.next.as_mut().unwrap().next.take();
node.val = next_val;
node.next = next_next;
```

**Cycle detection phase 2 — after meeting, both pointers move 1x:**
```rust
let mut slow: Option<&ListNode> = head.as_deref(); // reset to head
// fast stays at meeting point from phase 1
while !std::ptr::eq(slow.unwrap(), fast.unwrap()) {
    slow = slow.unwrap().next.as_deref();
    fast = fast.unwrap().next.as_deref();
}
// slow == fast == cycle start node
```

---

## Recursion vs Iteration Trade-off

| | Iterative | Recursive |
|--|-----------|-----------|
| Space | O(1) | O(n) stack |
| Readability | Lower | Higher |
| Preferred in interviews | ✅ Usually | Only if asked |

Exception: tree-like problems (flatten multilevel DLL) are naturally recursive.

---

## Common Rust APIs

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

// Vec<T> as stack (LIFO)
let mut stk: Vec<i32> = Vec::new();
stk.push(x);  // push
stk.pop();    // pop (returns Option<i32>)
stk.last();   // peek (returns Option<&i32>)

// BinaryHeap with Reverse<(val, idx)> for min-heap (Merge K Sorted)
let mut pq: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
// Push (node.val, list_index) wrapped in Reverse for min-heap ordering
// pq.push(Reverse((node.val, idx)));
// Safe comparator: compare by node value, no overflow risk
```

---

## Related Files

- [Common Mistakes](./Common%20Mistakes.md)
- [Dummy Node Technique](./Dummy%20Node%20Technique.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
