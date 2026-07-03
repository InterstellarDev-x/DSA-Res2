# Two Pointers on Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Nth from end, intersection, rotate, partition

---

## Core Idea

Unlike arrays, linked lists have no random access — so "two pointers" on a LL means:

1. **Gap pointer:** Advance one pointer N steps ahead, then move both at 1x until the leader hits `None`. Follower is now N steps from end.
2. **Intersection / sync:** Two pointers start at different heads. When each exhausts its list, it restarts from the **other** head. They meet at intersection (or both hit `None` if none).

---

## Template 1 — Remove Nth Node from End (LC 19)

```rust
#[derive(Debug)]
pub struct ListNode {
    pub val: i32,
    pub next: Option<Box<ListNode>>,
}

impl ListNode {
    pub fn new(val: i32) -> Self {
        ListNode { val, next: None }
    }
}

pub fn remove_nth_from_end(head: Option<Box<ListNode>>, n: i32) -> Option<Box<ListNode>> {
    let mut dummy = Box::new(ListNode { val: 0, next: head });
    // Use raw pointers to hold two simultaneous references into the list
    let ptr: *mut ListNode = dummy.as_mut();
    let mut fast: *const ListNode = ptr as *const ListNode;
    let mut slow: *mut ListNode = ptr;

    // Advance fast n+1 steps (so slow lands one BEFORE target)
    for _ in 0..=n {
        // SAFETY: valid n guarantees fast stays within list bounds
        unsafe { fast = (*fast).next.as_deref().map_or(std::ptr::null(), |node| node as *const _); }
    }

    while !fast.is_null() {
        // SAFETY: both pointers remain within the list
        unsafe {
            slow = (*slow).next.as_mut().unwrap().as_mut();
            fast = (*fast).next.as_deref().map_or(std::ptr::null(), |node| node as *const _);
        }
    }

    // SAFETY: slow points to node before target, which exists
    unsafe {
        let next_next = (*slow).next.as_mut().unwrap().next.take(); // delete target
        (*slow).next = next_next;
    }
    dummy.next
}
```

**Why n+1 steps:** We want `slow` to be the node before the target. Advancing `fast` one extra step creates the needed gap.

---

## Template 2 — Intersection of Two Linked Lists (LC 160)

```rust
// Note: True node intersection requires shared ownership.
// This demonstrates the algorithm using raw pointers (as used in LeetCode Rust).

pub fn get_intersection_node(
    head_a: *mut ListNode,
    head_b: *mut ListNode,
) -> *mut ListNode {
    let mut a = head_a;
    let mut b = head_b;
    while a != b {
        unsafe {
            a = if !a.is_null() {
                (*a).next.as_mut().map_or(std::ptr::null_mut(), |n| n.as_mut())
            } else {
                head_b
            };
            b = if !b.is_null() {
                (*b).next.as_mut().map_or(std::ptr::null_mut(), |n| n.as_mut())
            } else {
                head_a
            };
        }
    }
    a // null if no intersection (both reach null simultaneously)
}
```

**Why it works:** If list A has length `la` and list B has length `lb`, pointer `a` traverses `la + lb` nodes (its own list then B) and so does pointer `b`. They synchronize at the intersection node (or at `null` if no intersection).

**Important:** Check `a != b` with pointer equality, not value equality.

---

## Template 3 — Rotate List (LC 61)

```rust
pub fn rotate_right(mut head: Option<Box<ListNode>>, k: i32) -> Option<Box<ListNode>> {
    if head.is_none() || k == 0 { return head; }

    // Find length and tail
    let mut length = 1usize;
    let ptr: *mut ListNode = head.as_mut().unwrap().as_mut();
    let mut tail: *mut ListNode = ptr;

    unsafe {
        while (*tail).next.is_some() {
            tail = (*tail).next.as_mut().unwrap().as_mut();
            length += 1;
        }
    }

    let k = (k as usize) % length;
    if k == 0 { return head; }

    // Find new tail (length - k - 1 steps from head)
    let mut new_tail: *mut ListNode = ptr;
    unsafe {
        for _ in 0..length - k - 1 {
            new_tail = (*new_tail).next.as_mut().unwrap().as_mut();
        }

        let new_head = (*new_tail).next.take(); // detach new_head from new_tail
        (*tail).next = head; // connect old tail to old head
        new_head
    }
}
```

**Key:** Always compute `k % length` first — rotating by `length` is a no-op.

---

## Template 4 — Partition List (LC 86)

Partition list around value `x`: all nodes < x before nodes >= x, preserving relative order.

```rust
pub fn partition(mut head: Option<Box<ListNode>>, x: i32) -> Option<Box<ListNode>> {
    let mut less_dummy = Box::new(ListNode::new(0));
    let mut greater_dummy = Box::new(ListNode::new(0));
    let mut less: *mut ListNode = less_dummy.as_mut();
    let mut greater: *mut ListNode = greater_dummy.as_mut();

    while let Some(mut node) = head {
        head = node.next.take(); // detach node from list
        if node.val < x {
            unsafe {
                (*less).next = Some(node);
                less = (*less).next.as_mut().unwrap().as_mut();
            }
        } else {
            unsafe {
                (*greater).next = Some(node);
                greater = (*greater).next.as_mut().unwrap().as_mut();
            }
        }
    }

    unsafe { (*greater).next = None; } // CRITICAL: terminate — avoids a cycle if original tail < x
    unsafe { (*less).next = greater_dummy.next.take(); }
    less_dummy.next
}
```

**Critical:** Set `(*greater).next = None` before connecting. If the last node was already in `less`, `(*greater).next` still points into the original list, creating a cycle.

---

## Template 5 — Swapping Nodes (LC 1721)

Swap the kth node from beginning and kth from end (by value, not relinking):

```rust
pub fn swap_nodes(mut head: Option<Box<ListNode>>, k: i32) -> Option<Box<ListNode>> {
    let ptr: *mut ListNode = head.as_mut().unwrap().as_mut();
    let mut curr: *mut ListNode = ptr;

    // Advance curr k-1 steps to reach kth node from beginning
    for _ in 1..k {
        unsafe { curr = (*curr).next.as_mut().unwrap().as_mut(); }
    }
    let first: *mut ListNode = curr; // kth from beginning

    // Find kth from end: advance curr to end, second follows
    let mut second: *mut ListNode = ptr;
    unsafe {
        while (*curr).next.is_some() {
            curr = (*curr).next.as_mut().unwrap().as_mut();
            second = (*second).next.as_mut().unwrap().as_mut();
        }
        // second is now kth from end; swap values
        let tmp = (*first).val;
        (*first).val = (*second).val;
        (*second).val = tmp;
    }
    head
}
```

---

## Template 6 — Linked List Cycle II via Two-Pointer Length

Alternative to Floyd's for cycle entry: find cycle length `c`, then use two pointers with gap `c`.

```rust
// Note: Box<ListNode> cannot represent cycles in safe Rust.
// Use an index-based arena: next[i] = Some(j) means node i's successor is node j,
// None means end-of-list. Head is at index 0.

fn get_cycle_length(next: &[Option<usize>]) -> usize {
    // Omitted: uses Floyd's slow/fast pointer — see Fast & Slow Pointer pattern
    0 // returns 0 if no cycle
}

pub fn detect_cycle(next: &[Option<usize>]) -> Option<usize> {
    let cycle_len = get_cycle_length(next);
    if cycle_len == 0 { return None; }

    let mut p1 = 0usize; // both start at head (index 0)
    let mut p2 = 0usize;

    // Advance p1 by cycle_len
    for _ in 0..cycle_len {
        p1 = next[p1]?;
    }

    while p1 != p2 {
        p1 = next[p1]?;
        p2 = next[p2]?;
    }
    Some(p1) // index of cycle entry node
}
```

---

## Pointer Advancement Cheatsheet

| Goal | Steps to advance fast pointer |
|------|------------------------------|
| Find middle | n/2 (handled by fast = 2x speed) |
| k-th from end | k steps ahead of slow |
| (k-1)-th from end (node before target) | k+1 steps ahead |
| Intersection | restart from other head when null |

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Off-by-one: slow lands on target, not before it | Advance fast `n+1` times for "node before" |
| Intersection: using `head_a` vs `head_b` for restart incorrectly | When `a` is null, redirect to `head_b` (not `head_a`) |
| Rotate: not handling `k % length == 0` | Return `head` immediately if no rotation needed |
| Partition: `(*greater).next` not set to `None` — creates cycle | Always null-terminate the greater list |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Nth from end | O(n) | O(1) |
| Intersection | O(m + n) | O(1) |
| Rotate | O(n) | O(1) |
| Partition | O(n) | O(1) |
| Swap kth nodes | O(n) | O(1) |

---

## Related Patterns

- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — different two-pointer flavor (speed-based)
- [Cycle Detection](./Cycle%20Detection.md) — Floyd's for cycle entry
- [Arrays — Two Pointers](../../1.Arrays/Patterns/Two%20Pointers.md) — array analogue

---

**Back:** [Linked List README](../README.md) | **Prev:** [Cycle Detection](./Cycle%20Detection.md)

> **Last Updated:** 2026-06-26
