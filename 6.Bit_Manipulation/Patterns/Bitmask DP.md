# Bitmask DP

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Patterns
> **Applies to:** Subsets enumeration, TSP, Minimum XOR Sum, Shortest Path visiting all nodes

---

## Core Idea

Represent a **subset** of n elements as an integer bitmask where bit `i` is 1 iff element `i` is included. This gives 2^n possible subsets, each an integer in `[0, 2^n - 1]`.

DP state: `dp[mask]` = answer considering exactly the elements in `mask`.

---

## Iterating Subsets

**All subsets of n elements:**
```rust
for mask in 0..(1 << n) {
    // process subset represented by mask
}
```

**All subsets of a given mask (submasks):**
```rust
let mut sub = mask;
while sub > 0 {
    // process submask `sub` of `mask`
    // (sub - 1) & mask clears the lowest set bit of sub within mask's bits
    sub = (sub - 1) & mask;
}
// Note: sub=0 (empty subset) is reached but the loop ends; handle separately if needed
```

**Elements in a mask:**
```rust
for i in 0..n {
    if (mask >> i & 1) == 1 {
        // element i is in the subset
    }
}
```

---

## Template 1 — Subsets via Bitmask (LC 78)

```rust
fn subsets(nums: &[i32]) -> Vec<Vec<i32>> {
    let n = nums.len();
    let mut result = Vec::new();
    for mask in 0..(1 << n) {
        let mut subset = Vec::new();
        for i in 0..n {
            if (mask >> i & 1) == 1 {
                subset.push(nums[i]);
            }
        }
        result.push(subset);
    }
    result
}
```

**Time:** O(n × 2^n). Each of 2^n masks processes n bits.

---

## Template 2 — Minimum XOR Sum of Two Arrays (LC 1879)

Assign each element of `nums2` to one element of `nums1` (bijection). Minimize sum of `nums1[i] XOR nums2[assignment[i]]`.

```rust
fn minimum_xor_sum(nums1: &[i32], nums2: &[i32]) -> i32 {
    let n = nums1.len();
    let mut dp = vec![i32::MAX; 1 << n];
    dp[0] = 0;

    for mask in 0..(1 << n) {
        if dp[mask] == i32::MAX { continue; }
        let i = (mask as u32).count_ones() as usize; // index in nums1 (number of bits set = next to assign)
        if i == n { continue; }
        for j in 0..n {
            if (mask >> j & 1) == 0 { // nums2[j] not yet assigned
                let new_mask = mask | (1 << j);
                dp[new_mask] = dp[new_mask].min(dp[mask] + (nums1[i] ^ nums2[j]));
            }
        }
    }
    dp[(1 << n) - 1]
}
```

**State:** `dp[mask]` = minimum XOR sum when elements of `nums2` corresponding to bits in `mask` have been assigned.
**Transition:** Assign the next `nums1[i]` (where `i = popcount(mask)`) to some unassigned `nums2[j]`.

---

## Template 3 — Shortest Path Visiting All Nodes (LC 847)

BFS with bitmask state: `(node, visited)`.

```rust
use std::collections::VecDeque;

fn shortest_path_length(graph: &[Vec<usize>]) -> i32 {
    let n = graph.len();
    let full_mask = (1 << n) - 1;
    let mut q: VecDeque<(usize, usize, i32)> = VecDeque::new();
    let mut visited = vec![vec![false; 1 << n]; n];

    // Start BFS from all nodes simultaneously
    for i in 0..n {
        q.push_back((i, 1 << i, 0)); // (node, visited_mask, distance)
        visited[i][1 << i] = true;
    }

    while let Some((node, mask, dist)) = q.pop_front() {
        if mask == full_mask { return dist; }
        for &neighbor in &graph[node] {
            let new_mask = mask | (1 << neighbor);
            if !visited[neighbor][new_mask] {
                visited[neighbor][new_mask] = true;
                q.push_back((neighbor, new_mask, dist + 1));
            }
        }
    }
    -1
}
```

**State space:** n nodes × 2^n masks. BFS ensures minimum distance.

---

## Template 4 — Partition to K Equal Sum Subsets (LC 698)

```rust
fn can_partition_k_subsets(nums: &mut Vec<i32>, k: i32) -> bool {
    let sum: i32 = nums.iter().sum();
    if sum % k != 0 { return false; }
    let target = sum / k;
    nums.sort();
    if *nums.last().unwrap() > target { return false; }

    let n = nums.len();
    let mut dp = vec![false; 1 << n]; // dp[mask] = can form valid k-1 subsets using nums in mask
    dp[0] = true;
    let mut cur_sum = vec![0i32; 1 << n]; // current bucket sum for each mask

    for mask in 0..(1 << n) {
        if !dp[mask] { continue; }
        for i in 0..n {
            let new_mask = mask | (1 << i);
            if new_mask == mask { continue; } // i already in mask
            if cur_sum[mask] + nums[i] <= target {
                cur_sum[new_mask] = (cur_sum[mask] + nums[i]) % target;
                dp[new_mask] = true;
            }
        }
    }
    dp[(1 << n) - 1]
}
```

---

## Bitmask DP Pattern Recognition

| Signal | Bitmask DP is likely the answer |
|--------|--------------------------------|
| n ≤ 20 with "all subsets" | Yes |
| "Visit all cities/nodes" (TSP-like) | Yes |
| Assignment problem (bijection) | Yes (LC 1879 pattern) |
| "Minimum cost to complete all tasks" | Yes if n ≤ 15-20 |
| n > 20 | No — 2^20 = 1M is borderline, 2^25 is too large |

---

## Complexity

| Problem | Time | Space |
|---------|------|-------|
| Enumerate all subsets | O(n × 2^n) | O(2^n) |
| Enumerate all submasks | O(3^n) total (sum over all masks of 2^|mask|) | O(1) per submask |
| Minimum XOR Sum | O(n² × 2^n) | O(2^n) |
| Shortest Path All Nodes | O(n² × 2^n) | O(n × 2^n) |
| Partition K Subsets | O(n × 2^n) | O(2^n) |

**Submask enumeration is O(3^n):** Each element can be: in `mask` but not `sub` (2 choices), or in both `mask` and `sub` (1 choice), or in neither (not iterated). Total = 2^(n-|mask|) × 2^|mask| summed = 3^n via binomial identity.

---

## Related Patterns

- [Bit Basics](./Bit%20Basics.md) — foundation operators
- [Subsets & Combinations](../../5.Recursion/Patterns/Subsets%20and%20Combinations.md) — recursive alternative for small n
- [Dynamic Programming](../../14.Dynamic_Programming/README.md) — tabulation over bitmask states

---

**Back:** [Bit Manipulation README](../README.md) | **Prev:** [Counting Bits](./Counting%20Bits.md) | **Next:** [Bit Search & Trie](./Bit%20Search%20and%20Trie.md)

> **Last Updated:** 2026-06-26
