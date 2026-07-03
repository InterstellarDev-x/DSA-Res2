# Microsoft — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Microsoft

---

## Problem 1: Kth Largest Element — Both Approaches

**LC 215** · Medium

**Heap approach — O(n log k):**
```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn find_kth_largest(nums: Vec<i32>, k: usize) -> i32 {
    let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
    for &n in &nums {
        min_heap.push(Reverse(n));
        if min_heap.len() > k {
            min_heap.pop();
        }
    }
    min_heap.peek().unwrap().0
}
```

**Quickselect — O(n) average:**
```rust
fn partition(nums: &mut Vec<i32>, l: usize, r: usize) -> usize {
    let pivot = nums[r];
    let mut i = l;
    for j in l..r {
        if nums[j] <= pivot {
            nums.swap(i, j);
            i += 1;
        }
    }
    nums.swap(i, r);
    i
}

fn quickselect(nums: &mut Vec<i32>, l: usize, r: usize, target: usize) -> i32 {
    if l == r {
        return nums[l];
    }
    let pivot = partition(nums, l, r);
    if pivot == target {
        nums[pivot]
    } else if pivot < target {
        quickselect(nums, pivot + 1, r, target)
    } else {
        quickselect(nums, l, pivot - 1, target)
    }
}

fn find_kth_largest(mut nums: Vec<i32>, k: usize) -> i32 {
    let n = nums.len();
    quickselect(&mut nums, 0, n - 1, n - k)
}
```

**Q: Microsoft asks — "Which approach would you use in production and why?"**
A: "It depends on context. If I can't modify the input array (immutable data), I'd use the heap approach. If memory is constrained and the array is mutable, quickselect with O(1) extra space. For guaranteed O(n log k) without O(n²) worst case risk, heap is safer. For very large n with k ≈ 1, quickselect is faster in practice."

---

## Problem 2: Merge K Sorted Lists — Microsoft Style

**LC 23** · Hard

```rust
use std::cmp::Ordering;
use std::collections::BinaryHeap;

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

// Wrapper to enable min-heap ordering by val
struct HeapNode(Box<ListNode>);

impl PartialEq for HeapNode {
    fn eq(&self, other: &Self) -> bool { self.0.val == other.0.val }
}
impl Eq for HeapNode {}
impl PartialOrd for HeapNode {
    fn partial_cmp(&self, other: &Self) -> Option<Ordering> { Some(self.cmp(other)) }
}
impl Ord for HeapNode {
    fn cmp(&self, other: &Self) -> Ordering {
        other.0.val.cmp(&self.0.val) // Reverse for min-heap — no overflow
    }
}

pub fn merge_k_lists(lists: Vec<Option<Box<ListNode>>>) -> Option<Box<ListNode>> {
    let mut heap: BinaryHeap<HeapNode> = BinaryHeap::new();
    for node in lists.into_iter().flatten() {
        heap.push(HeapNode(node));
    }

    let mut dummy = Box::new(ListNode::new(0));
    let mut tail = &mut dummy;

    while let Some(HeapNode(mut node)) = heap.pop() {
        if let Some(next) = node.next.take() {
            heap.push(HeapNode(next));
        }
        tail.next = Some(node);
        tail = tail.next.as_mut().unwrap();
    }
    dummy.next
}
```

**Microsoft edge cases:**
- `lists = []` → return `None` (heap never filled, loop never runs, `dummy.next = None`)
- `lists = [None]` → `None` filtered in initialization; returns `None`
- Single list with elements → works correctly (heap has one item, processes all)

**Q: Compare to divide-and-conquer approach.**
A: D&C: merge pairs repeatedly → O(n log k) time, O(log k) recursion stack. Heap: O(n log k) time, O(k) heap space. Both are O(n log k) but D&C uses less peak memory when k >> log k.

---

## Related Files

- [Microsoft OA](../OA-Qns/Microsoft.md)
- [Amazon Interview Problems](./Amazon.md)
- [Top K Elements](../Patterns/Top%20K%20Elements.md)
- [K-Way Merge](../Patterns/K%20Way%20Merge.md)

> **Last Updated:** 2026-06-26
