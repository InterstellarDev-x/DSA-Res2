# Two Pointers Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Easy–Medium
> **Tags:** `two-pointers` `sorted-array` `in-place` `linear`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [Java Templates](#java-templates)
6. [Common Mistakes](#common-mistakes)
7. [Variations](#variations)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

Two Pointers uses two index variables that move through an array (or two arrays) to reduce a nested-loop O(n²) problem to O(n).

**Three sub-styles:**

| Style | Pointer Start | Move Condition |
|-------|--------------|----------------|
| Opposite ends | `l=0, r=n-1` | Move one inward per step |
| Same direction | `slow=0, fast=0` | Fast races ahead |
| Two arrays | `i=0, j=0` | Advance the smaller one |

The key invariant: at every step, pointers maintain a constraint (e.g. `sum ≤ target`) that guides which pointer moves.

---

## When to Use

- Sorted array, need pairs/triplets summing to target
- Remove duplicates / elements in-place
- Merge two sorted arrays/lists
- Palindrome check
- Container with most water (optimization)
- Partition problems (negative/positive)

---

## Recognition Cues

| Cue in Problem | Style |
|----------------|-------|
| "two numbers in sorted array that sum to target" | Opposite ends |
| "remove duplicates in-place" | Same direction (fast/slow) |
| "merge two sorted arrays" | Two arrays |
| "is palindrome" | Opposite ends |
| "minimum window / subarray" | See [Sliding Window](./Sliding%20Window.md) |
| "container with most water" | Opposite ends |
| "3Sum / 4Sum" | Sort + opposite ends inside loop |

---

## Complexity

| Algorithm | Time | Space |
|-----------|------|-------|
| Two Sum (sorted) | O(n) | O(1) |
| Remove Duplicates | O(n) | O(1) |
| 3Sum | O(n²) | O(1) |
| 4Sum | O(n³) | O(1) |
| Merge Two Arrays | O(m+n) | O(m+n) |

---

## Java Templates

### 1. Opposite Ends — Two Sum (Sorted)

```java
public int[] twoSum(int[] arr, int target) {
    int l = 0, r = arr.length - 1;
    while (l < r) {
        int sum = arr[l] + arr[r];
        if (sum == target) return new int[]{l, r};
        else if (sum < target) l++;
        else r--;
    }
    return new int[]{-1, -1};
}
// Time: O(n) | Space: O(1)
```

### 2. Same Direction — Remove Duplicates (Sorted)

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    int slow = 0;
    for (int fast = 1; fast < nums.length; fast++) {
        if (nums[fast] != nums[slow]) {
            slow++;
            nums[slow] = nums[fast];
        }
    }
    return slow + 1; // new length
}
// Time: O(n) | Space: O(1)
```

### 3. Move Zeros to End

```java
public void moveZeroes(int[] nums) {
    int slow = 0;
    for (int fast = 0; fast < nums.length; fast++) {
        if (nums[fast] != 0) {
            nums[slow++] = nums[fast];
        }
    }
    while (slow < nums.length) nums[slow++] = 0;
}
// Time: O(n) | Space: O(1)
```

### 4. 3Sum

```java
public List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < nums.length - 2; i++) {
        if (i > 0 && nums[i] == nums[i - 1]) continue; // skip duplicate i

        int l = i + 1, r = nums.length - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                result.add(Arrays.asList(nums[i], nums[l], nums[r]));
                while (l < r && nums[l] == nums[l + 1]) l++; // skip dup l
                while (l < r && nums[r] == nums[r - 1]) r--; // skip dup r
                l++; r--;
            } else if (sum < 0) l++;
            else r--;
        }
    }
    return result;
}
// Time: O(n²) | Space: O(1) excluding output
```

### 5. Container With Most Water

```java
public int maxArea(int[] height) {
    int l = 0, r = height.length - 1, max = 0;
    while (l < r) {
        max = Math.max(max, Math.min(height[l], height[r]) * (r - l));
        if (height[l] < height[r]) l++;
        else r--;
    }
    return max;
}
// Time: O(n) | Space: O(1)
```

### 6. Merge Two Sorted Arrays into One

```java
public int[] mergeSorted(int[] a, int[] b) {
    int i = 0, j = 0, k = 0;
    int[] result = new int[a.length + b.length];
    while (i < a.length && j < b.length) {
        result[k++] = (a[i] <= b[j]) ? a[i++] : b[j++];
    }
    while (i < a.length) result[k++] = a[i++];
    while (j < b.length) result[k++] = b[j++];
    return result;
}
// Time: O(m+n) | Space: O(m+n)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Applying Two Pointers to unsorted array | Sort first (adds O(n log n)) or use HashMap |
| Loop condition `l < r` vs `l <= r` | `l < r` for pairs; `l <= r` if element can be used alone |
| 3Sum: not skipping duplicates | Skip when `nums[i] == nums[i-1]` AND `nums[l] == nums[l+1]` |
| Moving both pointers when sum equals target | Move both; `l++` and `r--` together |
| Integer overflow in sum of large values | Cast to `long` before adding |

---

## Variations

| Variation | Description |
|-----------|-------------|
| 4Sum | Sort + three-pointer (one fixed, two-pointer inside) |
| Trapping Rain Water | Two pointers with max-left / max-right tracking |
| Minimum in Rotated Sorted Array | Modified Two Pointers + Binary Search |
| Palindrome (Two Pointers on String) | Opposite ends, compare chars |
| Dutch National Flag | Three pointers (lo, mid, hi) |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Two Sum II - Sorted](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Easy | LC 167 |
| [Remove Duplicates from Sorted Array](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Easy | LC 26 |
| [Move Zeroes](https://leetcode.com/problems/move-zeroes/) | Easy | LC 283 |
| [Squares of Sorted Array](https://leetcode.com/problems/squares-of-a-sorted-array/) | Easy | LC 977 |
| [Container With Most Water](https://leetcode.com/problems/container-with-most-water/) | Medium | LC 11 |
| [3Sum](https://leetcode.com/problems/3sum/) | Medium | LC 15 |
| [4Sum](https://leetcode.com/problems/4sum/) | Medium | LC 18 |
| [Trapping Rain Water](https://leetcode.com/problems/trapping-rain-water/) | Hard | LC 42 |

---

## Related Patterns

- [Sliding Window](./Sliding%20Window.md) — Two Pointers where the window size changes
- [Dutch National Flag](./Dutch%20National%20Flag.md) — Three-pointer variant
- [Prefix Sum](./Prefix%20Sum.md) — When counting subarrays instead of finding pairs

---

> **Interview Tip:** For any "find pair/triplet with sum" on unsorted arrays, ask: "Can I sort it?" Sorting costs O(n log n) but enables O(n) two-pointer scan — almost always the optimal approach.

> **Last Updated:** 2026-06-26
