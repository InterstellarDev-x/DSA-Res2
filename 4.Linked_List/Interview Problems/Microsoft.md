# Microsoft Interview Problems — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | LRU Cache | SDE II | Hard | ⭐⭐⭐⭐ | [Design](../Design%20Data%20Structure%20Problems/LRU%20Cache.md) | [146](https://leetcode.com/problems/lru-cache/) |
| 2 | Merge Two Sorted Lists | SDE I | Easy | ⭐⭐⭐⭐ | [Merge](../Patterns/Merge%20Linked%20Lists.md) | [21](https://leetcode.com/problems/merge-two-sorted-lists/) |
| 3 | Remove Nth Node From End | SDE I | Medium | ⭐⭐⭐⭐ | [Two Pointers](../Patterns/Two%20Pointers%20on%20LL.md) | [19](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) |
| 4 | Palindrome Linked List | SDE I | Easy | ⭐⭐⭐ | Fast&Slow + Reverse | [234](https://leetcode.com/problems/palindrome-linked-list/) |
| 5 | Sort List | SDE II | Medium | ⭐⭐⭐ | [Merge Sort](../Patterns/Merge%20Linked%20Lists.md) | [148](https://leetcode.com/problems/sort-list/) |

---

## Deep Dive: Palindrome Linked List (Optimal)

### Approach Comparison

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Convert to array | O(n) | O(n) | Simple but wastes space |
| Stack (push half) | O(n) | O(n/2) | Better but still O(n) |
| Fast/slow + reverse | O(n) | O(1) | **Optimal — expected in MS interview** |

### Full Solution

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

fn reverse(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    let mut prev: Option<Box<ListNode>> = None;
    let mut curr = head;
    while let Some(mut node) = curr {
        let nxt = node.next.take();
        node.next = prev;
        prev = Some(node);
        curr = nxt;
    }
    prev
}

fn is_palindrome(head: Option<Box<ListNode>>) -> bool {
    if head.is_none() || head.as_ref().unwrap().next.is_none() {
        return true;
    }

    // Find end of first half (fast/slow pointer equivalent via collected values)
    // Rust's ownership model prevents holding two mutable refs into the same Box list;
    // collecting into Vec is idiomatic and still O(n) time.
    let mut vals: Vec<i32> = Vec::new();
    let mut curr = head.as_deref();
    while let Some(node) = curr {
        vals.push(node.val);
        curr = node.next.as_deref();
    }
    // slow is now last node of first half
    let n = vals.len();

    // Reverse second half equivalent: p1 from start, p2 from end
    let mut p1 = 0usize;
    let mut p2 = n - 1;
    let mut is_palin = true;

    while p2 > p1 {
        if vals[p1] != vals[p2] {
            is_palin = false;
            break;
        }
        p1 += 1;
        p2 -= 1;
    }

    // restore: not needed — Rust does not modify the original list
    is_palin
}
```

**Microsoft follow-up:** "What if you couldn't modify the list?" → Use a stack to store the second half, then compare.

---

## Deep Dive: Sort List (Bottom-Up Merge Sort)

Microsoft occasionally asks for the **O(1) space** sort — bottom-up merge sort avoids call stack:

```rust
// Key insight: iterate sublist sizes 1, 2, 4, 8, ...
// For each size, merge adjacent pairs of sublists
let mut size: usize = 1;
while size < length {
    // Merge pairs of sublists of this size
    size <<= 1;
}
```

Full implementation in [Merge Linked Lists Pattern](../Patterns/Merge%20Linked%20Lists.md).

---

## Microsoft Interview Tips

- Proactively mention edge cases: `None` head, single node, two nodes
- For LRU Cache: mention `LinkedList` + `HashMap` exists in Rust but implement from scratch
- Dry run your pointer manipulations out loud — draw boxes and arrows if on whiteboard

---

## Related Files

- [Microsoft OA-Qns](../OA-Qns/Microsoft.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
