# Top K Elements

> **Topic:** [Heaps](../README.md) · **Pattern 2 of 5**
> **Problems:** Kth Largest in Array/Stream · K Closest Points · Top K Frequent Elements/Words · Kth Largest Integer

---

## Core Concept

**Goal:** Find the k largest (or smallest, or most frequent) elements efficiently.

**Key insight — min-heap of size k for "k largest":**
- Maintain a min-heap of the k largest elements seen so far
- The root of this heap = kth largest element
- When a new element arrives: push it; if heap size > k, pop (removes the smallest, keeping the k largest)

```
Finding k largest from [3,1,4,1,5,9,2,6], k=3:
Min-heap of size 3:
  After 3:     [3]
  After 1:     [1,3]
  After 4:     [1,3,4]
  After 1 (4th): push 1 → [1,1,3,4]; size>3 → pop 1 → [1,3,4]
  After 5:     push 5 → [1,3,4,5]; pop 1 → [3,4,5]
  After 9:     push 9 → [3,4,5,9]; pop 3 → [4,5,9]
  After 2:     push 2 → [2,4,5,9]; pop 2 → [4,5,9]
  After 6:     push 6 → [4,5,6,9]; pop 4 → [5,6,9]
  Root = 5 = kth largest ✓
```

**Template:**
```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new(); // min-heap of size k
for &num in &nums {
    min_heap.push(Reverse(num));
    if min_heap.len() > k { min_heap.pop(); }  // remove smallest; keep k largest
}
min_heap.peek().map(|&Reverse(x)| x).unwrap() // root = kth largest
```

---

## Problem 1: Kth Largest Element in a Stream — LC 703

Design class: `KthLargest(k, nums)` + `add(val)` returns current kth largest.

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

struct KthLargest {
    min_heap: BinaryHeap<Reverse<i32>>,
    k: usize,
}

impl KthLargest {
    fn new(k: i32, nums: Vec<i32>) -> Self {
        let mut kth = KthLargest {
            min_heap: BinaryHeap::new(),
            k: k as usize,
        };
        for n in nums { kth.add(n); }
        kth
    }

    fn add(&mut self, val: i32) -> i32 {
        self.min_heap.push(Reverse(val));
        if self.min_heap.len() > self.k {
            self.min_heap.pop();
        }
        self.min_heap.peek().map(|&Reverse(x)| x).unwrap()
    }
}
```

**Complexity:** O(log k) per `add` — heap stays size ≤ k.

---

## Problem 2: Kth Largest Element in an Array — LC 215

**Approach 1: Min-heap of size k — O(n log k) time, O(k) space**

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

fn find_kth_largest(nums: Vec<i32>, k: usize) -> i32 {
    let mut min_heap: BinaryHeap<Reverse<i32>> = BinaryHeap::new();
    for n in nums {
        min_heap.push(Reverse(n));
        if min_heap.len() > k { min_heap.pop(); }
    }
    min_heap.peek().map(|&Reverse(x)| x).unwrap()
}
```

**Approach 2: Quickselect — O(n) average, O(n²) worst, O(1) space**

```rust
fn partition_arr(nums: &mut Vec<i32>, left: usize, right: usize) -> usize {
    let pivot = nums[right];
    let mut i = left;
    for j in left..right {
        if nums[j] <= pivot {
            nums.swap(i, j);
            i += 1;
        }
    }
    nums.swap(i, right);
    i
}

fn quickselect(nums: &mut Vec<i32>, left: usize, right: usize, k_smallest: usize) -> i32 {
    if left == right { return nums[left]; }

    let pivot_idx = partition_arr(nums, left, right);
    if pivot_idx == k_smallest {
        nums[pivot_idx]
    } else if pivot_idx < k_smallest {
        quickselect(nums, pivot_idx + 1, right, k_smallest)
    } else {
        quickselect(nums, left, pivot_idx - 1, k_smallest)
    }
}

fn find_kth_largest(mut nums: Vec<i32>, k: usize) -> i32 {
    let n = nums.len();
    quickselect(&mut nums, 0, n - 1, n - k)
    // kth largest = (n-k)th smallest (0-indexed)
}
```

**Which to use?**
- Heap: guaranteed O(n log k), preferred when k << n or for streaming data
- Quickselect: O(n) average, but O(n²) worst case; preferred when modifying the array is OK and k ≈ n/2

---

## Problem 3: K Closest Points to Origin — LC 973

Distance² = x² + y² (no need to take square root — avoids floating point).

```rust
use std::collections::BinaryHeap;

fn k_closest(points: Vec<Vec<i32>>, k: usize) -> Vec<Vec<i32>> {
    // Max-heap of size k — keep k smallest distances; root = kth closest
    // Store (dist_squared, point) — BinaryHeap is max-heap, so farthest is at root
    let mut max_heap: BinaryHeap<(i32, Vec<i32>)> = BinaryHeap::new();

    for p in &points {
        let dist = p[0] * p[0] + p[1] * p[1];
        max_heap.push((dist, p.clone()));
        if max_heap.len() > k { max_heap.pop(); }  // remove farthest
    }

    max_heap.into_iter().map(|(_, p)| p).collect()
}
```

**Why max-heap for k smallest?** To keep the k closest, maintain a max-heap — the root is always the farthest among the k closest. When a new point is closer than the root, we replace the root (pop max, push new). This keeps only the k closest.

**Simpler alternative — sort:**
```rust
fn k_closest_sort(mut points: Vec<Vec<i32>>, k: usize) -> Vec<Vec<i32>> {
    points.sort_by(|a, b| {
        let da = a[0] * a[0] + a[1] * a[1];
        let db = b[0] * b[0] + b[1] * b[1];
        da.cmp(&db)
    });
    points[..k].to_vec()  // O(n log n) time, O(1) space
}
```

Use heap for streaming/online variant; sort for offline.

---

## Problem 4: Top K Frequent Elements — LC 347

```rust
use std::collections::{BinaryHeap, HashMap};
use std::cmp::Reverse;

fn top_k_frequent(nums: Vec<i32>, k: usize) -> Vec<i32> {
    let mut freq: HashMap<i32, i32> = HashMap::new();
    for &n in &nums { *freq.entry(n).or_insert(0) += 1; }

    // Min-heap by frequency: keep k most frequent
    // Store Reverse((freq, key)) so smallest freq is popped first
    let mut min_heap: BinaryHeap<Reverse<(i32, i32)>> = BinaryHeap::new();

    for (&key, &val) in &freq {
        min_heap.push(Reverse((val, key)));
        if min_heap.len() > k { min_heap.pop(); }  // remove least frequent
    }

    let mut result: Vec<i32> = Vec::new();
    while let Some(Reverse((_, key))) = min_heap.pop() {
        result.push(key);
    }
    result.reverse();
    result
}
```

**O(n log k) time, O(n) space** (for freq map + heap of size k)

**Alternative — Bucket Sort — O(n) time:**
```rust
use std::collections::HashMap;

fn top_k_frequent(nums: Vec<i32>, k: usize) -> Vec<i32> {
    let mut freq: HashMap<i32, usize> = HashMap::new();
    for &n in &nums { *freq.entry(n).or_insert(0) += 1; }

    // Buckets: bucket[i] = list of elements with frequency i
    let mut buckets: Vec<Vec<i32>> = vec![vec![]; nums.len() + 1];
    for (&key, &f) in &freq {
        buckets[f].push(key);
    }

    let mut result: Vec<i32> = Vec::new();
    for f in (0..buckets.len()).rev() {
        if result.len() >= k { break; }
        for &n in &buckets[f] {
            if result.len() < k { result.push(n); }
        }
    }
    result
}
```

---

## Problem 5: Top K Frequent Words — LC 692

Same as Top K Frequent Elements but with lexicographic tie-breaking.

```rust
use std::collections::{BinaryHeap, HashMap};

fn top_k_frequent(words: Vec<String>, k: usize) -> Vec<String> {
    let mut freq: HashMap<String, i32> = HashMap::new();
    for w in &words { *freq.entry(w.clone()).or_insert(0) += 1; }

    // Max-heap storing (-freq, word):
    // max element = highest -freq (lowest freq), for ties: lexicographically latest word (to evict)
    let mut heap: BinaryHeap<(i32, String)> = BinaryHeap::new();

    for (w, &f) in &freq {
        heap.push((-f, w.clone()));
        if heap.len() > k { heap.pop(); }  // evict worst (lowest freq, or latest word for ties)
    }

    // Pop gives worst-to-evict order; reverse to get best first
    let mut result: Vec<String> = Vec::new();
    while let Some((_, w)) = heap.pop() {
        result.push(w);
    }
    result.reverse();  // reverse order
    result
}
```

**Why the comparator for tie-breaking?** We want lexicographically smaller words to stay (appear first in answer). The min-heap evicts the "worst" element. For ties in frequency, "worse" = lexicographically larger. So `a < b` as the condition means: if a comes before b, a is "better" (kept); b gets evicted first.

---

## Top K — Decision Table

| Goal | Heap Type | Size | Root Meaning |
|------|----------|------|--------------|
| k largest elements | Min-heap | k | kth largest |
| k smallest elements | Max-heap | k | kth smallest |
| k most frequent elements | Min-heap (by freq) | k | kth most frequent |
| k closest points | Max-heap (by dist) | k | kth closest |

**Rule:** For "k [extreme]", use the **opposite** heap type and maintain size k. The root is the answer.

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
