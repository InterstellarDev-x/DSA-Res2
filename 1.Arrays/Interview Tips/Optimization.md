# Optimization Tips — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `optimization` `time-complexity` `space-complexity` `interview`

---

## Table of Contents

1. [Optimization Strategy](#optimization-strategy)
2. [Time Complexity Upgrades](#time-complexity-upgrades)
3. [Space Complexity Upgrades](#space-complexity-upgrades)
4. [Common Optimization Techniques](#common-optimization-techniques)
5. [When NOT to Optimize](#when-not-to-optimize)
6. [Related Files](#related-files)

---

## Optimization Strategy

**Step-by-step:**

1. **Start with brute force** — state it, but don't code it unless asked
2. **Identify the bottleneck** — is it the nested loop? The sort? The space?
3. **Ask: what extra information can I precompute?** — prefix sums, frequency maps
4. **Ask: can I avoid recomputation?** — sliding window, DP
5. **Ask: can I trade space for time?** — `std::unordered_map` / `std::unordered_set`
6. **Ask: can I eliminate space?** — in-place manipulation, cyclic sort

---

## Time Complexity Upgrades

| From | To | Technique | Example |
|------|----|-----------|---------|
| O(n²) pair check | O(n) | `std::unordered_map` | Two Sum |
| O(n²) subarray sum | O(n) | [Prefix Sum + `std::unordered_map`](../Patterns/Prefix%20Sum.md) | Subarray Sum = K |
| O(n²) window check | O(n) | [Sliding Window](../Patterns/Sliding%20Window.md) | Longest substring |
| O(n² log n) 3Sum naive | O(n²) | Sort + [Two Pointers](../Patterns/Two%20Pointers.md) | 3Sum |
| O(n log n) merge-find | O(n) | [Cyclic Sort](../Patterns/Cyclic%20Sort.md) | Missing number |
| O(n log n) sort-based majority | O(n) | [Moore's Voting](../Patterns/Moore's%20Voting.md) | Majority element |
| O(n) repeated max scans | O(1) per query | [Sparse Table](../Design%20Data%20Structure%20Problems/Sparse%20Table.md) | RMQ |

---

## Space Complexity Upgrades

| From | To | Technique | Example |
|------|----|-----------|---------|
| O(n) copy for reverse | O(1) | In-place two-pointer | Reverse array |
| O(n) hash set for duplicates | O(1) | Cyclic sort / Floyd's cycle | Find duplicate |
| O(n) prefix array | O(1) | Running variable | Stock buy/sell |
| O(n) result array | O(1) | Encode in input | Set Matrix Zeroes |
| O(n) stack for monotonic | O(1) | Two-pointer variant | Trapping Rain Water |

### Set Matrix Zeroes — O(1) Space Optimization

```cpp
#include <bits/stdc++.h>
using namespace std;

// Use first row and first column as markers
bool firstRowZero = false, firstColZero = false;
// Check if first row/col have zeros originally
for (int j = 0; j < cols; j++) if (matrix[0][j] == 0) firstRowZero = true;
for (int i = 0; i < rows; i++) if (matrix[i][0] == 0) firstColZero = true;
// Mark zeros using first row/col
for (int i = 1; i < rows; i++)
    for (int j = 1; j < cols; j++)
        if (matrix[i][j] == 0) { matrix[i][0] = 0; matrix[0][j] = 0; }
// Apply markers
for (int i = 1; i < rows; i++)
    for (int j = 1; j < cols; j++)
        if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;
// Handle first row and col
if (firstRowZero) fill(matrix[0].begin(), matrix[0].end(), 0);
if (firstColZero) for (int i = 0; i < rows; i++) matrix[i][0] = 0;
```

---

## Common Optimization Techniques

### 1. Precompute Prefix/Suffix Values

Instead of scanning left or right at each step, precompute:

```cpp
#include <bits/stdc++.h>
using namespace std;

// Suffix maximum for Trapping Rain Water
vector<int> rightMax(n);
rightMax[n-1] = height[n-1];
for (int i = n - 2; i >= 0; i--) rightMax[i] = max(height[i], rightMax[i+1]);
```

### 2. Use Complement in `std::unordered_map`

Instead of "find pair that sums to target", store "what I've seen" and check "target - current":

```cpp
#include <bits/stdc++.h>
using namespace std;

// O(n) instead of O(n²)
unordered_map<int, int> seen;
for (int i = 0; i < (int)nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement)) return {seen[complement], i};
    seen[nums[i]] = i;
}
```

### 3. Eliminate Sort with `std::unordered_set`

For "Longest Consecutive Sequence":
- Naive: sort then scan → O(n log n)
- Optimal: `std::unordered_set` + only extend from sequence start → O(n)

### 4. Early Termination

```cpp
// If max so far is already at theoretical maximum, stop
if (maxLen == n) return maxLen;
// In sorted array, if current element > target/3, no valid triplet possible
if (nums[i] > 0 && nums[i] * 3 > target) break;
```

---

## When NOT to Optimize

| Scenario | Reason |
|----------|--------|
| O(n log n) when n ≤ 1000 | Premature optimization — code clarity matters more |
| Trading readability for 2× speedup | Maintainability > micro-optimization |
| When the bottleneck is I/O, not algorithm | Optimizing wrong thing |
| When interviewer hasn't asked | Finish a correct solution first |

> **Rule:** Get a correct solution working first. Optimize only after it's verified correct and the interviewer asks.

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
