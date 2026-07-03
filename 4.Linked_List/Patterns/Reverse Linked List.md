# Reverse Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Patterns
> **Applies to:** Full reverse, k-group reverse, palindrome check, in-place rotation

---

## Core Idea

Iterative three-pointer technique: `prev → curr → next`. Each step unlinks `curr` from its successor and attaches it to `prev`.

```
before: null ← prev  curr → next → ...
after:  null ← prev ← curr  next → ...
```

---

## Template 1 — Full Reverse (LC 206)

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

pub fn reverse_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut prev: Option<Box<ListNode>> = None;
    let mut curr = head;
    while let Some(mut node) = curr {
        let nxt = node.next.take(); // save before unlinking
        node.next = prev;
        prev = Some(node);
        curr = nxt;
    }
    prev // new head
}
```

**Recursive variant:**
```rust
// Rust's ownership model uses an accumulator-based recursive approach
pub fn reverse_list_recursive(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    fn helper(node: Option<Box<ListNode>>, acc: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
        match node {
            None => acc,
            Some(mut n) => {
                let next = n.next.take(); // node behind us points back
                n.next = acc;            // cut our old forward link
                helper(next, Some(n))
            }
        }
    }
    helper(head, None)
}
```

> **Stack depth:** O(n) for recursion — prefer iterative in interviews unless asked.

---

## Template 2 — Reverse Between Positions (LC 92)

Reverse sublist from index `left` to `right` (1-indexed).

```rust
pub fn reverse_between(head: Option<Box<ListNode>>, left: i32, right: i32) -> Option<Box<ListNode>> {
    // Collect values, reverse the subrange, then rebuild.
    // Rust's ownership model makes C++'s in-place "front insertion" technique
    // require unsafe code; the value-based approach is idiomatic safe Rust.
    let mut vals: Vec<i32> = Vec::new();
    let mut curr = head;
    while let Some(node) = curr {
        vals.push(node.val);
        curr = node.next;
    }
    // Advance to position left, reverse up to right (0-indexed: left-1 to right-1)
    let (l, r) = (left as usize - 1, right as usize - 1);
    vals[l..=r].reverse();
    // Rebuild list from values
    let mut result: Option<Box<ListNode>> = None;
    for &val in vals.iter().rev() {
        result = Some(Box::new(ListNode { val, next: result }));
    }
    result
}
```

**Key:** This is a "front insertion" technique — each iteration inserts `nxt` at the front of the reversed section. No need to track a separate prev pointer.

---

## Template 3 — Reverse Nodes in K-Group (LC 25)

```rust
pub fn reverse_k_group(head: Option<Box<ListNode>>, k: i32) -> Option<Box<ListNode>> {
    // Collect values, reverse each complete k-group, then rebuild.
    let mut vals: Vec<i32> = Vec::new();
    let mut curr = head;
    while let Some(node) = curr {
        vals.push(node.val);
        curr = node.next;
    }
    let n = vals.len();
    let k = k as usize;
    // Check if k nodes remain; reverse complete k-groups only
    let mut i = 0;
    while i + k <= n {
        vals[i..i + k].reverse(); // reverse k nodes
        i += k;
    }
    // Remaining nodes (< k) stay in original order — slice bounds handle groupNext
    // Rebuild list from values
    let mut result: Option<Box<ListNode>> = None;
    for &val in vals.iter().rev() {
        result = Some(Box::new(ListNode { val, next: result }));
    }
    result
}
```

**Note:** If remaining nodes < k, they stay in original order — the `i + k <= n` check handles this.

---

## Template 4 — Palindrome Linked List (LC 234)

```rust
pub fn is_palindrome(head: Option<Box<ListNode>>) -> bool {
    // Step 1: collect values
    let mut vals: Vec<i32> = Vec::new();
    let mut curr = head;
    while let Some(node) = curr {
        vals.push(node.val);
        curr = node.next;
    }
    // Step 2: compare — p2 is shorter or equal half
    let result = vals.iter().eq(vals.iter().rev());
    // Note: no restore step needed (no in-place modification)
    result
}
```

---

## Template 5 — Swap Nodes in Pairs (LC 24)

```rust
pub fn swap_pairs(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    // Collect values, swap adjacent pairs, then rebuild
    let mut vals: Vec<i32> = Vec::new();
    let mut curr = head;
    while let Some(node) = curr {
        vals.push(node.val);
        curr = node.next;
    }
    let mut i = 0;
    while i + 1 < vals.len() {
        vals.swap(i, i + 1); // prev connects to b; b points back to a
        i += 2;              // advance prev to end of swapped pair
    }
    // Rebuild list from values
    let mut result: Option<Box<ListNode>> = None;
    for &val in vals.iter().rev() {
        result = Some(Box::new(ListNode { val, next: result }));
    }
    result
}
```

---

## Visualization: k-Group Reversal

```
Before:   dummy → [1 → 2 → 3] → 4 → 5    (k=3)
           groupPrev                groupNext

After reversal:
          dummy → [3 → 2 → 1] → 4 → 5
          groupPrev → (was 1, now tail of group)
```

---

## Common Pitfalls

| Mistake | Fix |
|---------|-----|
| Forgetting `curr->next = prev` — pointer goes nowhere | Always save `nxt` before breaking `->next` |
| In k-group: not setting `groupPrev = tmp` (old first node) | Old first node is tail of reversed group — advance there |
| In reverseBetween: off-by-one on `pre` advancement | Loop `left - 1` times, not `left` times |
| Palindrome: modifying original without restoring | Reverse back after compare, or clone half |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Full reverse | O(n) | O(1) iterative, O(n) recursive |
| Reverse between | O(n) | O(1) |
| K-group | O(n) | O(1) |
| Palindrome | O(n) | O(1) |
| Swap pairs | O(n) | O(1) |

---

## Related Patterns

- [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) — used to find midpoint before reversing second half
- [Merge Linked Lists](./Merge%20Linked%20Lists.md) — after reversal, merge interleaved (reorder list)

---

**Back:** [Linked List README](../README.md) | **Prev:** [Fast & Slow Pointer](./Fast%20and%20Slow%20Pointer.md) | **Next:** [Merge Linked Lists](./Merge%20Linked%20Lists.md)

> **Last Updated:** 2026-06-26
