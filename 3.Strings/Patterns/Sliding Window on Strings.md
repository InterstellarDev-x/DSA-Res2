# Sliding Window on Strings

> **Topic:** [Strings](../README.md) · **Difficulty:** Medium–Hard
> **Tags:** `sliding-window` `frequency-map` `substring` `anagram`
> **See also:** [Arrays Sliding Window](../../1.Arrays/Patterns/Sliding%20Window.md)

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

Sliding Window on strings uses a **frequency map** (usually `int[26]` for lowercase letters) instead of a numeric sum. The window expands right on each step; when the window violates a constraint, it shrinks from the left.

**Two flavors:**
| Flavor | Shrink condition | Goal |
|--------|-----------------|------|
| **Fixed window** | size > k | Find matching window |
| **Variable window** | constraint violated | Maximize or minimize length |

**Critical string-specific difference from numeric arrays:**
- The invariant tracks a `formed` counter (how many chars satisfy their required frequency) rather than a running sum.
- This avoids scanning the entire map on every shrink step — O(26) per step vs O(1).

---

## When to Use

- Longest/shortest substring with some character-count constraint
- Find all anagrams / permutations of a pattern in a text
- Minimum window containing all chars of a pattern
- Substring with at most/exactly K distinct chars

---

## Recognition Cues

| Phrase | Pattern |
|--------|---------|
| "find all anagrams of p in s" | Fixed window = `len(p)`, char freq comparison |
| "longest substring without repeating characters" | Variable window + `lastSeen` map |
| "minimum window substring" | Variable window + `formed` counter |
| "at most K distinct characters" | Variable window + freq map size |
| "permutation of s in t" | Fixed window = `len(s)`, freq match |

---

## Complexity

| Variant | Time | Space |
|---------|------|-------|
| Fixed window (freq array) | O(n) | O(1) — `int[26]` |
| Variable window (freq map) | O(n) | O(charset) |
| Min window (formed counter) | O(n + m) | O(charset) |

---

## Java Templates

### 1. Fixed Window — Find All Anagrams (LC 438)

```java
public List<Integer> findAnagrams(String s, String p) {
    List<Integer> result = new ArrayList<>();
    if (s.length() < p.length()) return result;

    int[] pFreq = new int[26];
    int[] wFreq = new int[26];
    for (char c : p.toCharArray()) pFreq[c - 'a']++;

    int k = p.length();
    for (int r = 0; r < s.length(); r++) {
        wFreq[s.charAt(r) - 'a']++;
        if (r >= k) wFreq[s.charAt(r - k) - 'a']--; // remove leftmost
        if (Arrays.equals(pFreq, wFreq)) result.add(r - k + 1);
    }
    return result;
}
// Time: O(26n) = O(n) | Space: O(1) — arrays of size 26
```

### 2. Variable Window — Longest Substring Without Repeating Chars (LC 3)

```java
public int lengthOfLongestSubstring(String s) {
    int[] lastSeen = new int[128]; // ASCII
    Arrays.fill(lastSeen, -1);
    int l = 0, maxLen = 0;

    for (int r = 0; r < s.length(); r++) {
        int c = s.charAt(r);
        if (lastSeen[c] >= l) l = lastSeen[c] + 1; // jump past duplicate
        lastSeen[c] = r;
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1) — fixed size array
```

### 3. Variable Window — Minimum Window Substring (LC 76)

```java
public String minWindow(String s, String t) {
    int[] need = new int[128];
    for (char c : t.toCharArray()) need[c]++;

    int required = (int) t.chars().distinct().count();
    int formed = 0, l = 0;
    int[] ans = {Integer.MAX_VALUE, 0, 0}; // [len, l, r]

    int[] window = new int[128];
    for (int r = 0; r < s.length(); r++) {
        char rc = s.charAt(r);
        window[rc]++;
        if (need[rc] > 0 && window[rc] == need[rc]) formed++;

        while (formed == required) {
            if (r - l + 1 < ans[0]) { ans[0] = r - l + 1; ans[1] = l; ans[2] = r; }
            char lc = s.charAt(l++);
            window[lc]--;
            if (need[lc] > 0 && window[lc] < need[lc]) formed--;
        }
    }
    return ans[0] == Integer.MAX_VALUE ? "" : s.substring(ans[1], ans[2] + 1);
}
// Time: O(|s| + |t|) | Space: O(1) — fixed ASCII arrays
```

### 4. Variable Window — Longest Substring with At Most K Distinct Chars (LC 340)

```java
public int lengthOfLongestSubstringKDistinct(String s, int k) {
    Map<Character, Integer> freq = new HashMap<>();
    int l = 0, maxLen = 0;

    for (int r = 0; r < s.length(); r++) {
        freq.merge(s.charAt(r), 1, Integer::sum);
        while (freq.size() > k) {
            char lc = s.charAt(l++);
            freq.merge(lc, -1, Integer::sum);
            if (freq.get(lc) == 0) freq.remove(lc);
        }
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(k)
```

### 5. Exactly K Distinct = atMost(K) − atMost(K−1)

```java
// Count subarrings with exactly K distinct chars
public int subarraysWithKDistinct(String s, int k) {
    return atMost(s, k) - atMost(s, k - 1);
}

private int atMost(String s, int k) {
    int[] freq = new int[26];
    int l = 0, count = 0, distinct = 0;
    for (int r = 0; r < s.length(); r++) {
        if (freq[s.charAt(r) - 'a']++ == 0) distinct++;
        while (distinct > k) {
            if (--freq[s.charAt(l++) - 'a'] == 0) distinct--;
        }
        count += r - l + 1; // all substrings ending at r with ≤ k distinct
    }
    return count;
}
// Time: O(n) | Space: O(1)
```

### 6. Longest Repeating Character Replacement (LC 424)

```java
public int characterReplacement(String s, int k) {
    int[] freq = new int[26];
    int l = 0, maxFreq = 0, maxLen = 0;

    for (int r = 0; r < s.length(); r++) {
        maxFreq = Math.max(maxFreq, ++freq[s.charAt(r) - 'a']);
        // window size - maxFreq = number of replacements needed
        while ((r - l + 1) - maxFreq > k) {
            freq[s.charAt(l++) - 'a']--;
            // Note: maxFreq may be stale but never decreases incorrectly
        }
        maxLen = Math.max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `Arrays.equals(pFreq, wFreq)` is O(26) per step | For larger alphabets, use a `matches` counter instead |
| Forgetting to remove key from map when freq hits 0 | `if (freq.get(c) == 0) freq.remove(c)` — otherwise `size()` is wrong |
| Min window: using `>=` for `formed` increment | Only `==` needed — catches the exact moment a char becomes satisfied |
| Fixed window: off-by-one when removing the outgoing char | Remove `s.charAt(r - k)`, not `s.charAt(r - k + 1)` |
| Counting `lastSeen[c] >= 0` instead of `>= l` | Without `>= l` guard, stale entries cause wrong left boundary jumps |

---

## Variations

| Variation | Key Change |
|-----------|-----------|
| Permutation in string (LC 567) | Fixed window, same as anagram but return bool |
| Minimum window with at most duplicates | Modify formed counter logic |
| Max vowels in substring of length k | Fixed window, count vowels |
| Binary string max ones after k flips | Window with at most k zeros |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Longest Substring Without Repeating Characters](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Medium | LC 3 |
| [Minimum Window Substring](https://leetcode.com/problems/minimum-window-substring/) | Hard | LC 76 |
| [Find All Anagrams in a String](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Medium | LC 438 |
| [Permutation in String](https://leetcode.com/problems/permutation-in-string/) | Medium | LC 567 |
| [Longest Repeating Character Replacement](https://leetcode.com/problems/longest-repeating-character-replacement/) | Medium | LC 424 |
| [Subarrays with K Different Integers](https://leetcode.com/problems/subarrays-with-k-different-integers/) | Hard | LC 992 |
| [Longest Substring with At Most Two Distinct Characters](https://leetcode.com/problems/longest-substring-with-at-most-two-distinct-characters/) | Medium | LC 159 |
| [Max Consecutive Ones III](https://leetcode.com/problems/max-consecutive-ones-iii/) | Medium | LC 1004 |

---

## Related Patterns

- [Arrays Sliding Window](../../1.Arrays/Patterns/Sliding%20Window.md) — numeric version
- [Two Pointers on Strings](./Two%20Pointers%20on%20Strings.md) — single-pointer problems
- [Frequency Count](./Frequency%20Count.md) — static frequency comparison

---

> **Interview Tip:** For the `formed` counter pattern (min window), the key insight is: increment `formed` only when `window[c] == need[c]` (not `>=`). This gives O(1) per step instead of scanning the whole frequency map.

> **Last Updated:** 2026-06-26
