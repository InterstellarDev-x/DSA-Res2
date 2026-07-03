# Google — Arrays Interview Questions

> **Topic:** [Arrays](../README.md) · **Section:** Interview Problems
> **Tags:** `google` `interview` `arrays` `FAANG`

---

## Table of Contents

1. [What Google Checks](#what-google-checks)
2. [Frequently Asked Questions](#frequently-asked-questions)
3. [Google Interview Ladder Context](#google-interview-ladder-context)
4. [Related Files](#related-files)

---

## What Interviewers Usually Check

| Dimension | What They Want to See |
|-----------|----------------------|
| **Optimal from the start** | Don't just brute force — explain the optimal approach first |
| **Clean code** | Readable, modular — prefer helper methods |
| **Mathematical rigor** | Prove correctness, not just "it seems to work" |
| **Complexity analysis** | Both time AND space, before and after optimization |
| **Edge cases upfront** | Mention them before coding, handle them in code |
| **Scalability** | "What if n = 10^9 and we can only use O(1) space?" |
| **Communication** | Narrate thought process, don't code silently |

---

## Frequently Asked Questions

### 1. Product of Array Except Self

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Onsite Round 1 |
| **Skill tested** | Prefix + Suffix products, O(1) space trick |
| **Optimal solution** | O(n) time, O(1) extra space |
| **LeetCode** | [LC 238](https://leetcode.com/problems/product-of-array-except-self/) |

**C++ Solution:**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> productExceptSelf(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n);
    result[0] = 1;
    // Left pass: result[i] = product of all nums[0..i-1]
    for (int i = 1; i < n; i++) result[i] = result[i - 1] * nums[i - 1];
    // Right pass: multiply by suffix product
    int right = 1;
    for (int i = n - 1; i >= 0; i--) {
        result[i] *= right;
        right *= nums[i];
    }
    return result;
}
// Time: O(n) | Space: O(1) excluding output array
```

**Follow-ups:**
- "What if you can use division?" → Handle zeros carefully
- "What if multiple zeros?" → Result is all zeros except possibly one position

---

### 2. Trapping Rain Water

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Onsite Round 2/3 |
| **Skill tested** | Multiple approaches — stack, two pointers, prefix |
| **Optimal solution** | Two Pointers O(n) O(1) |
| **Pattern** | [Two Pointers](../Patterns/Two%20Pointers.md) |
| **LeetCode** | [LC 42](https://leetcode.com/problems/trapping-rain-water/) |

**Follow-ups:**
- "Explain three different approaches and their trade-offs"
- "Volume of water in 3D" (LC 407 — Trapping Rain Water II)

---

### 3. Longest Consecutive Sequence

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Round 1 |
| **Skill tested** | unordered_set, O(n) without sorting |
| **Optimal solution** | unordered_set, only extend sequences from their start |
| **LeetCode** | [LC 128](https://leetcode.com/problems/longest-consecutive-sequence/) |

```cpp
#include <bits/stdc++.h>
using namespace std;

int longestConsecutive(vector<int>& nums) {
    unordered_set<int> set(nums.begin(), nums.end());
    int maxLen = 0;
    for (auto& n : set) {
        if (set.find(n - 1) == set.end()) { // start of a sequence
            int cur = n, len = 1;
            while (set.count(cur + 1)) { cur++; len++; }
            maxLen = max(maxLen, len);
        }
    }
    return maxLen;
}
// Time: O(n) | Space: O(n)
```

---

### 4. Maximum Subarray

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone screen |
| **Skill tested** | Kadane's + DP thinking |
| **Optimal solution** | Kadane O(n) O(1) |
| **Pattern** | [Kadane's Algorithm](../Patterns/Kadane's%20Algorithm.md) |
| **LeetCode** | [LC 53](https://leetcode.com/problems/maximum-subarray/) |

---

### 5. First Missing Positive

| Field | Detail |
|-------|--------|
| **Difficulty** | Hard |
| **Round** | Onsite Round 2 |
| **Skill tested** | Cyclic Sort / Index manipulation |
| **Optimal solution** | O(n) time O(1) space |
| **Pattern** | [Cyclic Sort](../Patterns/Cyclic%20Sort.md) |
| **LeetCode** | [LC 41](https://leetcode.com/problems/first-missing-positive/) |

---

### 6. Spiral Matrix II

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Round 1 |
| **Skill tested** | 2D array simulation, boundary tracking |
| **LeetCode** | [LC 59](https://leetcode.com/problems/spiral-matrix-ii/) |

---

### 7. Subarray Sum Equals K

| Field | Detail |
|-------|--------|
| **Difficulty** | Medium |
| **Round** | Phone, Round 1 |
| **Skill tested** | Prefix Sum + HashMap |
| **Pattern** | [Prefix Sum](../Patterns/Prefix%20Sum.md) |
| **LeetCode** | [LC 560](https://leetcode.com/problems/subarray-sum-equals-k/) |

---

## Google Interview Ladder Context

| Level | Focus |
|-------|-------|
| L3 (SWE) | Medium arrays, correct + clean solution |
| L4 (SWE II) | Medium-Hard, optimal solution required |
| L5 (Senior) | Hard arrays + scalability discussion |
| L6 (Staff) | System design component with arrays |

---

## Related Files

- [Google OA Questions](../OA-Qns/Google.md)
- [Arrays Most Recent Questions](../Most%20Recent%20Questions/2025.md)
- [Interview Tips — Optimization](../Interview%20Tips/Optimization.md)
- [Prefix Sum Pattern](../Patterns/Prefix%20Sum.md)
- [Two Pointers Pattern](../Patterns/Two%20Pointers.md)

> **Last Updated:** 2026-06-26
