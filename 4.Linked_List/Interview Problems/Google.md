# Google Interview Problems — Linked List

> **Topic:** [Linked List](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Reverse Nodes in K-Group | L4–L6 | Hard | ⭐⭐⭐⭐⭐ | [Reverse](../Patterns/Reverse%20Linked%20List.md) | [25](https://leetcode.com/problems/reverse-nodes-in-k-group/) |
| 2 | Merge K Sorted Lists | L4–L5 | Hard | ⭐⭐⭐⭐⭐ | [Merge + D&C](../Patterns/Merge%20Linked%20Lists.md) | [23](https://leetcode.com/problems/merge-k-sorted-lists/) |
| 3 | Find the Duplicate Number (O(1) space) | L4 | Medium | ⭐⭐⭐⭐ | [Floyd's](../Patterns/Cycle%20Detection.md) | [287](https://leetcode.com/problems/find-the-duplicate-number/) |
| 4 | LFU Cache | L5–L6 | Hard | ⭐⭐⭐ | [Design](../Design%20Data%20Structure%20Problems/LFU%20Cache.md) | [460](https://leetcode.com/problems/lfu-cache/) |
| 5 | Flatten Multilevel Doubly LL | L4 | Medium | ⭐⭐ | DFS | [430](https://leetcode.com/problems/flatten-a-multilevel-doubly-linked-list/) |

---

## Deep Dive: Reverse K-Group (Google Favorite)

### Common Follow-ups at Google

**Q1: What if k > length of remaining list?**
Leave them in original order. The check `getKth(groupPrev, k) == None` handles this.

**Q2: Can you do it recursively?**

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

fn reverse_k_group(head: Option<Box<ListNode>>, k: i32) -> Option<Box<ListNode>> {
    // Check if k nodes exist
    let mut node = &head;
    for _ in 0..k {
        match node {
            None => return head, // < k nodes, no reversal
            Some(n) => node = &n.next,
        }
    }

    // Collect k nodes, detaching them from the rest
    let mut curr = head;
    let mut segment: Vec<Box<ListNode>> = Vec::with_capacity(k as usize);
    for _ in 0..k {
        if let Some(mut boxed) = curr {
            curr = boxed.next.take();
            segment.push(boxed);
        }
    }
    // segment = [a1, a2, ..., ak], curr = rest

    // head is now tail of reversed group; connect to recursed rest
    let mut result = reverse_k_group(curr, k);

    // Build reversed chain by iterating forward through segment:
    // a1, a2, ..., ak  becomes  ak -> a(k-1) -> ... -> a1 -> result
    for mut node in segment.into_iter() {
        node.next = result;
        result = Some(node);
    }

    result // result is new head (prev in iterative version)
}
```

Space: O(n/k) call stack. Iterative is O(1) — mention the tradeoff.

**Q3: What's the time complexity?**
O(n) — every node is visited exactly twice (once to check if k remain, once to reverse).

---

## Deep Dive: Find Duplicate Number (Floyd's)

### The Array-as-Graph Insight

- Array contains n+1 integers in `[1, n]`
- Treat index as node, `nums[index]` as the "next" pointer
- `nums[0]` = "head" (index 0 always points somewhere in `[1, n]`)
- Duplicate value = two indices pointing to the same next = cycle exists

```
nums = [1, 3, 4, 2, 2]
index:  0  1  2  3  4
next:   1  3  4  2  2
Graph: 0→1→3→2→4→2 (cycle at 2)
```

```rust
fn find_duplicate(nums: &[i32]) -> i32 {
    let mut slow = nums[0] as usize;
    let mut fast = nums[nums[0] as usize] as usize;
    while slow != fast {
        slow = nums[slow] as usize;
        fast = nums[nums[fast] as usize] as usize;
    }
    slow = 0;
    while slow != fast {
        slow = nums[slow] as usize;
        fast = nums[fast] as usize;
    }
    slow as i32
}
```

**Google follow-up:** What if there are multiple duplicates? This Floyd's approach only finds one. For all duplicates, sort or use frequency map.

---

## Google Interview Framework

Google values **abstraction and generalization**:
1. State brute force first, then optimize
2. Identify the invariant (what's always true at each step of the loop)
3. Explain why your solution terminates (especially for while loops)
4. Trace through an example with 3–4 nodes minimum

---

## Related Files

- [Google OA-Qns](../OA-Qns/Google.md)
- [Linked List README](../README.md)

> **Last Updated:** 2026-06-26
