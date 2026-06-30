# Variable Size Window

> **Topic:** [Sliding Window](../README.md) · **Pattern 2 of 4**
> **Problems:** Min Subarray Sum · LSWORC · Max Cons Ones III · Fruits Into Baskets · Char Replacement · Min Window Substring · All Three Characters · Longest Subarray After Delete

---

## Core Concept

A **variable-size window** expands its right boundary to include new elements and shrinks its left boundary when a constraint is violated. The window is always valid — it represents a range satisfying the problem constraint.

**Two variants:**

| Variant | Shrink When | Typical Goal |
|---------|-------------|--------------|
| **Minimize window** | Constraint is MET (keep shrinking as long as valid) | Minimum length satisfying constraint |
| **Maximize window** | Constraint is VIOLATED (shrink until valid again) | Maximum length satisfying constraint |

---

## Template A: Maximize Window (Shrink on Violation)

```java
// Find the longest window satisfying the constraint
int left = 0, maxLen = 0;
// data structure to track window state (Map, array, counter)

for (int right = 0; right < n; right++) {
    // 1. Expand: add nums[right] to window
    add(nums[right]);

    // 2. Shrink: while constraint violated, remove from left
    while (violated()) {
        remove(nums[left]);
        left++;
    }

    // 3. Window [left..right] is now valid — update answer
    maxLen = Math.max(maxLen, right - left + 1);
}
return maxLen;
```

---

## Template B: Minimize Window (Shrink While Valid)

```java
// Find the smallest window satisfying the constraint
int left = 0, minLen = Integer.MAX_VALUE;

for (int right = 0; right < n; right++) {
    // 1. Expand: add nums[right]
    add(nums[right]);

    // 2. Shrink: while constraint is satisfied, try to make window smaller
    while (satisfied()) {
        minLen = Math.min(minLen, right - left + 1);
        remove(nums[left]);
        left++;
    }
}
return minLen == Integer.MAX_VALUE ? 0 : minLen;
```

---

## Problem 1: Minimum Size Subarray Sum — LC 209

Find the minimum length subarray with sum ≥ target.

```java
public int minSubArrayLen(int target, int[] nums) {
    int left = 0, sum = 0, minLen = Integer.MAX_VALUE;

    for (int right = 0; right < nums.length; right++) {
        sum += nums[right];
        while (sum >= target) {            // constraint MET → shrink
            minLen = Math.min(minLen, right - left + 1);
            sum -= nums[left++];
        }
    }
    return minLen == Integer.MAX_VALUE ? 0 : minLen;
}
```

**Why `while` and not `if`?** After removing `nums[left]` from the window, the sum might still be ≥ target. We keep shrinking as long as the window remains valid (shrinking finds a shorter valid window).

**Complexity:** O(n) time — each element enters and exits the window at most once.

---

## Problem 2: Longest Substring Without Repeating Characters — LC 3

```java
public int lengthOfLongestSubstring(String s) {
    int[] freq = new int[128];  // ASCII
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        freq[s.charAt(right)]++;

        while (freq[s.charAt(right)] > 1) {  // duplicate found → shrink
            freq[s.charAt(left++)]--;
        }

        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Alternative: HashMap with last-seen index — jump left directly**

```java
public int lengthOfLongestSubstring(String s) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (lastSeen.containsKey(c) && lastSeen.get(c) >= left) {
            left = lastSeen.get(c) + 1;   // jump left past the duplicate
        }
        lastSeen.put(c, right);
        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Which to use?** Frequency array approach is more general (works for all sliding window problems). Last-seen approach is faster for this specific problem but only applies when the window shrinks to a specific index.

---

## Problem 3: Max Consecutive Ones III — LC 1004

Flip at most `k` zeroes. Find the longest subarray of ones.

```java
public int longestOnes(int[] nums, int k) {
    int left = 0, zeros = 0, maxLen = 0;

    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeros++;

        while (zeros > k) {          // too many zeros → shrink
            if (nums[left] == 0) zeros--;
            left++;
        }

        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Framing:** "Flip at most k zeroes" = "at most k zeroes in the window." Window shrinks when zero count exceeds k.

---

## Problem 4: Fruits Into Baskets — LC 904

Two baskets (at most 2 distinct fruit types). Find the longest subarray with ≤ 2 distinct values.

```java
public int totalFruit(int[] fruits) {
    Map<Integer, Integer> basket = new HashMap<>();
    int left = 0, maxLen = 0;

    for (int right = 0; right < fruits.length; right++) {
        basket.merge(fruits[right], 1, Integer::sum);  // add fruit

        while (basket.size() > 2) {                    // > 2 types → shrink
            basket.merge(fruits[left], -1, Integer::sum);
            if (basket.get(fruits[left]) == 0) basket.remove(fruits[left]);
            left++;
        }

        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**General pattern:** "At most k distinct elements" — replace `2` with `k`. This is a classic category.

---

## Problem 5: Longest Repeating Character Replacement — LC 424

Replace at most `k` characters. Find the longest substring where you can make all characters the same.

**Key insight:** In a valid window, the number of characters to replace = `window_size - max_frequency`. If this exceeds `k`, shrink.

```java
public int characterReplacement(String s, int k) {
    int[] freq = new int[26];
    int left = 0, maxFreq = 0, maxLen = 0;

    for (int right = 0; right < s.length(); right++) {
        freq[s.charAt(right) - 'A']++;
        maxFreq = Math.max(maxFreq, freq[s.charAt(right) - 'A']);

        // Window size - max_freq = chars to replace
        if ((right - left + 1) - maxFreq > k) {   // shrink
            freq[s.charAt(left) - 'A']--;
            left++;
            // Note: maxFreq is NOT recomputed — see explanation below
        }

        maxLen = Math.max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Why don't we recompute `maxFreq` on shrink?** `maxFreq` can only increase or stay the same to improve the answer. Even if the current window's actual max frequency is lower, we don't need a shorter valid window than what we already have. We're looking for the maximum length, so `maxFreq` serves as a lower bound — any window with a smaller max frequency would be shorter, which isn't useful.

**Formal:** `maxLen = right - left + 1` never shrinks (we shrink by exactly 1 when condition violated, so window size is non-decreasing). `maxFreq` correspondingly can only increase to allow a longer valid window.

**Complexity:** O(n) time, O(1) space (26-element array)

---

## Problem 6: Minimum Window Substring — LC 76

Find the minimum window in `s` containing all characters of `t`.

```java
public String minWindow(String s, String t) {
    if (s.length() < t.length()) return "";

    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;

    int left = 0, have = 0, required = t.length();
    int minLen = Integer.MAX_VALUE, minLeft = 0;

    for (int right = 0; right < s.length(); right++) {
        char c = s.charAt(right);
        if (need[c] > 0) have++;    // we needed this char and now have one more
        need[c]--;                  // reduce need (can go negative for excess chars)

        while (have == required) {   // all chars covered → try to shrink
            if (right - left + 1 < minLen) {
                minLen = right - left + 1;
                minLeft = left;
            }
            char l = s.charAt(left);
            need[l]++;               // return char to need
            if (need[l] > 0) have--; // now we need this char again
            left++;
        }
    }
    return minLen == Integer.MAX_VALUE ? "" : s.substring(minLeft, minLeft + minLen);
}
```

**Why `need[c] > 0` for `have++`?** `need` can be negative (we have excess of a char). We only increment `have` when we're actually satisfying a need (not just adding excess). Similarly, `need[l]++` returns the char; only if it becomes positive does `have--` (we lost a needed char, not just excess).

**Complexity:** O(|s| + |t|) time, O(1) space (128-element array)

---

## Problem 7: Number of Substrings Containing All Three Characters — LC 1358

Count substrings containing at least one each of 'a', 'b', 'c'.

```java
public int numberOfSubstrings(String s) {
    int[] count = new int[3];
    int left = 0, result = 0;

    for (int right = 0; right < s.length(); right++) {
        count[s.charAt(right) - 'a']++;

        while (count[0] > 0 && count[1] > 0 && count[2] > 0) {
            // All three chars present → count all extensions to the right
            result += s.length() - right;   // [left..right], [left..right+1], ..., [left..n-1]
            count[s.charAt(left) - 'a']--;
            left++;
        }
    }
    return result;
}
```

**Key insight:** When `[left..right]` contains all three, then so does `[left..right+1]`, `[left..right+2]`, ..., `[left..n-1]`. All `n - right` extensions are valid from this `left`. After removing `left`, check if still valid.

---

## Problem 8: Longest Subarray of 1s After Deleting One Element — LC 1493

Delete exactly one element. Find longest subarray of 1s.

```java
public int longestSubarray(int[] nums) {
    int left = 0, zeros = 0, maxLen = 0;

    for (int right = 0; right < nums.length; right++) {
        if (nums[right] == 0) zeros++;

        while (zeros > 1) {
            if (nums[left] == 0) zeros--;
            left++;
        }

        // -1 because we must delete exactly one element (the 0 in the window, or any 1)
        maxLen = Math.max(maxLen, right - left);  // right - left = window_size - 1
    }
    return maxLen;
}
```

**Why `right - left` not `right - left + 1`?** We must delete exactly one element. The answer length = window size - 1 (we delete the zero in the window, or any element if window is all ones).

---

## Pattern Summary

| Problem | Window State | Expand Condition | Shrink Condition |
|---------|-------------|-----------------|-----------------|
| Min Subarray Sum | running sum | always | sum ≥ target |
| LSWORC | freq array | always | freq > 1 (duplicate) |
| Max Cons Ones III | zero count | always | zeros > k |
| Fruits Into Baskets | fruit type map | always | map.size() > 2 |
| Char Replacement | freq array + maxFreq | always | windowSize - maxFreq > k |
| Min Window Substring | need array, have count | always | have == required |
| All Three Characters | char counts | always | all 3 counts > 0 |

---

## Related Files

- [Fixed Size Window](./Fixed%20Size%20Window.md)
- [At Most K Trick](./At%20Most%20K%20Trick.md)
- [Two Pointers](./Two%20Pointers.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
