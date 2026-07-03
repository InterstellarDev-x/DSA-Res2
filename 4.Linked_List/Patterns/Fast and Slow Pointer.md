# Fast & Slow Pointer (Floyd's Tortoise and Hare)

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Cycle detection, middle-finding, k-th from end variants

---

## Core Idea

Two pointers start at `head`. Slow moves 1 step; fast moves 2 steps.

- **No cycle:** Fast hits `None`. Slow is at middle.
- **Cycle present:** They meet inside the cycle.

```
slow: head → 1 → 2 → 3  (3 steps for n=6)
fast: head → 2 → 4 → 6  (6 steps, wraps if cycle)
```

---

## Template 1 — Find Middle

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

fn middle_node(head: &Option<Box<ListNode>>) -> &ListNode {
    let mut slow: &Option<Box<ListNode>> = head;
    let mut fast: &Option<Box<ListNode>> = head;
    while fast.is_some() && fast.as_ref().unwrap().next.is_some() {
        slow = &slow.as_ref().unwrap().next;
        fast = &fast.as_ref().unwrap().next.as_ref().unwrap().next;
    }
    slow.as_ref().unwrap() // for even length, returns SECOND middle
}
```

**Variant — first middle (for palindrome check):**
```rust
// Advance fast one step before the loop to bias toward first middle
// let mut slow: &Option<Box<ListNode>> = head;
// let mut fast: &Option<Box<ListNode>> = &head.as_ref().unwrap().next;
// while fast.is_some() && fast.as_ref().unwrap().next.is_some() { ... }
// slow is now the last node of first half — split here
```

---

## Template 2 — Detect Cycle (LC 141)

> Note: Safe Rust cannot represent cyclic structures with `Box<T>` (single-ownership prevents cycles). For cycle detection we use an index-based adjacency array: `nodes[i]` is the next-node index, with `usize::MAX` marking end of list.

```rust
// nodes[i] = next node index; usize::MAX means no next (end of list)
fn has_cycle(nodes: &[usize], head: usize) -> bool {
    if nodes.is_empty() {
        return false;
    }
    let end = usize::MAX;
    let mut slow = head;
    let mut fast = head;
    while nodes[fast] != end && nodes[nodes[fast]] != end {
        slow = nodes[slow];
        fast = nodes[nodes[fast]];
        if slow == fast {
            return true;
        }
    }
    false
}
```

**Why they always meet:** If cycle length = c, when slow enters the cycle, fast is d steps ahead. Each iteration, fast gains 1 step relative to slow. They meet in at most c iterations. Time: O(n). Space: O(1).

---

## Template 3 — Cycle Entry Point (LC 142)

```rust
// nodes[i] = next node index; usize::MAX means no next (end of list)
fn detect_cycle(nodes: &[usize], head: usize) -> Option<usize> {
    if nodes.is_empty() {
        return None;
    }
    let end = usize::MAX;
    let mut slow = head;
    let mut fast = head;
    // Phase 1: find meeting point
    loop {
        if nodes[fast] == end || nodes[nodes[fast]] == end {
            return None; // no cycle
        }
        slow = nodes[slow];
        fast = nodes[nodes[fast]];
        if slow == fast {
            break;
        }
    }
    // Phase 2: find entry — reset one pointer to head
    slow = head;
    while slow != fast {
        slow = nodes[slow];
        fast = nodes[fast]; // both move at 1x now
    }
    Some(slow) // entry node index
}
```

**Mathematical proof:** Let head→entry = F, entry→meeting = a, cycle length = c.
- slow walked: F + a
- fast walked: F + a + n×c (full loops)
- fast = 2×slow → F + a + nc = 2(F + a) → F = nc - a
- From meeting point, moving fast F steps lands exactly at entry (nc − a steps back in cycle = entry).

---

## Template 4 — Happy Number (LC 202)

```rust
fn sum_of_squares(mut n: i32) -> i32 {
    let mut sum = 0;
    while n > 0 {
        let d = n % 10;
        sum += d * d;
        n /= 10;
    }
    sum
}

fn is_happy(n: i32) -> bool {
    let mut slow = n;
    let mut fast = sum_of_squares(n);
    while fast != 1 && slow != fast {
        slow = sum_of_squares(slow);
        fast = sum_of_squares(sum_of_squares(fast));
    }
    fast == 1
}
```

---

## Template 5 — Reorder List (LC 143)

Uses fast/slow to split, then reverse second half, then merge:

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

fn reorder_list(head: &mut Option<Box<ListNode>>) {
    if head.is_none() || head.as_ref().unwrap().next.is_none() {
        return;
    }

    // Step 1: find split point using fast/slow step count
    let mut steps = 0usize;
    {
        let mut slow: &Option<Box<ListNode>> = head;
        let mut fast: &Option<Box<ListNode>> = head;
        while fast.is_some() && fast.as_ref().unwrap().next.is_some() {
            steps += 1;
            slow = &slow.as_ref().unwrap().next;
            fast = &fast.as_ref().unwrap().next.as_ref().unwrap().next;
        }
        let _ = slow; // mid is reached after `steps` advances
    }

    // Step 2: advance to split point and cut
    let mut cur = head.as_mut();
    for _ in 0..steps {
        cur = cur.unwrap().next.as_mut();
    }
    let mut second = cur.unwrap().next.take(); // cut list here

    // Step 3: reverse second half
    let mut prev: Option<Box<ListNode>> = None;
    while let Some(mut node) = second {
        second = node.next.take();
        node.next = prev;
        prev = Some(node);
    }
    let mut l2 = prev;

    // Step 4: merge alternating — splice each l2 node after each l1 node
    let mut cursor: &mut Option<Box<ListNode>> = head;
    while let Some(mut s) = l2 {
        l2 = s.next.take();
        // Insert s after cursor's current node
        let rest = cursor.as_mut().unwrap().next.take();
        s.next = rest;
        cursor.as_mut().unwrap().next = Some(s);
        // Advance cursor two steps: past current node, past inserted s
        cursor = &mut cursor.as_mut().unwrap().next; // -> s
        cursor = &mut cursor.as_mut().unwrap().next; // -> original next
    }
}
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| `while fast.is_some()` only — misses single-node lists | Always check `fast.is_some() && fast.as_ref().unwrap().next.is_some()` |
| Not breaking after meeting in cycle detection | Break when `slow == fast`, then run phase 2 |
| Resetting only `slow` in phase 2 — fast stays at meeting | Reset `slow = head`; fast stays at meeting point, both move 1x |
| Even-length middle bias: first vs second | Use `fast = &head.as_ref().unwrap().next` init for "first" middle |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Find middle | O(n) | O(1) |
| Detect cycle | O(n) | O(1) |
| Find cycle entry | O(n) | O(1) |
| Reorder list | O(n) | O(1) |

---

## Related Patterns

- [Reverse Linked List](./Reverse%20Linked%20List.md) — used after finding middle in palindrome/reorder
- [Cycle Detection](./Cycle%20Detection.md) — deep dive on Floyd's with duplicate-number variant
- [Two Pointers on LL](./Two%20Pointers%20on%20LL.md) — nth-from-end gap trick

---

**Back:** [Linked List README](../README.md) | **Next Pattern:** [Reverse Linked List](./Reverse%20Linked%20List.md)

> **Last Updated:** 2026-06-26
