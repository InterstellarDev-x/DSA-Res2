# Dummy Node Technique

> **Topic:** [Linked List](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Why Dummy Nodes?

Linked list problems frequently require:
1. Building a new list (merge, partition, sort)
2. Deleting the head node (remove nth, delete duplicates)
3. Inserting at the front (reverse sublist)

Without a dummy node, you write `if head.is_none()` or `if prev.is_none()` guards throughout. With a dummy node, the head is never the "first real node" — it's always a safe sentinel, and `dummy.next` is the answer.

---

## Pattern 1 — Building a New List

```rust
// pub struct ListNode { pub val: i32, pub next: Option<Box<ListNode>> }
// impl ListNode { pub fn new(val: i32) -> Self { ListNode { val, next: None } } }

let mut dummy = Box::new(ListNode::new(0));
let mut cur = &mut dummy.next;

// Append nodes
*cur = Some(some_node);
cur = &mut cur.as_mut().unwrap().next;

dummy.next // real answer
```

**Problems:** Merge Two Sorted, Merge K Sorted, Add Two Numbers, Odd Even LL, Partition List.

---

## Pattern 2 — Deleting a Node (May Delete Head)

```rust
let mut dummy = Box::new(ListNode::new(0));
dummy.next = head;
let mut prev = &mut *dummy;

// Walk until prev.next is the node to delete
let next_next = prev.next.as_mut().unwrap().next.take();
prev.next = next_next; // delete

dummy.next // head may have changed
```

**Without dummy:** If the head itself needs deletion, `prev` would be `None` — you'd need to special-case `if prev.is_none() { head = head.unwrap().next }`.

**Problems:** Remove Nth from End, Remove Duplicates (82), Remove Elements.

---

## Pattern 3 — Reverse Sublist (Protect Left Boundary)

```rust
let mut dummy = Box::new(ListNode::new(0));
dummy.next = head;
let mut pre = &mut *dummy; // node just before the reversal window

// Walk pre to position left-1
for _ in 0..left - 1 {
    pre = pre.next.as_deref_mut().unwrap();
}

// Reverse from pre.next to pre.next + (right-left) nodes
// pre.next will be head of reversed section; pre is its left anchor
```

**Without dummy:** When `left = 1`, `pre` would be `None`.

---

## Pattern 4 — Sentinel Head + Tail (DLL, LRU/LFU)

```rust
// Index-based doubly linked list (indices as "pointers")
// Sentinel head = index 0, sentinel tail = index 1
struct Node {
    val: i32,
    key: i32,
    prev: usize,
    next: usize,
}

let mut nodes: Vec<Node> = vec![
    Node { val: 0, key: 0, prev: 0, next: 1 }, // sentinel head (index 0)
    Node { val: 0, key: 0, prev: 0, next: 1 }, // sentinel tail (index 1)
];
// nodes[0].next = 1 and nodes[1].prev = 0 already set above
```

**All operations become uniform:**
```rust
// Insert at front (most recent)
fn insert_front(nodes: &mut Vec<Node>, idx: usize) {
    let head_next = nodes[0].next;
    nodes[idx].next = head_next; // never invalid (at worst, tail at index 1)
    nodes[idx].prev = 0;
    nodes[head_next].prev = idx;
    nodes[0].next = idx;
}

// Remove LRU (just before tail)
fn remove_lru(nodes: &mut Vec<Node>) -> usize {
    let lru = nodes[1].prev; // never invalid (at worst, head — but we check size)
    remove_node(nodes, lru);
    lru
}
```

**Without sentinels:** `insert_front` must check `if nodes[0].next == 1 ...`; `remove_last` must check `if nodes[1].prev == 0 ...`

---

## When NOT to Use Dummy Node

- In-place problems where you return the original head unchanged (e.g., palindrome check, cycle detection)
- When you only traverse, never modify the list structure

---

## Dummy Node Value Convention

Use `ListNode::new(0)` — the value doesn't matter since `dummy` is never part of the answer. Some coders use `-1` or `i32::MIN` to distinguish from real values during debugging.

---

## Cheat Sheet

| Scenario | Use Dummy? | Return |
|----------|-----------|--------|
| Building output list | ✅ Yes | `dummy.next` |
| Deleting node (head may change) | ✅ Yes | `dummy.next` |
| Reversing sublist | ✅ Yes | `dummy.next` |
| Just traversing | ❌ No | unchanged `head` |
| DLL with O(1) insert/remove | ✅ Yes (both ends) | `nodes[0].next` |

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
