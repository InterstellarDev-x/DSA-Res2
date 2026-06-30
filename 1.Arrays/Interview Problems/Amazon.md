# Amazon — Arrays Interview Questions

> **Topic:** [Arrays](../README.md) · **Section:** Interview Problems
> **Tags:** `amazon` `interview` `arrays` `FAANG`

---

## Table of Contents

1. [What Amazon Checks](#what-amazon-checks)
2. [Frequently Asked Questions](#frequently-asked-questions)
3. [Amazon Leadership Principles in DSA](#amazon-leadership-principles-in-dsa)
4. [Related Files](#related-files)

---

## What Interviewers Usually Check

| Dimension | What They Want to See |
|-----------|----------------------|
| **Problem decomposition** | Break complex problem into sub-problems |
| **Pattern recognition** | Identify sliding window / prefix sum within 2 minutes |
| **Communication** | Think aloud, narrate your reasoning |
| **Edge cases** | Empty array, all same elements, single element |
| **Complexity analysis** | Proactively state time and space after solution |
| **Optimization** | Move from O(n²) brute force → O(n) optimal |
| **Code quality** | Clean variable names, no magic numbers |
| **LP alignment** | Frame approach in terms of Customer Obsession, Bias for Action |

---

## Frequently Asked Questions

### 1. Two Sum

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy |
| **Round** | Phone screen, Round 1 |
| **Skill tested** | HashMap, Two Pointers on sorted |
| **Optimal solution** | HashMap O(n) time / O(n) space |
| **Pattern** | [Two Pointers](../Patterns/Two%20Pointers.md) |
| **LeetCode** | [LC 1](https://leetcode.com/problems/two-sum/) |

**Common follow-ups:**
- "What if the array is sorted?" → Two Pointers O(1) space
- "What if there are multiple pairs?" → Return all pairs
- "What if we need to handle duplicates?" → Use multiset

---

### 2. Subarray Sum Equals K

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1, Round 2 |
| **Skill tested** | Prefix Sum + HashMap |
| **Optimal solution** | O(n) time, O(n) space |
| **Pattern** | [Prefix Sum](../Patterns/Prefix%20Sum.md) |
| **LeetCode** | [LC 560](https://leetcode.com/problems/subarray-sum-equals-k/) |

**Common follow-ups:**
- "What if K can be negative?" → Same algorithm handles it
- "What if you need the actual subarray?" → Track indices in map

---

### 3. Merge Intervals

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1, Bar Raiser |
| **Skill tested** | Sorting, Greedy |
| **Optimal solution** | Sort by start, O(n log n) time |
| **Pattern** | [Merge Intervals](../Patterns/Merge%20Intervals.md) |
| **LeetCode** | [LC 56](https://leetcode.com/problems/merge-intervals/) |

**Common follow-ups:**
- "What if intervals come in a stream?" → Use TreeMap
- "What's the minimum number of rooms needed?" → Meeting Rooms II (LC 253)
- "What if intervals can have weights?" → Weighted interval scheduling (DP)

---

### 4. Trapping Rain Water

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Round 2, Bar Raiser |
| **Skill tested** | Two Pointers, Prefix/Suffix max |
| **Optimal solution** | Two Pointers O(n) time O(1) space |
| **Pattern** | [Two Pointers](../Patterns/Two%20Pointers.md) |
| **LeetCode** | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |

**Common follow-ups:**
- "Explain why two pointers works vs stack approach"
- "What if the array can be updated?" → Segment Tree

**Java solution (Two Pointers):**
```java
public int trap(int[] height) {
    int l = 0, r = height.length - 1;
    int leftMax = 0, rightMax = 0, water = 0;
    while (l < r) {
        if (height[l] < height[r]) {
            leftMax = Math.max(leftMax, height[l]);
            water += leftMax - height[l];
            l++;
        } else {
            rightMax = Math.max(rightMax, height[r]);
            water += rightMax - height[r];
            r--;
        }
    }
    return water;
}
```

---

### 5. Maximum Subarray (Kadane's)

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone screen, Round 1 |
| **Skill tested** | Kadane's, DP thinking |
| **Optimal solution** | O(n) time O(1) space |
| **Pattern** | [Kadane's Algorithm](../Patterns/Kadane's%20Algorithm.md) |
| **LeetCode** | [LC 53](https://leetcode.com/problems/maximum-subarray/) |

**Common follow-ups:**
- "Print the actual subarray" → Track start/end indices
- "What if all numbers are negative?" → Max is the least negative
- "Circular array version?" → Kadane + (total - minKadane)

---

### 6. Maximum Units on a Truck (Greedy)

| Field | Detail |
|-------|--------|
| **Difficulty** | Easy |
| **Round** | OA, Phone screen |
| **Skill tested** | Greedy sort, Array manipulation |
| **Optimal solution** | Sort by units per box descending, O(n log n) |
| **Pattern** | Greedy |
| **LeetCode** | [LC 1710](https://leetcode.com/problems/maximum-units-on-a-truck/) |

---

### 7. First Missing Positive

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Round 2 |
| **Skill tested** | Cyclic Sort, In-place |
| **Optimal solution** | O(n) time O(1) space |
| **Pattern** | [Cyclic Sort](../Patterns/Cyclic%20Sort.md) |
| **LeetCode** | [LC 41](https://leetcode.com/problems/first-missing-positive/) |

**Common follow-ups:**
- "Why can't you use a HashSet?" → O(n) space — interviewer wants O(1)
- "Why does cyclic sort give O(n)?" → Each element placed at most once

---

### 8. Max Consecutive Ones III

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1 |
| **Skill tested** | Sliding Window |
| **Optimal solution** | Variable sliding window O(n) |
| **Pattern** | [Sliding Window](../Patterns/Sliding%20Window.md) |
| **LeetCode** | [LC 1004](https://leetcode.com/problems/max-consecutive-ones-iii/) |

---

## Amazon Leadership Principles in DSA

| LP | Application in Interview |
|----|-------------------------|
| **Bias for Action** | Start with brute force, refine — don't freeze |
| **Deliver Results** | Always have a working solution, even if not optimal |
| **Dive Deep** | Explain every line, analyze every edge case |
| **Think Big** | Discuss scalability — what if n = 10^9? |
| **Earn Trust** | Admit when stuck, ask clarifying questions |

---

## Related Files

- [Amazon OA Questions](../OA-Qns/Amazon.md)
- [Arrays Most Recent Questions](../Most%20Recent%20Questions/2025.md)
- [Interview Tips — Communication](../Interview%20Tips/Communication.md)
- [Interview Tips — Follow-up Questions](../Interview%20Tips/Follow-up%20Questions.md)
- [Sliding Window Pattern](../Patterns/Sliding%20Window.md)
- [Prefix Sum Pattern](../Patterns/Prefix%20Sum.md)

> **Last Updated:** 2026-06-26
