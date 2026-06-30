# Sliding Window Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `sliding-window` `subarray` `two-pointers` `frequency-map`

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

Sliding Window maintains a **dynamic subarray** `[l, r]` that expands and contracts according to a validity condition. Instead of recomputing properties from scratch for each window, it incrementally updates as the window slides.

**Two flavors:**

| Type | Window Size | Use When |
|------|-------------|----------|
| Fixed Window | Constant `k` | "subarray of size k with max sum" |
| Variable Window | Grows/shrinks | "longest subarray with property X" |

**Core loop structure (variable window):**
```
expand r → include nums[r]
while (window invalid):
    shrink l → exclude nums[l], l++
update answer
```

---

## When to Use

- Subarray / substring problems with a **contiguous** constraint
- Max/min length subarray satisfying a condition
- Count of distinct elements in window
- Fixed-size window maximum/minimum/sum
- All values non-negative (for sum-based variable window)

---

## Recognition Cues

| Cue in Problem | Style |
|----------------|-------|
| "subarray of size k" | Fixed window |
| "longest subarray / substring with ..." | Variable window (maximize) |
| "smallest subarray with sum ≥ k" | Variable window (minimize) |
| "at most k distinct characters / elements" | Variable window + frequency map |
| "no repeating characters" | Variable window + HashSet |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Fixed window (sum) | O(n) | O(1) |
| Variable window (HashSet) | O(n) | O(n) |
| Variable window (freq map) | O(n) | O(26) / O(n) |

---

## Java Templates

### 1. Fixed Window — Maximum Sum of Size K

```java
public int maxSumSubarray(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i]; // first window

    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k]; // slide: add new, remove old
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
// Time: O(n) | Space: O(1)
```

### 2. Variable Window — Longest Subarray with Sum ≤ K (Non-negative values)

```java
public int longestSubarrayWithSumK(int[] arr, int k) {
    int l = 0, sum = 0, maxLen = 0;
    for (int r = 0; r < arr.length; r++) {
        sum += arr[r];
        while (sum > k) sum -= arr[l++]; // shrink from left
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1)
// Note: while loop is amortized O(n) total — l never decreases
```

### 3. Variable Window — Longest Substring Without Repeating Characters

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int l = 0, maxLen = 0;
    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= l) {
            l = lastSeen.get(c) + 1; // jump l past the duplicate
        }
        lastSeen.put(c, r);
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(min(n, 26))
```

### 4. Variable Window — At Most K Distinct Characters

```java
public int longestSubstringKDistinct(String s, int k) {
    Map<Character, Integer> freq = new HashMap<>();
    int l = 0, maxLen = 0;
    for (int r = 0; r < s.length(); r++) {
        freq.merge(s.charAt(r), 1, Integer::sum);
        while (freq.size() > k) {
            char left = s.charAt(l++);
            freq.merge(left, -1, Integer::sum);
            if (freq.get(left) == 0) freq.remove(left);
        }
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(k)
```

### 5. Minimum Window Substring (Hard)

```java
public String minWindow(String s, String t) {
    if (s.isEmpty() || t.isEmpty()) return "";
    Map<Character, Integer> need = new HashMap<>();
    for (char c : t.toCharArray()) need.merge(c, 1, Integer::sum);

    int l = 0, formed = 0, required = need.size();
    Map<Character, Integer> window = new HashMap<>();
    int[] ans = {-1, 0, 0}; // [length, l, r]

    for (int r = 0; r < s.length(); r++) {
        char c = s.charAt(r);
        window.merge(c, 1, Integer::sum);
        if (need.containsKey(c) && window.get(c).equals(need.get(c))) formed++;

        while (formed == required) {
            if (ans[0] == -1 || r - l + 1 < ans[0]) {
                ans[0] = r - l + 1; ans[1] = l; ans[2] = r;
            }
            char lc = s.charAt(l++);
            window.merge(lc, -1, Integer::sum);
            if (need.containsKey(lc) && window.get(lc) < need.get(lc)) formed--;
        }
    }
    return ans[0] == -1 ? "" : s.substring(ans[1], ans[2] + 1);
}
// Time: O(|s| + |t|) | Space: O(|s| + |t|)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using sliding window when values can be negative | Switch to Prefix Sum + HashMap |
| Not removing key from map when frequency → 0 | Always `map.remove(key)` when count hits 0 |
| Fixed window: off-by-one when removing old element | Remove `arr[i - k]`, not `arr[i - k + 1]` |
| Variable window: over-shrinking | `while` condition, not `if` |
| Counting `formed` with `>=` instead of `==` | Only increment `formed` when count matches exactly |

---

## Variations

| Variation | Description |
|-----------|-------------|
| Sliding Window Maximum | Monotonic Deque inside window (LC 239) |
| Exactly K distinct = atMost(K) - atMost(K-1) | Count subarrays |
| Permutation in String | Fixed window with char frequency |
| Find All Anagrams | Fixed window + frequency comparison |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Maximum Average Subarray I](https://leetcode.com/problems/maximum-average-subarray-i/) | Easy | LC 643 |
| [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | LC 3 |
| [Minimum Size Subarray Sum](https://leetcode.com/problems/minimum-size-subarray-sum/) | Medium | LC 209 |
| [Fruit Into Baskets](https://leetcode.com/problems/fruit-into-baskets/) | Medium | LC 904 |
| [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | LC 567 |
| [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | LC 438 |
| [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | LC 424 |
| [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | LC 76 |
| [Sliding Window Maximum](https://leetcode.com/problems/sliding-window-maximum/) | Hard | LC 239 |

---

## Related Patterns

- [Two Pointers](./Two%20Pointers.md) — Sliding Window is Two Pointers with extra bookkeeping
- [Prefix Sum](./Prefix%20Sum.md) — when values can be negative (Sliding Window breaks)
- [Monotonic Stack/Deque](../../7.Stacks_and_Queues/Patterns/Monotonic%20Stack.md) — for window max/min in O(n)

---

> **Interview Tip:** The most important question to ask yourself: "Are all values non-negative?" If yes, sliding window works for sum constraints. If values can be negative, switch to Prefix Sum + HashMap.

> **Last Updated:** 2026-06-26
