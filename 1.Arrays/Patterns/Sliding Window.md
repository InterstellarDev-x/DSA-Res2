# Sliding Window Pattern

> **Topic:** [Arrays](../README.md) · **Difficulty:** Medium
> **Tags:** `sliding-window` `subarray` `two-pointers` `frequency-map`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [C++ Templates](#cpp-templates)
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
| "no repeating characters" | Variable window + unordered_set |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Fixed window (sum) | O(n) | O(1) |
| Variable window (unordered_set) | O(n) | O(n) |
| Variable window (freq map) | O(n) | O(26) / O(n) |

---

## C++ Templates

### 1. Fixed Window — Maximum Sum of Size K

```cpp
#include <bits/stdc++.h>
using namespace std;

int maxSumSubarray(vector<int>& arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i]; // first window

    int maxSum = windowSum;
    for (int i = k; i < (int)arr.size(); i++) {
        windowSum += arr[i] - arr[i - k]; // slide: add new, remove old
        maxSum = max(maxSum, windowSum);
    }
    return maxSum;
}
// Time: O(n) | Space: O(1)
```

### 2. Variable Window — Longest Subarray with Sum ≤ K (Non-negative values)

```cpp
#include <bits/stdc++.h>
using namespace std;

int longestSubarrayWithSumK(vector<int>& arr, int k) {
    int l = 0, sum = 0, maxLen = 0;
    for (int r = 0; r < (int)arr.size(); r++) {
        sum += arr[r];
        while (sum > k) sum -= arr[l++]; // shrink from left
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1)
// Note: while loop is amortized O(n) total — l never decreases
```

### 3. Variable Window — Longest Substring Without Repeating Characters

```cpp
#include <bits/stdc++.h>
using namespace std;

int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> lastSeen;
    int l = 0, maxLen = 0;
    for (int r = 0; r < (int)s.length(); r++) {
        char c = s[r];
        if (lastSeen.count(c) && lastSeen[c] >= l) {
            l = lastSeen[c] + 1; // jump l past the duplicate
        }
        lastSeen[c] = r;
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(min(n, 26))
```

### 4. Variable Window — At Most K Distinct Characters

```cpp
#include <bits/stdc++.h>
using namespace std;

int longestSubstringKDistinct(string s, int k) {
    unordered_map<char, int> freq;
    int l = 0, maxLen = 0;
    for (int r = 0; r < (int)s.length(); r++) {
        freq[s[r]]++;
        while ((int)freq.size() > k) {
            char left = s[l++];
            freq[left]--;
            if (freq[left] == 0) freq.erase(left);
        }
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(k)
```

### 5. Minimum Window Substring (Hard)

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow(string s, string t) {
    if (s.empty() || t.empty()) return "";
    unordered_map<char, int> need;
    for (char c : t) need[c]++;

    int l = 0, formed = 0, required = (int)need.size();
    unordered_map<char, int> window;
    int ansLen = -1, ansL = 0, ansR = 0; // [length, l, r]

    for (int r = 0; r < (int)s.length(); r++) {
        char c = s[r];
        window[c]++;
        if (need.count(c) && window[c] == need[c]) formed++;

        while (formed == required) {
            if (ansLen == -1 || r - l + 1 < ansLen) {
                ansLen = r - l + 1; ansL = l; ansR = r;
            }
            char lc = s[l++];
            window[lc]--;
            if (need.count(lc) && window[lc] < need[lc]) formed--;
        }
    }
    return ansLen == -1 ? "" : s.substr(ansL, ansLen);
}
// Time: O(|s| + |t|) | Space: O(|s| + |t|)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using sliding window when values can be negative | Switch to Prefix Sum + HashMap |
| Not removing key from map when frequency → 0 | Always `map.erase(key)` when count hits 0 |
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
