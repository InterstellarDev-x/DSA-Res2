# K-Way Merge

> **Topic:** [Heaps](../README.md) · **Pattern 3 of 5**
> **Problems:** Merge K Sorted Lists · Kth Smallest in Matrix · K Pairs Smallest Sums · Sort K-Sorted Array · Smallest Range from K Lists

---

## Core Concept

**K-way merge** efficiently merges k sorted sequences or finds the kth element across k sorted sequences using a min-heap.

**Template:**
1. Initialize heap with the **first element** from each of the k lists
2. Poll the minimum element (current global minimum)
3. Push the **next element** from the same list that was just polled
4. Repeat until heap is empty or target is found

Each element enters and exits the heap at most once → **O(n log k)** total, where n = total elements.

---

## Problem 1: Merge K Sorted Lists — LC 23

```rust
use std::cmp::Reverse;
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

fn merge_k_lists(mut lists: Vec<Option<Box<ListNode>>>) -> Option<Box<ListNode>> {
    // Min-heap: (value, list_index)
    let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();

    // Initialize: push head of each non-empty list
    for (i, list) in lists.iter().enumerate() {
        if let Some(node) = list {
            heap.push(Reverse((node.val, i)));
        }
    }

    let mut sorted_vals: Vec<i32> = Vec::new();

    while let Some(Reverse((val, idx))) = heap.pop() {
        sorted_vals.push(val);
        // Advance the list head to next node
        let next = lists[idx].take().and_then(|node| node.next);
        lists[idx] = next;
        if let Some(next_node) = &lists[idx] {
            heap.push(Reverse((next_node.val, idx)));
        }
    }

    // Rebuild linked list from sorted values
    let mut head: Option<Box<ListNode>> = None;
    for &v in sorted_vals.iter().rev() {
        let mut node = Box::new(ListNode::new(v));
        node.next = head;
        head = Some(node);
    }
    head
}
```

**Complexity:** O(n log k) time, O(k) space (heap holds at most one node per list)

**Why compare on `node.val` not subtraction?** Node values could be `i32::MIN` / `i32::MAX` — subtraction overflows. In Rust, use `Reverse((node.val, idx))` in the heap entry to get min-heap behavior without a custom comparator.

---

## Problem 2: Sort an Almost Sorted Array (Sort K-Sorted Array)

Each element is at most k positions away from its correct position.

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn sort_k_sorted_array(arr: &[i32], k: usize) -> Vec<i32> {
    let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
    let mut result = Vec::with_capacity(arr.len());

    // Add first k+1 elements to the heap
    for &x in arr.iter().take(k + 1) {
        min_heap.push(Reverse(x));
    }

    // Slide window: for each new element, extract minimum
    for &x in arr.iter().skip(k + 1) {
        if let Some(Reverse(min)) = min_heap.pop() {
            result.push(min);
        }
        min_heap.push(Reverse(x));
    }

    // Drain the heap
    while let Some(Reverse(min)) = min_heap.pop() {
        result.push(min);
    }
    result
}
```

**Why window of k+1?** An element at position i can be at most k positions ahead in the sorted array, meaning it's at most at position i+k. To place element correctly at position i, we must consider elements from position i to i+k — that's k+1 elements.

**Complexity:** O(n log k) time, O(k) space

---

## Problem 3: Find K Pairs with Smallest Sums — LC 373

Given sorted arrays `nums1`, `nums2`, find k pairs `(nums1[i], nums2[j])` with smallest sums.

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn k_smallest_pairs(nums1: &[i32], nums2: &[i32], k: usize) -> Vec<Vec<i32>> {
    let mut result: Vec<Vec<i32>> = Vec::new();
    if nums1.is_empty() || nums2.is_empty() {
        return result;
    }

    // Min-heap: (sum, i, j) — initially push (nums1[i], nums2[0]) for all i
    let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();

    // Key optimization: only push first min(k, nums1.len()) pairs initially
    for i in 0..nums1.len().min(k) {
        heap.push(Reverse((nums1[i] + nums2[0], i, 0)));
    }

    while let Some(Reverse((_, i, j))) = heap.pop() {
        result.push(vec![nums1[i], nums2[j]]);
        if result.len() == k {
            break;
        }
        if j + 1 < nums2.len() {
            heap.push(Reverse((nums1[i] + nums2[j + 1], i, j + 1)));
        }
    }
    result
}
```

**Key insight:** For each row `i` in the conceptual matrix of all `(i,j)` pairs, only advance `j` (columns in sorted order). Initialize with all `(i, 0)` pairs (first column). When we pick `(i, j)`, push `(i, j+1)`.

**Why not push all pairs initially?** O(n²) space. We only need O(k) pairs.

---

## Problem 4: Kth Smallest Element in Sorted Matrix — LC 378

Matrix where each row and column is sorted.

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn kth_smallest(matrix: &[Vec<i32>], k: usize) -> i32 {
    let n = matrix.len();
    // Min-heap: (value, row, col)
    let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();

    // Push first element of each row (first column)
    for r in 0..n.min(k) {
        heap.push(Reverse((matrix[r][0], r, 0)));
    }

    let mut result = 0;
    for _ in 0..k {
        if let Some(Reverse((val, r, c))) = heap.pop() {
            result = val;
            if c + 1 < n {
                heap.push(Reverse((matrix[r][c + 1], r, c + 1)));
            }
        }
    }
    result
}
```

**Complexity:** O(k log k) time for k polls — better than O(n² log n) sort if k is small.

**Alternative — Binary Search — O(n log(max-min)):**
Binary search on the answer value; for each mid, count elements ≤ mid using the staircase search (O(n) per count). Use when k is large.

---

## Problem 5: Smallest Range Covering Elements from K Lists — LC 632

Find the smallest range `[a, b]` such that each of the k lists has at least one element in `[a, b]`.

```rust
use std::cmp::Reverse;
use std::collections::BinaryHeap;

fn smallest_range(nums: &[Vec<i32>]) -> Vec<i32> {
    // Min-heap: (value, list_index, element_index)
    let mut heap: BinaryHeap<Reverse<(i32, usize, usize)>> = BinaryHeap::new();
    let mut max_val = i32::MIN;

    // Push first element from each list
    for (i, list) in nums.iter().enumerate() {
        heap.push(Reverse((list[0], i, 0)));
        max_val = max_val.max(list[0]);
    }

    let mut range_min = heap.peek().unwrap().0.0;
    let mut result = vec![range_min, max_val];

    loop {
        let Reverse((_, list_idx, elem_idx)) = heap.pop().unwrap();

        if elem_idx + 1 == nums[list_idx].len() {
            break; // exhausted a list
        }

        let next_val = nums[list_idx][elem_idx + 1];
        heap.push(Reverse((next_val, list_idx, elem_idx + 1)));
        max_val = max_val.max(next_val);

        range_min = heap.peek().unwrap().0.0;
        if max_val - range_min < result[1] - result[0] {
            result[0] = range_min;
            result[1] = max_val;
        }
    }
    result
}
```

**Invariant:** At all times, the heap contains exactly one element from each list. The range is `[heap.peek(), max_val]`. We minimize this range by advancing the minimum (popping heap root and pushing next from same list). We track `max_val` separately as it can only increase.

**Why stop when a list is exhausted?** We need one element from each list. If any list has no more elements, the current minimum cannot be replaced — the range can only get worse.

---

## K-Way Merge — Heap Entry Design

| Problem | Heap Entry | Compare By |
|---------|-----------|------------|
| Merge K Sorted Lists | `ListNode` | `node.val` |
| K Pairs Smallest Sums | `(sum, i, j)` | `sum` |
| Kth Smallest in Matrix | `(val, row, col)` | `val` |
| Smallest Range | `(val, list_idx, elem_idx)` | `val` |
| Sort K-Sorted | `i32` | natural order |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [Two Heaps](./Two%20Heaps.md)
- [Merge Linked Lists (Linked List topic)](../../4.Linked_List/Patterns/Merge%20Linked%20Lists.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
