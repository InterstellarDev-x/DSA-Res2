# Cyclic Sort Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `cyclic-sort` `in-place` `missing-number` `duplicate`

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

Cyclic Sort places each element at its **correct index** in a single O(n) pass. It works when the array contains numbers in range `[1, n]` or `[0, n]`.

**Core Insight:** If `nums[i]` belongs at index `nums[i] - 1` and isn't already there, swap it into place. Don't advance `i` until the element at `i` is correct.

```
i = 0
while i < n:
    correct = nums[i] - 1   // index where nums[i] belongs
    if nums[i] != nums[correct]:
        swap(nums[i], nums[correct])
    else:
        i++
```

After sorting, scan for anomalies (missing index, duplicate value).

---

## When to Use

- Array contains numbers in range `[1, n]` or `[0, n-1]`
- Finding missing/duplicate numbers in O(n) time, O(1) space
- Placing elements at their "natural" index

---

## Recognition Cues

| Cue in Problem | Pattern |
|----------------|---------|
| "numbers from 1 to n, find missing" | Cyclic Sort |
| "numbers from 1 to n, find duplicate" | Cyclic Sort |
| "find all missing numbers in [1, n]" | Cyclic Sort + scan |
| "first missing positive" | Cyclic Sort variant |
| "set of n+1 numbers in [1, n]" | Cyclic Sort |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Cyclic sort pass | O(n) | O(1) |
| Find one missing | O(n) | O(1) |
| Find all missing | O(n) | O(1) |

> **Why O(n)?** Each element is swapped at most once — into its correct position. Total swaps ≤ n.

---

## Java Templates

### 1. Cyclic Sort — Sort Numbers [1, n]

```java
public void cyclicSort(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i] - 1; // nums[i] belongs at index correct
        if (nums[i] != nums[correct]) {
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else {
            i++;
        }
    }
}
// Time: O(n) | Space: O(1)
```

### 2. Find the Missing Number [0, n]

```java
public int missingNumber(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i];
        // nums[i] should be at index nums[i], range [0, n]
        if (nums[i] < nums.length && nums[i] != nums[correct]) {
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else {
            i++;
        }
    }
    // Scan for mismatch
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != j) return j;
    }
    return nums.length; // n is missing
}
// Time: O(n) | Space: O(1)
```

### 3. Find the Duplicate Number [1, n] (One Duplicate)

```java
public int findDuplicate(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i] - 1;
        if (nums[i] != i + 1) {
            if (nums[i] != nums[correct]) {
                int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
            } else {
                return nums[i]; // duplicate found
            }
        } else {
            i++;
        }
    }
    return -1;
}
// Time: O(n) | Space: O(1)
```

### 4. Find All Missing Numbers [1, n]

```java
public List<Integer> findDisappearedNumbers(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i] - 1;
        if (nums[i] != nums[correct]) {
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else {
            i++;
        }
    }
    List<Integer> missing = new ArrayList<>();
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != j + 1) missing.add(j + 1);
    }
    return missing;
}
// Time: O(n) | Space: O(1) (output not counted)
```

### 5. Find All Duplicates [1, n]

```java
public List<Integer> findAllDuplicates(int[] nums) {
    int i = 0;
    while (i < nums.length) {
        int correct = nums[i] - 1;
        if (nums[i] != nums[correct]) {
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else {
            i++;
        }
    }
    List<Integer> duplicates = new ArrayList<>();
    for (int j = 0; j < nums.length; j++) {
        if (nums[j] != j + 1) duplicates.add(nums[j]);
    }
    return duplicates;
}
```

### 6. First Missing Positive (Cyclic Sort Variant)

```java
public int firstMissingPositive(int[] nums) {
    int n = nums.length, i = 0;
    while (i < n) {
        int correct = nums[i] - 1;
        if (nums[i] > 0 && nums[i] <= n && nums[i] != nums[correct]) {
            int tmp = nums[i]; nums[i] = nums[correct]; nums[correct] = tmp;
        } else {
            i++;
        }
    }
    for (int j = 0; j < n; j++) {
        if (nums[j] != j + 1) return j + 1;
    }
    return n + 1;
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Advancing `i` after every swap | Only advance when `nums[i] == i+1` (already correct) |
| Not handling out-of-range values | Guard: `nums[i] > 0 && nums[i] <= n` |
| Infinite loop when duplicate exists | Check `nums[i] != nums[correct]` before swapping |
| Missing the `n` case in `[0, n]` | After scan, return `n` if nothing found |
| Modifying input when you shouldn't | Clarify with interviewer if mutation is allowed |

---

## Variations

| Variation | Description |
|-----------|-------------|
| [0, n-1] range | `correct = nums[i]` instead of `nums[i] - 1` |
| Multiple missing numbers | Scan all mismatches after sort |
| Corrupt pair (one missing, one duplicate) | Run cyclic sort, scan for both |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Missing Number](https://leetcode.com/problems/missing-number/) | Easy | LC 268 |
| [Find the Duplicate Number](https://leetcode.com/problems/find-the-duplicate-number/) | Medium | LC 287 |
| [Find All Numbers Disappeared in an Array](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Easy | LC 448 |
| [Find All Duplicates in an Array](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Medium | LC 442 |
| [First Missing Positive](https://leetcode.com/problems/first-missing-positive/) | Hard | LC 41 |
| [Set Mismatch](https://leetcode.com/problems/set-mismatch/) | Easy | LC 645 |

---

## Related Patterns

- [Dutch National Flag](./Dutch%20National%20Flag.md) — in-place element placement
- [Two Pointers](./Two%20Pointers.md) — for in-place swapping
- [Bit Manipulation](../../6.Bit_Manipulation/README.md) — XOR trick for single missing number

---

> **Interview Tip:** LC 41 (First Missing Positive) is a classic Hard that uses this exact pattern. The "don't advance i" rule is the key insight interviewers check — many candidates write infinite loops.

> **Last Updated:** 2026-06-26
