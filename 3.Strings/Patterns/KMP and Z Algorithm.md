# KMP Algorithm & Z-Algorithm

> **Topic:** [Strings](../README.md) · **Difficulty:** Hard
> **Tags:** `KMP` `z-algorithm` `pattern-matching` `failure-function` `LPS`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [KMP — C++ Implementation](#kmp--c-implementation)
6. [Z-Algorithm — C++ Implementation](#z-algorithm--c-implementation)
7. [Common Mistakes](#common-mistakes)
8. [Applications](#applications)
9. [Practice Problems](#practice-problems)
10. [Related Patterns](#related-patterns)

---

## Pattern Overview

**KMP (Knuth-Morris-Pratt):**
Find all occurrences of pattern `p` in text `t` in O(n + m) by pre-computing the **Longest Proper Prefix which is also Suffix (LPS / failure function)** for the pattern. When a mismatch occurs, instead of restarting, the LPS array tells us the furthest valid restart point.

**Z-Algorithm:**
Computes an array `Z[i]` = length of the longest substring starting at `i` that is also a prefix of the string. O(n) construction. Equivalent power to KMP but often simpler to reason about.

```
KMP:  lps[i] = length of longest proper prefix of p[0..i] that is also a suffix
Z:    z[i]   = length of longest string starting from s[i] matching a prefix of s
```

---

## When to Use

- Find occurrences of a pattern in a text — O(n + m)
- Check if string A is a rotation of string B (search in A+A)
- Find shortest palindrome by prepending characters (reverse + KMP)
- Repeated string structure — check if pattern is a repeated unit
- Longest prefix that is also a suffix

---

## Recognition Cues

| Cue | Algorithm |
|-----|-----------|
| "find pattern in string" | KMP or Z |
| "is s a rotation of t?" | Search in `s+s` with KMP |
| "shortest palindrome by prepending" | Reverse + KMP LPS |
| "repeated substring pattern" | KMP LPS: if `n % (n - lps[n-1]) == 0` |
| "longest prefix = suffix" | KMP LPS array directly |
| "count distinct substrings" | [String Hashing](./String%20Hashing.md) |

---

## Complexity

| Algorithm | Build | Search | Space |
|-----------|-------|--------|-------|
| Naive search | — | O(n × m) | O(1) |
| KMP | O(m) | O(n) | O(m) |
| Z-Algorithm | O(n) | O(n) | O(n) |
| Rabin-Karp | O(n) | O(n) avg | O(1) |

---

## KMP — C++ Implementation

### Step 1: Build LPS (Failure Function)

```cpp
#include <bits/stdc++.h>
using namespace std;

// lps[i] = length of longest proper prefix of p[0..i] that is also a suffix
vector<int> buildLPS(const string& pattern) {
    int m = pattern.length();
    vector<int> lps(m);
    int len = 0; // length of previous longest prefix suffix
    int i = 1;

    while (i < m) {
        if (pattern[i] == pattern[len]) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len - 1]; // don't increment i — try shorter prefix
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

### Step 2: KMP Search

```cpp
vector<int> kmpSearch(const string& text, const string& pattern) {
    vector<int> indices;
    int n = text.length(), m = pattern.length();
    if (m == 0) return indices;

    vector<int> lps = buildLPS(pattern);
    int i = 0, j = 0; // i = text index, j = pattern index

    while (i < n) {
        if (text[i] == pattern[j]) {
            i++; j++;
        }
        if (j == m) {
            indices.push_back(i - j); // found at i - m
            j = lps[j - 1];    // continue with partial match
        } else if (i < n && text[i] != pattern[j]) {
            if (j > 0) j = lps[j - 1]; // use LPS to skip
            else i++;
        }
    }
    return indices;
}
// Total: O(n + m) | Space: O(m)
```

### Application 1: Repeated Substring Pattern (LC 459)

```cpp
bool repeatedSubstringPattern(const string& s) {
    int n = s.length();
    vector<int> lps = buildLPS(s);
    int len = lps[n - 1];
    // If len > 0 and n is divisible by (n - len), s is built from a repeating unit
    return len > 0 && n % (n - len) == 0;
}
// Example: "abababab" → lps[7]=6 → n%(n-6) = 8%2 = 0 → true
```

### Application 2: Shortest Palindrome by Prepending (LC 214)

```cpp
string shortestPalindrome(const string& s) {
    // Find longest palindromic prefix using KMP
    string rev = s;
    reverse(rev.begin(), rev.end());
    string combined = s + "#" + rev; // '#' prevents overlap
    vector<int> lps = buildLPS(combined);
    int longestPalinPrefix = lps[combined.length() - 1];
    return rev.substr(0, s.length() - longestPalinPrefix) + s;
}
// Time: O(n) | Space: O(n)
```

### Application 3: Is Rotation? (LC 796)

```cpp
bool isRotation(const string& s, const string& goal) {
    if (s.length() != goal.length()) return false;
    return !kmpSearch(s + s, goal).empty();
}
// Time: O(n) | Space: O(n)
```

---

## Z-Algorithm — C++ Implementation

```cpp
#include <bits/stdc++.h>
using namespace std;

// z[i] = length of longest substring starting at s[i] that matches a prefix of s
// z[0] = 0 by convention (or n, depending on definition)
vector<int> zFunction(const string& s) {
    int n = s.length();
    vector<int> z(n);
    int l = 0, r = 0;

    for (int i = 1; i < n; i++) {
        if (i < r) {
            z[i] = min(r - i, z[i - l]);
        }
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) {
            z[i]++;
        }
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}
```

### Z-Algorithm Pattern Search

```cpp
vector<int> zSearch(const string& text, const string& pattern) {
    string combined = pattern + "$" + text; // '$' not in alphabet
    vector<int> z = zFunction(combined);
    vector<int> result;
    int m = pattern.length();
    for (int i = m + 1; i < (int)combined.length(); i++) {
        if (z[i] == m) result.push_back(i - m - 1); // offset in text
    }
    return result;
}
// Time: O(n + m) | Space: O(n + m)
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| KMP LPS: using `i++` after `len = lps[len-1]` | Do NOT increment `i` when falling back — re-check same `i` |
| KMP search: resetting `j = 0` on mismatch | Use `j = lps[j-1]` — preserves partial match information |
| Rotation check: `s + s` without `#` separator | `shortestPalindrome` must have separator to prevent LPS crossing boundary |
| Z-function: `z[0]` definition varies | Some define `z[0] = n`, some = 0 — be explicit |
| Repeated substring: checking `len > 0` only | Must also check `n % (n - len) == 0` |

---

## Applications

| Problem | Algorithm | Key Step |
|---------|-----------|----------|
| Pattern matching | KMP or Z | Standard |
| Rotation check | KMP | Search pattern in `s+s` |
| Shortest palindrome | KMP | LPS of `s + "#" + reverse(s)` |
| Repeated unit | KMP LPS | `n % (n - lps[n-1]) == 0` |
| Longest happy prefix | KMP LPS | LPS array value at last index |
| Count pattern occurrences | KMP | Count times `j == m` |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Find the Index of the First Occurrence](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | LC 28 |
| [Repeated Substring Pattern](https://leetcode.com/problems/repeated-substring-pattern/) | Easy | LC 459 |
| [Shortest Palindrome](https://leetcode.com/problems/shortest-palindrome/) | Hard | LC 214 |
| [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/) | Hard | LC 1392 |
| [String Matching in an Array](https://leetcode.com/problems/string-matching-in-an-array/) | Easy | LC 1408 |
| [Rotate String](https://leetcode.com/problems/rotate-string/) | Easy | LC 796 |
| Minimum Characters to Make Palindrome | Medium | GFG |

---

## Related Patterns

- [String Hashing](./String%20Hashing.md) — alternative O(n) pattern matching
- [Two Pointers on Strings](./Two%20Pointers%20on%20Strings.md) — palindrome problems
- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — window-based pattern search

---

> **Interview Tip:** KMP is rarely asked "implement KMP from scratch" in product-company interviews (Google/Meta/Amazon). It IS asked at quant/competitive firms (Jane Street, Citadel). Know it conceptually and be able to code the LPS array. More commonly, interviewers ask applications: repeated substring, rotation, shortest palindrome — all are KMP in disguise.

> **Last Updated:** 2026-06-26
