# Microsoft — Arrays Interview Questions

> **Topic:** [Arrays](../README.md) · **Section:** Interview Problems
> **Tags:** `microsoft` `interview` `arrays`

---

## Table of Contents

1. [What Microsoft Checks](#what-microsoft-checks)
2. [Frequently Asked Questions](#frequently-asked-questions)
3. [Related Files](#related-files)

---

## What Interviewers Usually Check

| Dimension | What They Want to See |
|-----------|----------------------|
| **Problem understanding** | Ask clarifying questions before coding |
| **Multiple approaches** | Present brute force → optimized |
| **Code quality** | Readable Java, proper naming, helper methods |
| **Test cases** | Write test cases on the whiteboard/IDE |
| **Communication** | Explain as you code |
| **Handle edge cases** | Null, empty, single element, all same |
| **Complexity** | Explicitly state after solution |

---

## Frequently Asked Questions

### 1. Two Sum

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy |
| **Round** | Phone screen |
| **LeetCode** | [LC 1](https://leetcode.com/problems/two-sum/) |
| **Pattern** | HashMap |
| **Follow-ups** | Three sum? Sorted array? Return all pairs? |

---

### 2. Best Time to Buy and Sell Stock

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy |
| **Round** | Phone screen, Round 1 |
| **LeetCode** | [LC 121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) |
| **Skill tested** | Single-pass greedy |
| **Pattern** | Prefix min tracking |

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE, maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
```

**Follow-ups:**
- "What if you can buy/sell multiple times?" → LC 122 (Greedy)
- "What if you can hold at most 2 transactions?" → LC 123 (DP)

---

### 3. Rotate Array

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1 |
| **LeetCode** | [LC 189](https://leetcode.com/problems/rotate-array/) |
| **Skill tested** | In-place reversal trick |
| **Pattern** | Three-reverse |

```java
public void rotate(int[] nums, int k) {
    k %= nums.length;
    reverse(nums, 0, nums.length - 1);
    reverse(nums, 0, k - 1);
    reverse(nums, k, nums.length - 1);
}
private void reverse(int[] arr, int l, int r) {
    while (l < r) { int t = arr[l]; arr[l++] = arr[r]; arr[r--] = t; }
}
```

---

### 4. Set Matrix Zeroes

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1, Round 2 |
| **LeetCode** | [LC 73](https://leetcode.com/problems/set-matrix-zeroes/) |
| **Skill tested** | In-place markers using first row/col |
| **Follow-ups** | Can you do it in O(1) extra space? |

---

### 5. Merge Intervals

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 2 |
| **LeetCode** | [LC 56](https://leetcode.com/problems/merge-intervals/) |
| **Pattern** | [Merge Intervals](../Patterns/Merge%20Intervals.md) |
| **Follow-ups** | Insert interval? Stream of intervals? |

---

### 6. Maximum Subarray

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Round 1 |
| **LeetCode** | [LC 53](https://leetcode.com/problems/maximum-subarray/) |
| **Pattern** | [Kadane's Algorithm](../Patterns/Kadane's%20Algorithm.md) |
| **Follow-ups** | Print the actual subarray? Circular version? |

---

## Related Files

- [Microsoft OA Questions](../OA-Qns/Microsoft.md)
- [Arrays Most Recent Questions](../Most%20Recent%20Questions/2025.md)
- [Interview Tips — Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Kadane's Algorithm Pattern](../Patterns/Kadane's%20Algorithm.md)

> **Last Updated:** 2026-06-26
