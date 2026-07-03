> **Topic:** [Advanced Strings](../README.md) · **Section:** Interview Problems

# Amazon — Advanced Strings (Interview Deep Dives)

Amazon's string bar is about **clean, correct, well-explained** solutions that map to the Leadership Principles (especially *Dive Deep* and *Insist on the Highest Standards*). The three headline problems are `strStr`, Longest Palindromic Substring, and Find All Anagrams.

---

## 1. Implement strStr() (LC 28) — naive vs KMP

> Return the index of the first occurrence of `needle` in `haystack`, or `-1`.

### Naive scan

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int strStrNaive(string haystack, string needle) {
        int n = haystack.length(), m = needle.length();
        if (m == 0) return 0;
        for (int i = 0; i + m <= n; i++) {
            int j = 0;
            while (j < m && haystack[i + j] == needle[j]) j++;
            if (j == m) return i;
        }
        return -1;
    }
};
```

O(nm) worst case — consider `haystack = "aaaa...a"`, `needle = "aaa...b"`: each of the `n` start positions re-scans almost all of `needle`. For most real inputs it is fine, but Amazon will probe the worst case.

### KMP — O(n+m)

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle.empty()) return 0;
        vector<int> lps = buildLPS(needle);
        int i = 0, j = 0, n = haystack.length(), m = needle.length();
        while (i < n) {
            if (haystack[i] == needle[j]) {
                i++; j++;
                if (j == m) return i - j;
            } else if (j > 0) {
                j = lps[j - 1];   // reuse the matched border; i does NOT move back
            } else {
                i++;
            }
        }
        return -1;
    }

private:
    vector<int> buildLPS(string s) {
        int n = s.length();
        vector<int> lps(n);
        int len = 0, i = 1;
        while (i < n) {
            if (s[i] == s[len]) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        return lps;
    }
};
```

**When does KMP matter?** Only when (a) library calls are forbidden, (b) the interviewer asks for guaranteed worst-case linear, or (c) you will search the *same* needle in many haystacks (precompute `lps` once). Otherwise `s.find(needle)` is the honest answer — say so, then offer KMP. See the full LPS derivation in [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md).

---

## 2. Longest Palindromic Substring (LC 5) — expand-around-center

> Return the longest substring of `s` that is a palindrome.

Expand-around-center is the **interview-preferred** solution: it is short, obviously correct, and O(1) space. Every palindrome has a center — a single character (odd length) or a gap between two characters (even length) — giving `2n-1` centers. Expand outward from each.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    string longestPalindrome(string s) {
        if (s.empty()) return "";
        int start = 0, end = 0;
        for (int i = 0; i < (int)s.length(); i++) {
            int odd = expand(s, i, i);       // odd-length center at i
            int even = expand(s, i, i + 1);  // even-length center between i and i+1
            int len = max(odd, even);
            if (len > end - start + 1) {
                start = i - (len - 1) / 2;
                end = i + len / 2;
            }
        }
        return s.substr(start, end - start + 1);
    }

private:
    // Returns the length of the palindrome that expands from (l, r).
    int expand(string& s, int l, int r) {
        while (l >= 0 && r < (int)s.length() && s[l] == s[r]) {
            l--; r++;
        }
        return r - l - 1; // characters strictly between the failed bounds
    }
};
```

**Complexity:** O(n²) time, O(1) space.

### When to mention Manacher

Bring up **Manacher's algorithm** only when the interviewer explicitly asks for **O(n)**. It transforms the string with separators (`^#a#b#a#$`) so every palindrome is odd-length, then reuses a mirror to avoid recomputing. It is correct but error-prone under interview pressure — state that you *know* it exists and could implement it, but that expand-around-center is the pragmatic choice unless O(n) is required. Full treatment in [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md).

### Edge cases
- Empty string → `""`.
- Single character → itself.
- All identical characters `"aaaa"` → the whole string (expansion never short-circuits early).

---

## 3. Find All Anagrams in a String (LC 438) — sliding window frequency

> Return the start indices of every substring of `s` that is an anagram of `p`.

This is a **fixed-size sliding window** of length `p.length()` over a 26-letter frequency array. Maintain a running count of how many letters currently have the correct frequency, so each shift is O(1).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> res;
        int n = s.length(), m = p.length();
        if (n < m) return res;

        vector<int> need(26), win(26);
        for (int i = 0; i < m; i++) {
            need[p[i] - 'a']++;
            win[s[i] - 'a']++;
        }
        if (need == win) res.push_back(0);

        for (int i = m; i < n; i++) {
            win[s[i] - 'a']++;         // add the entering char
            win[s[i - m] - 'a']--;     // remove the leaving char
            if (need == win) res.push_back(i - m + 1);
        }
        return res;
    }
};
```

**Complexity:** O(n · 26) = O(n) time, O(26) = O(1) space. The vector `==` comparison over 26 entries is constant; for a stricter O(1)-per-step, track a `matches` counter that increments/decrements as individual letter counts hit/leave equality. Window basis: [Fixed Size Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md).

### Edge cases
- `p` longer than `s` → empty list (the `n < m` guard).
- Repeated answers overlap (`s = "abab", p = "ab"` → `[0,1,2]`); the window naturally finds all.

---

## Amazon Leadership Principle Alignment

| Problem | LP demonstrated | How |
|---------|-----------------|-----|
| strStr (naive → KMP) | **Dive Deep** | Explaining *why* KMP avoids re-scanning, not just coding it |
| Longest Palindromic Substring | **Invent and Simplify** | Choosing expand-around-center over Manacher unless O(n) is required |
| Find All Anagrams | **Insist on the Highest Standards** | O(1)-per-step window instead of recounting each window |
| All three | **Bias for Action** | Stating the library/pragmatic answer first, then the algorithmic one |

---

## Related

- OA frequency: [OA-Qns → Amazon](../OA-Qns/Amazon.md)
- Other deep dives: [Google](Google.md) · [Microsoft](Microsoft.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
