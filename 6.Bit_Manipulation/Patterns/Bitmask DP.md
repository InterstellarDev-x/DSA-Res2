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
```java
for (int mask = 0; mask < (1 << n); mask++) {
    // process subset represented by mask
}
```

**All subsets of a given mask (submasks):**
```java
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) {
    // process submask `sub` of `mask`
    // (sub - 1) & mask clears the lowest set bit of sub within mask's bits
}
// Note: sub=0 (empty subset) is reached but the loop ends; handle separately if needed
```

**Elements in a mask:**
```java
for (int i = 0; i < n; i++) {
    if ((mask >> i & 1) == 1) {
        // element i is in the subset
    }
}
```

---

## Template 1 — Subsets via Bitmask (LC 78)

```java
public List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();
    for (int mask = 0; mask < (1 << n); mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask >> i & 1) == 1) subset.add(nums[i]);
        }
        result.add(subset);
    }
    return result;
}
```

**Time:** O(n × 2^n). Each of 2^n masks processes n bits.

---

## Template 2 — Minimum XOR Sum of Two Arrays (LC 1879)

Assign each element of `nums2` to one element of `nums1` (bijection). Minimize sum of `nums1[i] XOR nums2[assignment[i]]`.

```java
public int minimumXORSum(int[] nums1, int[] nums2) {
    int n = nums1.length;
    int[] dp = new int[1 << n];
    Arrays.fill(dp, Integer.MAX_VALUE);
    dp[0] = 0;

    for (int mask = 0; mask < (1 << n); mask++) {
        if (dp[mask] == Integer.MAX_VALUE) continue;
        int i = Integer.bitCount(mask); // index in nums1 (number of bits set = next to assign)
        if (i == n) continue;
        for (int j = 0; j < n; j++) {
            if ((mask >> j & 1) == 0) { // nums2[j] not yet assigned
                int newMask = mask | (1 << j);
                dp[newMask] = Math.min(dp[newMask], dp[mask] + (nums1[i] ^ nums2[j]));
            }
        }
    }
    return dp[(1 << n) - 1];
}
```

**State:** `dp[mask]` = minimum XOR sum when elements of `nums2` corresponding to bits in `mask` have been assigned.
**Transition:** Assign the next `nums1[i]` (where `i = popcount(mask)`) to some unassigned `nums2[j]`.

---

## Template 3 — Shortest Path Visiting All Nodes (LC 847)

BFS with bitmask state: `(node, visited)`.

```java
public int shortestPathLength(int[][] graph) {
    int n = graph.length;
    int fullMask = (1 << n) - 1;
    Queue<int[]> queue = new LinkedList<>();
    boolean[][] visited = new boolean[n][1 << n];

    // Start BFS from all nodes simultaneously
    for (int i = 0; i < n; i++) {
        queue.offer(new int[]{i, 1 << i, 0}); // {node, visited_mask, distance}
        visited[i][1 << i] = true;
    }

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int node = curr[0], mask = curr[1], dist = curr[2];
        if (mask == fullMask) return dist;
        for (int neighbor : graph[node]) {
            int newMask = mask | (1 << neighbor);
            if (!visited[neighbor][newMask]) {
                visited[neighbor][newMask] = true;
                queue.offer(new int[]{neighbor, newMask, dist + 1});
            }
        }
    }
    return -1;
}
```

**State space:** n nodes × 2^n masks. BFS ensures minimum distance.

---

## Template 4 — Partition to K Equal Sum Subsets (LC 698)

```java
public boolean canPartitionKSubsets(int[] nums, int k) {
    int sum = Arrays.stream(nums).sum();
    if (sum % k != 0) return false;
    int target = sum / k;
    Arrays.sort(nums);
    if (nums[nums.length - 1] > target) return false;

    int n = nums.length;
    boolean[] dp = new boolean[1 << n]; // dp[mask] = can form valid k-1 subsets using nums in mask
    dp[0] = true;
    int[] curSum = new int[1 << n]; // current bucket sum for each mask

    for (int mask = 0; mask < (1 << n); mask++) {
        if (!dp[mask]) continue;
        for (int i = 0; i < n; i++) {
            int newMask = mask | (1 << i);
            if (newMask == mask) continue; // i already in mask
            if (curSum[mask] + nums[i] <= target) {
                curSum[newMask] = (curSum[mask] + nums[i]) % target;
                dp[newMask] = true;
            }
        }
    }
    return dp[(1 << n) - 1];
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
