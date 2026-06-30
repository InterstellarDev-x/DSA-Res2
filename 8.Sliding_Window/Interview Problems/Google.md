# Google — Sliding Window Interview Problems

> **Topic:** [Sliding Window](../README.md) · **Company:** Google

---

## Problem 1: Binary Subarrays With Sum — Deep Dive

**LC 930** · Medium

### Both Approaches

```java
// Approach 1: atMost trick — O(n) time, O(1) space
public int numSubarraysWithSum(int[] nums, int goal) {
    return atMost(nums, goal) - atMost(nums, goal - 1);
}
private int atMost(int[] nums, int goal) {
    if (goal < 0) return 0;
    int left = 0, sum = 0, result = 0;
    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum > goal) sum -= nums[left++];
        result += right - left + 1;
    }
    return result;
}

// Approach 2: Prefix Sum + HashMap — O(n) time, O(n) space
public int numSubarraysWithSum(int[] nums, int goal) {
    Map<Integer, Integer> prefixCount = new HashMap<>();
    prefixCount.put(0, 1);
    int sum = 0, result = 0;
    for (int num : nums) {
        sum += num;
        result += prefixCount.getOrDefault(sum - goal, 0);
        prefixCount.merge(sum, 1, Integer::sum);
    }
    return result;
}
```

**Q: When would you prefer prefix sum over atMost?**
A: Prefix sum works for negative numbers too. `atMost` sliding window assumes monotonic sums — adding an element can only increase the sum. For arrays with negatives, the `while (sum > goal)` shrink loop can overshoot. Prefix sum doesn't have this restriction.

**Q: Why `prefixCount.put(0, 1)` initialization?**
A: The subarray from index 0 to some index `j` has sum = `prefix[j+1]`. For this to equal `goal`, we need `prefix[j+1] - prefix[0] = goal → prefix[0] = prefix[j+1] - goal`. Since `prefix[0] = 0`, we initialize with `count[0] = 1` to count subarrays starting from index 0.

---

## Problem 2: Max Consecutive Ones III — Google Style

**LC 1004** · Medium

```java
public int longestOnes(int[] nums, int k) {
    int left = 0, zeros = 0, maxLen = 0;
    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeros++;
        while (zeros > k) {
            if (nums[left] == 0) zeros--;
            left++;
        }
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Q: Google follow-up — what if we can also flip at most k ones to zeros (i.e., keep at most k zeros OR ones)?**
A: Two separate calls: `Math.max(longestOnes(nums, k), longestZeros(nums, k))` where `longestZeros` flips the values.

**Q: Google follow-up — if you could flip any substring (not individual bits) at most once, find the longest all-ones subarray?**
A: This becomes a different problem — find the longest window with ≤ 1 "block" of zeros (since flipping a substring flips all bits in it). Use segment analysis or dynamic programming.

---

## Problem 3: Number of Substrings Containing All Three Characters

**LC 1358** · Medium

```java
public int numberOfSubstrings(String s) {
    int[] count = new int[3];
    int left = 0, result = 0;
    for (int right = 0; right < s.length(); right++) {
        count[s.charAt(right) - 'a']++;
        while (count[0] > 0 && count[1] > 0 && count[2] > 0) {
            result += s.length() - right;
            count[s.charAt(left++) - 'a']--;
        }
    }
    return result;
}
```

**Q: Explain the `result += s.length() - right` insight.**
A: When `[left..right]` contains all three characters, extending the right boundary to any position from `right` to `n-1` still contains all three. So every substring `[left..right]`, `[left..right+1]`, ..., `[left..n-1]` is valid — that's `n - right` subarrays starting at `left`. Then we shrink `left` and check if the shorter window still has all three.

**Google L4 follow-up:** Generalize to count substrings containing all characters of a given target string `t`. Use the `have/need` framework from Minimum Window Substring.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | LSWORC, Max Consecutive Ones, Minimum Subarray Sum |
| L4 | Min Window Substring, Binary Subarrays, All Three Characters |
| L5+ | Min Window Substring with follow-ups, combined prefix sum + sliding window |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [At Most K Trick](../Patterns/At%20Most%20K%20Trick.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
