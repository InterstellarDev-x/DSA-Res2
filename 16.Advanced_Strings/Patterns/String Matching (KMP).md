> **Topic:** [Advanced Strings](../README.md) · **Pattern 1 of 3**

# String Matching — KMP (Knuth–Morris–Pratt) & the Prefix Function

The single most important idea in advanced string processing is the **prefix function** (also called the **LPS array** or **failure function**). Master it once and five LeetCode problems collapse into a one-liner on top of it: `strStr`, Longest Happy Prefix, Repeated Substring Pattern, Repeated String Match, and (via `s + reverse(s)`) Shortest Palindrome.

---

## 1. The Prefix Function (LPS array)

For a string `s` of length `n`, define:

> `lps[i]` = the length of the **longest proper prefix** of `s[0..i]` that is **also a suffix** of `s[0..i]`.

- *Proper* means it cannot be the whole substring `s[0..i]` itself (otherwise the answer would trivially be `i+1`).
- "Prefix that is also a suffix" is often abbreviated **PPS** or just **border**. `lps[i]` is the length of the longest border of the prefix ending at index `i`.

### Why it matters

When matching a pattern against a text and a mismatch happens at pattern index `j`, we already know that `s[0..j-1]` matched. The longest border `lps[j-1]` tells us the longest stretch we can *reuse* without re-reading the text. We jump `j` back to `lps[j-1]` instead of restarting at 0 — that is what turns naive O(nm) matching into O(n+m).

### Construction — full C++

```cpp
#include <bits/stdc++.h>
using namespace std;

// lps[i] = length of the longest proper prefix of s[0..i] that is also a suffix.
vector<int> buildLPS(const string& s) {
    int n = s.length();
    vector<int> lps(n);
    lps[0] = 0;            // a single char has no proper prefix
    int len = 0;          // length of the previous longest border
    int i = 1;
    while (i < n) {
        if (s[i] == s[len]) {
            len++;            // extend the current border by one
            lps[i] = len;
            i++;
        } else if (len > 0) {
            // fall back to the longest border of the border (do NOT advance i)
            len = lps[len - 1];
        } else {
            lps[i] = 0;       // no border possible
            i++;
        }
    }
    return lps;
}
```

The crucial line is `len = lps[len - 1]`. On a mismatch we do **not** reset `len` to 0 immediately — we try the next-best border. Only when there is no shorter border (`len == 0`) do we record 0 and move on. **Never advance `i` on the fallback branch** — that is the #1 construction bug.

---

## 2. KMP Search using the LPS array

```cpp
#include <bits/stdc++.h>
using namespace std;

// Returns the first index in text where pattern occurs, or -1.
int kmpSearch(const string& text, const string& pattern) {
    if (pattern.empty()) return 0;
    vector<int> lps = buildLPS(pattern);
    int i = 0; // index into text
    int j = 0; // index into pattern
    int n = text.length(), m = pattern.length();
    while (i < n) {
        if (text[i] == pattern[j]) {
            i++; j++;
            if (j == m) return i - j;          // full match found
        } else if (j > 0) {
            j = lps[j - 1];                    // reuse the matched border; i stays
        } else {
            i++;                               // pattern restart, text advances
        }
    }
    return -1;
}
```

To find **all** occurrences, replace `return i - j;` with: record `i - j`, then continue via `j = lps[j - 1];`.

---

## 3. Problem 1 — Implement strStr() (LC 28, Easy)

> Return the index of the first occurrence of `needle` in `haystack`, or -1.

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
                j = lps[j - 1];
            } else {
                i++;
            }
        }
        return -1;
    }

private:
    vector<int> buildLPS(const string& s) {
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

**Complexity:** O(n + m) time, O(m) space. In C++ `haystack.find(needle) != string::npos` does the same job; use KMP only when the interviewer forbids library calls or asks for the algorithm explicitly.

---

## 4. Problem 4 — Longest Happy Prefix (LC 1392, Hard)

> A *happy prefix* is a non-empty prefix that is also a suffix (excluding the whole string). Return the longest happy prefix.

This is the prefix function **by definition**: the longest border of the whole string is `lps[n-1]`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    string longestPrefix(string s) {
        int n = s.length();
        vector<int> lps(n);
        int len = 0, i = 1;
        while (i < n) {
            if (s[i] == s[len]) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        return s.substr(0, lps[n - 1]);   // longest proper prefix == suffix
    }
};
```

**Complexity:** O(n) time, O(n) space. This is the cleanest demonstration that the prefix function *is* the answer, not just a tool.

---

## 5. Problem 8 — Repeated Substring Pattern (LC 459, Easy)

> Return true if `s` can be built by concatenating copies of some substring of it.

**Key lemma.** Let `n = s.length()` and `k = lps[n-1]`. The string is periodic with smallest period `p = n - k`. The string is built from repeated copies **iff** `k != 0` and `n % p == 0`.

### Why the smallest period works

`lps[n-1] = k` means the length-`k` prefix equals the length-`k` suffix. Overlaying these two borders forces `s[i] == s[i+p]` for all valid `i`, i.e. `p = n - k` is a period of `s`. If `p` divides `n`, the block `s[0..p-1]` tiles the whole string exactly `n/p` times. The `k != 0` guard rules out strings with no border at all (e.g. `"abc"`), where `p = n` would falsely "divide" `n` once.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        int n = s.length();
        vector<int> lps(n);
        int len = 0, i = 1;
        while (i < n) {
            if (s[i] == s[len]) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        int k = lps[n - 1];
        return k != 0 && n % (n - k) == 0;
    }
};
```

**Complexity:** O(n) time, O(n) space.

---

## 6. Problem 2 — Repeated String Match (LC 686, Medium)

> Return the minimum number of times `a` must be repeated so that `b` is a substring of the repeated string, or -1 if impossible.

**Insight.** If `b` is a substring of `a` repeated `q` times, the smallest `q` is at least `ceil(|b| / |a|)`. But `b` may straddle a boundary, so we may need **one more** copy. Therefore only `q` and `q+1` can ever be the answer — test both with a substring/KMP search.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int repeatedStringMatch(string a, string b) {
        string sb = "";
        int count = 0;
        // Append a until the built string is at least as long as b.
        while (sb.length() < b.length()) { sb += a; count++; }
        // b might cross the boundary -> try with one extra copy.
        if (contains(sb, b)) return count;
        sb += a;
        if (contains(sb, b)) return count + 1;
        return -1;
    }

private:
    // KMP membership test (or simply: return text.find(pattern) != string::npos; in an interview).
    bool contains(const string& text, const string& pattern) {
        vector<int> lps = buildLPS(pattern);
        int i = 0, j = 0, n = text.length(), m = pattern.length();
        while (i < n) {
            if (text[i] == pattern[j]) {
                i++; j++;
                if (j == m) return true;
            } else if (j > 0) { j = lps[j - 1]; }
            else { i++; }
        }
        return false;
    }

    vector<int> buildLPS(const string& s) {
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

**Complexity:** O(|a| + |b|) time and space. The KMP membership keeps the search linear; the bound "only `q` or `q+1`" keeps the number of copies minimal.

---

## 7. Dry-run — LPS construction for `"ABABAC"`

We track `(i, len, comparison, lps[i])`. Start `lps[0]=0`, `len=0`, `i=1`.

| i | s[i] | s[len] | branch | len after | lps[i] |
|---|------|--------|--------|-----------|--------|
| 1 | B | A (len 0) | mismatch, len=0 → record 0 | 0 | 0 |
| 2 | A | A (len 0) | match → len 1 | 1 | 1 |
| 3 | B | B (len 1) | match → len 2 | 2 | 2 |
| 4 | A | A (len 2) | match → len 3 | 3 | 3 |
| 5 | C | B (len 3) | mismatch, len>0 → len=lps[2]=2 | 2 | — |
| 5 | C | A (len 2) | mismatch, len>0 → len=lps[1]=0 | 0 | — |
| 5 | C | A (len 0) | mismatch, len=0 → record 0 | 0 | 0 |

Final `lps = [0,0,1,2,3,0]`. Read it: the prefix `ABABA` (index 4) has its longest border `ABA` of length 3 — prefix `ABA` equals suffix `ABA`. The fallback chain at `i=5` (`3 → 2 → 0`) is exactly the "try the next shorter border" logic.

A second classic, `"AABAACAABAA"`, yields `lps = [0,1,0,1,2,0,1,2,3,4,5]`; the final value 5 says the 5-char prefix `AABAA` equals the 5-char suffix — useful when explaining the failure function to an interviewer.

---

## Recognition Signals

| Signal in the prompt | Reach for |
|----------------------|-----------|
| "first occurrence", "index of substring", "implement strStr" | KMP search (or `find` if allowed) |
| "prefix that is also a suffix", "happy prefix", "border" | `lps[n-1]` directly |
| "repeated copies of a substring", "is the string periodic" | period `n - lps[n-1]`, check `n % p == 0` |
| "how many times repeat A to contain B" | build until `>= |b|`, test `q` and `q+1` |
| "shortest palindrome by prepending" | LPS on `s + '#' + reverse(s)` (see [Palindrome Algorithms](Palindrome%20Algorithms.md)) |
| Need O(n+m) and library calls forbidden | KMP over naive scanning |

---

## Cross-links

- [Z Algorithm and Hashing](Z%20Algorithm%20and%20Hashing.md) — the Z-array is an equivalent linear-matcher; `z[i]` and `lps` carry the same information in different coordinates.
- [Palindrome Algorithms](Palindrome%20Algorithms.md) — Shortest Palindrome reuses this exact LPS construction.
- [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md) — the hashing alternative to KMP for matching.

> **Last Updated:** 2026-06-26
