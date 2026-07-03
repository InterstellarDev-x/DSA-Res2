# Amazon — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Amazon

---

## Problem 1: Top K Frequent Elements — Deep Dive

**LC 347** · Medium

### Two Approaches

**Approach 1: Min-heap of size k — O(n log k)**
```rust
use std::collections::{HashMap, BinaryHeap};
use std::cmp::Reverse;

fn top_k_frequent(nums: Vec<i32>, k: usize) -> Vec<i32> {
    let mut freq: HashMap<i32, i32> = HashMap::new();
    for &n in &nums {
        *freq.entry(n).or_insert(0) += 1;
    }

    // min-heap by frequency: Reverse((freq, num))
    let mut min_heap: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new();

    for (&num, &f) in &freq {
        min_heap.push(Reverse((f, num)));
        if min_heap.len() > k {
            min_heap.pop();
        }
    }

    let mut result = vec![0i32; k];
    for i in (0..k).rev() {
        if let Some(Reverse((_, num))) = min_heap.pop() {
            result[i] = num;
        }
    }
    result
}
```

**Approach 2: Bucket sort — O(n)**
```rust
use std::collections::HashMap;

fn top_k_frequent(nums: Vec<i32>, k: usize) -> Vec<i32> {
    let mut freq: HashMap<i32, usize> = HashMap::new();
    for &n in &nums {
        *freq.entry(n).or_insert(0) += 1;
    }

    let mut buckets: Vec<Vec<i32>> = vec![Vec::new(); nums.len() + 1];
    for (&num, &f) in &freq {
        buckets[f].push(num);
    }

    let mut result = Vec::new();
    'outer: for f in (0..buckets.len()).rev() {
        for &n in &buckets[f] {
            if result.len() >= k {
                break 'outer;
            }
            result.push(n);
        }
    }
    result
}
```

**Q: When would you use bucket sort over heap?**
A: When the input values are bounded (nums are integers in a known range). Bucket sort is O(n) but uses O(n) space for the bucket array. For floating-point frequencies or very sparse data, heap is more appropriate.

**Q: Amazon follow-up — Top K Frequent Words with lexicographic tie-breaking?**
A: Add a second comparator condition: for equal frequencies, lexicographically later strings should be evicted first from the min-heap.

---

## Problem 2: Task Scheduler — Amazon Deep Dive

**LC 621** · Medium

**Both approaches must be known:**

```rust
// Formula — O(26 log 26) = O(1) effectively
fn least_interval(tasks: Vec<char>, n: i32) -> i32 {
    let mut freq = vec![0i32; 26];
    for &t in &tasks {
        freq[(t as u8 - b'A') as usize] += 1;
    }
    freq.sort();
    let max_freq = freq[25];
    let max_count = freq.iter().filter(|&&f| f == max_freq).count() as i32;
    ((max_freq - 1) * (n + 1) + max_count).max(tasks.len() as i32)
}
```

**Q: Explain the formula.**
A: Build "frames" of size `n+1`. The most frequent task (`maxFreq`) anchors each frame. There are `maxFreq - 1` full frames, each of size `n+1`. The final slot holds all tasks with max frequency (`maxCount`). Total = `(maxFreq-1)*(n+1) + maxCount`. But if tasks fill all slots with no idle time, the answer is just `tasks.len()`.

**Q: Amazon variant — what if n is very large?**
A: For large n, idle time dominates. The formula still handles this correctly — `(maxFreq-1)*(n+1) + maxCount` grows with n.

**Q: Amazon variant — different cooldown per task type?**
A: The heap simulation approach generalizes to this. The cooldown queue can track per-task cooldowns. The formula approach doesn't generalize.

---

## Problem 3: Merge K Sorted Lists — Amazon Phone Screen

**LC 23** · Hard · O(n log k)

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

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

fn merge_k_lists(lists: Vec<Option<Box<ListNode>>>) -> Option<Box<ListNode>> {
    let mut nodes: Vec<Option<Box<ListNode>>> = lists;
    let k = nodes.len();

    // min-heap: (Reverse(val), list_index) — only one node per list at a time: O(n log k)
    let mut heap: BinaryHeap<Reverse<(i32, usize)>> = BinaryHeap::new();
    for i in 0..k {
        if let Some(ref n) = nodes[i] {
            heap.push(Reverse((n.val, i)));
        }
    }

    let mut result_vals: Vec<i32> = Vec::new();
    while let Some(Reverse((_, i))) = heap.pop() {
        let node = nodes[i].take().unwrap();
        result_vals.push(node.val);
        nodes[i] = node.next;
        if let Some(ref next_node) = nodes[i] {
            heap.push(Reverse((next_node.val, i)));
        }
    }

    // Build linked list from sorted values
    let mut head: Option<Box<ListNode>> = None;
    for &val in result_vals.iter().rev() {
        let mut node = Box::new(ListNode::new(val));
        node.next = head;
        head = Some(node);
    }
    head
}
```

**Q: Why not just concatenate and sort?**
A: O(n log n) where n = total nodes. K-way merge with heap is O(n log k) — better when k << n (many short lists).

**Q: What if k is very large (k = n lists, each length 1)?**
A: Building the heap costs O(k) = O(n). Each poll is O(log k) = O(log n). Total O(n log n) — same as sort. For this degenerate case, sorting is simpler.

---

## Amazon LP Alignment

| LP | Connection |
|----|-----------|
| Dive Deep | Know both heap and bucket sort for Top K; formula and simulation for Task Scheduler |
| Deliver Results | O(n log k) beats O(n log n) for Merge K Lists when k is small |
| Invent & Simplify | Bucket sort O(n) simplifies Top K when input range is bounded |

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Top K Elements](../Patterns/Top%20K%20Elements.md)
- [Heap + Frequency](../Patterns/Heap%20and%20Frequency.md)

> **Last Updated:** 2026-06-26
