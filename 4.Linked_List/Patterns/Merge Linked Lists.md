# Merge Linked Lists

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Merge two sorted, merge K sorted, sort list, add two numbers

---

## Core Idea

Use a **dummy head** node so you never special-case the head pointer. Maintain a `tail` pointer. Always attach the smaller current node to `tail.next`, then advance `tail`.

```rust
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

let mut dummy = Box::new(ListNode::new(0));
let mut tail = &mut dummy;
// ... merge logic ...
// return dummy.next as the result head
```

---

## Template 1 — Merge Two Sorted Lists (LC 21)

```rust
pub fn merge_two_lists(
    mut l1: Option<Box<ListNode>>,
    mut l2: Option<Box<ListNode>>,
) -> Option<Box<ListNode>> {
    let mut dummy = Box::new(ListNode::new(0));
    let mut tail = &mut dummy;
    loop {
        match (l1.as_ref(), l2.as_ref()) {
            (Some(n1), Some(n2)) => {
                if n1.val <= n2.val {
                    tail.next = l1.take();
                    tail = tail.next.as_mut().unwrap();
                    l1 = tail.next.take();
                } else {
                    tail.next = l2.take();
                    tail = tail.next.as_mut().unwrap();
                    l2 = tail.next.take();
                }
            }
            _ => break,
        }
    }
    tail.next = if l1.is_some() { l1 } else { l2 }; // attach remaining
    dummy.next
}
```

**Time:** O(n + m). **Space:** O(1) — reuses existing nodes.

---

## Template 2 — Merge K Sorted Lists (LC 23)

**Approach 1: Min-Heap (best for large k)**

```rust
use std::cmp::Ordering;
use std::collections::BinaryHeap;

// Wrapper to enable min-heap ordering on ListNode
struct NodeWrapper(Box<ListNode>);

impl Eq for NodeWrapper {}
impl PartialEq for NodeWrapper {
    fn eq(&self, other: &Self) -> bool { self.0.val == other.0.val }
}
impl Ord for NodeWrapper {
    fn cmp(&self, other: &Self) -> Ordering {
        other.0.val.cmp(&self.0.val) // reversed for min-heap
    }
}
impl PartialOrd for NodeWrapper {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> { Some(self.cmp(other)) }
}

pub fn merge_k_lists(lists: Vec<Option<Box<ListNode>>>) -> Option<Box<ListNode>> {
    let mut heap = BinaryHeap::new();
    for node in lists.into_iter().flatten() {
        heap.push(NodeWrapper(node));
    }
    let mut dummy = Box::new(ListNode::new(0));
    let mut tail = &mut dummy;
    while let Some(NodeWrapper(mut node)) = heap.pop() {
        if let Some(next) = node.next.take() {
            heap.push(NodeWrapper(next));
        }
        tail.next = Some(node);
        tail = tail.next.as_mut().unwrap();
    }
    dummy.next
}
```

**Time:** O(N log k) where N = total nodes, k = number of lists. **Space:** O(k) for the heap.

**Approach 2: Divide & Conquer (better constant factor)**

```rust
fn merge_range(lists: &mut Vec<Option<Box<ListNode>>>, lo: usize, hi: usize) -> Option<Box<ListNode>> {
    if lo == hi {
        return lists[lo].take();
    }
    let mid = lo + (hi - lo) / 2;
    let left = merge_range(lists, lo, mid);
    let right = merge_range(lists, mid + 1, hi);
    merge_two_lists(left, right)
}

pub fn merge_k_lists(mut lists: Vec<Option<Box<ListNode>>>) -> Option<Box<ListNode>> {
    if lists.is_empty() { return None; }
    let hi = lists.len() - 1;
    merge_range(&mut lists, 0, hi)
}
```

**Time:** O(N log k). **Space:** O(log k) call stack.

---

## Template 3 — Sort List (LC 148)

Merge sort on a linked list: O(n log n) time, O(log n) space (call stack only, no extra array).

```rust
// Split: first half gets ceil(len/2) nodes (matches the fast=head->next slow/fast variant)
fn split_at_mid(mut head: Option<Box<ListNode>>) -> (Option<Box<ListNode>>, Option<Box<ListNode>>) {
    let mut len = 0usize;
    { let mut curr = &head; while let Some(node) = curr { len += 1; curr = &node.next; } }
    if len <= 1 { return (head, None); }
    let split_steps = (len - 1) / 2; // advance this many times to land on last node of first half
    let mut curr = &mut head;
    for _ in 0..split_steps {
        curr = &mut curr.as_mut().unwrap().next;
    }
    let right_head = curr.as_mut().unwrap().next.take(); // cut the list
    (head, right_head)
}

pub fn sort_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    if head.is_none() || head.as_ref().unwrap().next.is_none() {
        return head;
    }
    // Split at middle
    let (left_head, right_head) = split_at_mid(head);
    let left = sort_list(left_head);
    let right = sort_list(right_head);
    merge_two_lists(left, right)
}
```

**Bottom-up iterative version (O(1) space):**

```rust
// Take up to n nodes from head; return (taken_segment, rest)
fn split_n(mut head: Option<Box<ListNode>>, n: usize) -> (Option<Box<ListNode>>, Option<Box<ListNode>>) {
    if n == 0 { return (None, head); }
    // Count how many nodes are actually available (up to n)
    let actual = {
        let mut count = 0usize;
        let mut curr = &head;
        while count < n {
            match curr { Some(node) => { count += 1; curr = &node.next; } None => break, }
        }
        count
    };
    if actual == 0 { return (None, None); }
    // Advance (actual - 1) steps to reach the last node of the segment
    let mut curr = &mut head;
    for _ in 0..actual - 1 {
        curr = &mut curr.as_mut().unwrap().next;
    }
    let rest = curr.as_mut().unwrap().next.take();
    (head, rest)
}

pub fn sort_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut length = 0usize;
    { let mut curr = &head; while let Some(n) = curr { length += 1; curr = &n.next; } }
    if length <= 1 { return head; }

    let mut dummy = Box::new(ListNode::new(0));
    dummy.next = head;

    let mut size = 1usize;
    while size < length {
        // Raw pointer for tail: avoids borrow checker conflicts when appending merged segments
        let mut tail: *mut ListNode = &mut *dummy;
        let mut curr = dummy.next.take();

        while curr.is_some() {
            let (left, rest) = split_n(curr, size);
            let (right, next_curr) = split_n(rest, size);
            curr = next_curr;
            // Inline merge of left and right, appending each node to tail
            let mut l1 = left;
            let mut l2 = right;
            unsafe {
                while l1.is_some() || l2.is_some() {
                    let pick_l1 = match (l1.as_ref(), l2.as_ref()) {
                        (Some(a), Some(b)) => a.val <= b.val,
                        (Some(_), None) => true,
                        _ => false,
                    };
                    let src = if pick_l1 { &mut l1 } else { &mut l2 };
                    let mut node = src.take().unwrap();
                    *src = node.next.take();
                    (*tail).next = Some(node);
                    tail = &mut *(*tail).next.as_mut().unwrap();
                }
            }
        }
        size <<= 1;
    }
    dummy.next
}
```

---

## Template 4 — Add Two Numbers (LC 2)

Numbers stored in **forward order** (digits head to tail = least to most significant).

```rust
pub fn add_two_numbers(
    mut l1: Option<Box<ListNode>>,
    mut l2: Option<Box<ListNode>>,
) -> Option<Box<ListNode>> {
    let mut dummy = Box::new(ListNode::new(0));
    let mut tail = &mut dummy;
    let mut carry = 0;
    while l1.is_some() || l2.is_some() || carry != 0 {
        let mut sum = carry;
        if let Some(node) = l1.take() { sum += node.val; l1 = node.next; }
        if let Some(node) = l2.take() { sum += node.val; l2 = node.next; }
        carry = sum / 10;
        tail.next = Some(Box::new(ListNode::new(sum % 10)));
        tail = tail.next.as_mut().unwrap();
    }
    dummy.next
}
```

**Add Two Numbers II (LC 445 — digits in reverse order):** Push both lists onto stacks (Vec), then pop + add with carry.

```rust
pub fn add_two_numbers(
    mut l1: Option<Box<ListNode>>,
    mut l2: Option<Box<ListNode>>,
) -> Option<Box<ListNode>> {
    let mut s1: Vec<i32> = Vec::new();
    let mut s2: Vec<i32> = Vec::new();
    while let Some(node) = l1 { s1.push(node.val); l1 = node.next; }
    while let Some(node) = l2 { s2.push(node.val); l2 = node.next; }
    let mut carry = 0;
    let mut head: Option<Box<ListNode>> = None;
    while !s1.is_empty() || !s2.is_empty() || carry != 0 {
        let mut sum = carry;
        if let Some(v) = s1.pop() { sum += v; }
        if let Some(v) = s2.pop() { sum += v; }
        carry = sum / 10;
        let mut node = Box::new(ListNode::new(sum % 10));
        node.next = head; // prepend (building in reverse)
        head = Some(node);
    }
    head
}
```

---

## Template 5 — Reorder List (LC 143)

L0 → L1 → L2 → ... → Ln  becomes  L0 → Ln → L1 → Ln-1 → L2 → ...

```rust
fn reverse_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut prev = None;
    let mut curr = head;
    while let Some(mut node) = curr {
        curr = node.next.take();
        node.next = prev;
        prev = Some(node);
    }
    prev
}

pub fn reorder_list(head: &mut Option<Box<ListNode>>) {
    if head.is_none() || head.as_ref().unwrap().next.is_none() {
        return;
    }
    // 1. Find mid (first half gets ceil(len/2) nodes)
    let (first_half, second_half) = split_at_mid(head.take());

    // 2. Reverse second half
    let mut second = reverse_list(second_half);

    // 3. Interleave merge: take from first, then second, alternating
    let mut first = first_half;
    let mut result_dummy = Box::new(ListNode::new(0));
    let mut tail = &mut result_dummy;
    while second.is_some() {
        if let Some(mut n) = first.take() {
            first = n.next.take();
            tail.next = Some(n);
            tail = tail.next.as_mut().unwrap();
        }
        if let Some(mut n) = second.take() {
            second = n.next.take();
            tail.next = Some(n);
            tail = tail.next.as_mut().unwrap();
        }
    }
    tail.next = first; // attach any remaining first-half node (for odd-length lists)
    *head = result_dummy.next;
}
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Not initializing `tail = &mut dummy` — dereferencing null | Set `let mut tail = &mut dummy` immediately |
| In Add Two Numbers: forgetting final carry | Loop condition is `l1.is_some() \|\| l2.is_some() \|\| carry != 0` |
| Heap comparator subtraction can overflow for large ints | Use `.cmp()` comparison in `Ord` impl instead of subtraction |
| Sort list: not cutting the list at mid | `curr.as_mut().unwrap().next.take()` before recursive calls |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Merge two sorted | O(n + m) | O(1) |
| Merge K sorted (heap) | O(N log k) | O(k) |
| Merge K sorted (D&C) | O(N log k) | O(log k) |
| Sort list (top-down) | O(n log n) | O(log n) |
| Sort list (bottom-up) | O(n log n) | O(1) |
| Add two numbers | O(max(n, m)) | O(max(n, m)) |

---

## Related Patterns

- [Reverse Linked List](./Reverse%20Linked%20List.md) — used in reorder list
- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — split list for merge sort
- [Heaps](../../9.Heaps/README.md) — min-heap for k-way merge

---

**Back:** [Linked List README](../README.md) | **Prev:** [Reverse Linked List](./Reverse%20Linked%20List.md) | **Next:** [Cycle Detection](./Cycle%20Detection.md)

> **Last Updated:** 2026-06-26
