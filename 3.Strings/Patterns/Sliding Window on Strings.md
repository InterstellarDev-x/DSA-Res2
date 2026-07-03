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
5. [C++ Templates](#c-templates)
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

## C++ Templates

### 1. Fixed Window — Find All Anagrams (LC 438)

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> findAnagrams(string s, string p) {
    vector<int> result;
    if (s.length() < p.length()) return result;

    int pFreq[26] = {};
    int wFreq[26] = {};
    for (char c : p) pFreq[c - 'a']++;

    int k = p.length();
    for (int r = 0; r < (int)s.length(); r++) {
        wFreq[s[r] - 'a']++;
        if (r >= k) wFreq[s[r - k] - 'a']--; // remove leftmost
        if (equal(begin(pFreq), end(pFreq), begin(wFreq))) result.push_back(r - k + 1);
    }
    return result;
}
// Time: O(26n) = O(n) | Space: O(1) — arrays of size 26
```

### 2. Variable Window — Longest Substring Without Repeating Chars (LC 3)

```cpp
#include <bits/stdc++.h>
using namespace std;

int lengthOfLongestSubstring(string s) {
    int lastSeen[128];
    fill(lastSeen, lastSeen + 128, -1); // ASCII
    int l = 0, maxLen = 0;

    for (int r = 0; r < (int)s.length(); r++) {
        int c = s[r];
        if (lastSeen[c] >= l) l = lastSeen[c] + 1; // jump past duplicate
        lastSeen[c] = r;
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1) — fixed size array
```

### 3. Variable Window — Minimum Window Substring (LC 76)

```cpp
#include <bits/stdc++.h>
using namespace std;

string minWindow(string s, string t) {
    int need[128] = {};
    for (char c : t) need[c]++;

    int required = (int)unordered_set<char>(t.begin(), t.end()).size();
    int formed = 0, l = 0;
    int ans[3] = {INT_MAX, 0, 0}; // [len, l, r]

    int window[128] = {};
    for (int r = 0; r < (int)s.length(); r++) {
        char rc = s[r];
        window[rc]++;
        if (need[rc] > 0 && window[rc] == need[rc]) formed++;

        while (formed == required) {
            if (r - l + 1 < ans[0]) { ans[0] = r - l + 1; ans[1] = l; ans[2] = r; }
            char lc = s[l++];
            window[lc]--;
            if (need[lc] > 0 && window[lc] < need[lc]) formed--;
        }
    }
    return ans[0] == INT_MAX ? "" : s.substr(ans[1], ans[2] - ans[1] + 1);
}
// Time: O(|s| + |t|) | Space: O(1) — fixed ASCII arrays
```

### 4. Variable Window — Longest Substring with At Most K Distinct Chars (LC 340)

```cpp
#include <bits/stdc++.h>
using namespace std;

int lengthOfLongestSubstringKDistinct(string s, int k) {
    unordered_map<char, int> freq;
    int l = 0, maxLen = 0;

    for (int r = 0; r < (int)s.length(); r++) {
        freq[s[r]]++;
        while ((int)freq.size() > k) {
            char lc = s[l++];
            freq[lc]--;
            if (freq[lc] == 0) freq.erase(lc);
        }
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(k)
```

### 5. Exactly K Distinct = atMost(K) − atMost(K−1)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Count subarrings with exactly K distinct chars
int atMost(string s, int k) {
    int freq[26] = {};
    int l = 0, count = 0, distinct = 0;
    for (int r = 0; r < (int)s.length(); r++) {
        if (freq[s[r] - 'a']++ == 0) distinct++;
        while (distinct > k) {
            if (--freq[s[l++] - 'a'] == 0) distinct--;
        }
        count += r - l + 1; // all substrings ending at r with <= k distinct
    }
    return count;
}

int subarraysWithKDistinct(string s, int k) {
    return atMost(s, k) - atMost(s, k - 1);
}
// Time: O(n) | Space: O(1)
```

### 6. Longest Repeating Character Replacement (LC 424)

```cpp
#include <bits/stdc++.h>
using namespace std;

int characterReplacement(string s, int k) {
    int freq[26] = {};
    int l = 0, maxFreq = 0, maxLen = 0;

    for (int r = 0; r < (int)s.length(); r++) {
        maxFreq = max(maxFreq, ++freq[s[r] - 'a']);
        // window size - maxFreq = number of replacements needed
        while ((r - l + 1) - maxFreq > k) {
            freq[s[l++] - 'a']--;
            // Note: maxFreq may be stale but never decreases incorrectly
        }
        maxLen = max(maxLen, r - l + 1);
    }
    return maxLen;
}
// Time: O(n) | Space: O(1)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| `equal(begin(pFreq), end(pFreq), begin(wFreq))` is O(26) per step | For larger alphabets, use a `matches` counter instead |
| Forgetting to remove key from map when freq hits 0 | `if (freq[c] == 0) freq.erase(c)` — otherwise `size()` is wrong |
| Min window: using `>=` for `formed` increment | Only `==` needed — catches the exact moment a char becomes satisfied |
| Fixed window: off-by-one when removing the outgoing char | Remove `s[r - k]`, not `s[r - k + 1]` |
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
