> **Topic:** [Dynamic Programming](../README.md) · **Pattern 8 of 8**

# DP on Strings

String DP problems revolve around comparing, matching, partitioning, or transforming one or more strings. The defining trait is a **state indexed by positions** within the string(s) — typically `dp[i][j]` describing the relationship between a prefix of length `i` of one string and a prefix of length `j` of another, or `dp[i]` describing a property of the first `i` characters of a single string. From these states we derive transitions by inspecting one character at a time (`s[i - 1]`), branching on equality, special pattern symbols (`?`, `*`, `.`), or dictionary membership.

This pattern is the natural extension of [Longest Common Subsequence](./Longest%20Common%20Subsequence.md): once you internalize the `i`/`j` prefix-state formulation, edit distance, wildcard matching, regex matching, and interleaving all become variations on the same skeleton. Single-string problems (word break, decode ways, palindromic substrings) reuse the prefix idea with a 1D state or a 2D substring table `dp[i][j]` over the span `[i, j]`.

This document covers eight canonical problems (48–55). Each section gives the **state definition**, **recurrence**, **base cases**, a **complexity table**, full compilable C++ (recursion → memoization → tabulation → space optimization where relevant), and at least one worked dry-run.

---

## 1. Edit Distance (LC 72)

Given two strings `word1` and `word2`, return the minimum number of operations — **insert**, **delete**, or **replace** a single character — to convert `word1` into `word2`.

### State

`dp[i][j]` = minimum edits to convert the first `i` characters of `word1` into the first `j` characters of `word2`.

### Recurrence

Let `m = word1.size()`, `n = word2.size()`. Consider the last characters `word1[i - 1]` and `word2[j - 1]`:

- If they match: `dp[i][j] = dp[i - 1][j - 1]` (no operation needed).
- Otherwise take the best of three operations:
  - **Replace:** `dp[i - 1][j - 1] + 1`
  - **Delete** from `word1`: `dp[i - 1][j] + 1`
  - **Insert** into `word1`: `dp[i][j - 1] + 1`

### Base Cases

- `dp[0][j] = j` — empty `word1` needs `j` insertions.
- `dp[i][0] = i` — emptying `word1` needs `i` deletions.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Recursion (naive) | O(3^(m+n)) | O(m + n) stack |
| Memoization | O(m · n) | O(m · n) + O(m + n) stack |
| Tabulation | O(m · n) | O(m · n) |
| 1D space optimized | O(m · n) | O(n) |

### C++ — Recursion

```cpp
#include <bits/stdc++.h>
using namespace std;

class EditDistanceRecursion {
public:
    int minDistance(string word1, string word2) {
        return solve(word1, word2, word1.size(), word2.size());
    }

private:
    int solve(const string& w1, const string& w2, int i, int j) {
        if (i == 0) return j; // insert remaining j chars
        if (j == 0) return i; // delete remaining i chars

        if (w1[i - 1] == w2[j - 1]) {
            return solve(w1, w2, i - 1, j - 1);
        }
        int replace = solve(w1, w2, i - 1, j - 1);
        int del = solve(w1, w2, i - 1, j);
        int ins = solve(w1, w2, i, j - 1);
        return 1 + min(replace, min(del, ins));
    }
};
```

### C++ — Memoization

```cpp
#include <bits/stdc++.h>
using namespace std;

class EditDistanceMemo {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, -1));
        return solve(word1, word2, m, n, dp);
    }

private:
    int solve(const string& w1, const string& w2, int i, int j, vector<vector<int>>& dp) {
        if (i == 0) return j;
        if (j == 0) return i;
        if (dp[i][j] != -1) return dp[i][j];

        if (w1[i - 1] == w2[j - 1]) {
            return dp[i][j] = solve(w1, w2, i - 1, j - 1, dp);
        }
        int replace = solve(w1, w2, i - 1, j - 1, dp);
        int del = solve(w1, w2, i - 1, j, dp);
        int ins = solve(w1, w2, i, j - 1, dp);
        return dp[i][j] = 1 + min(replace, min(del, ins));
    }
};
```

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class EditDistanceTabulation {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1, 0));

        for (int j = 0; j <= n; j++) dp[0][j] = j; // base row
        for (int i = 0; i <= m; i++) dp[i][0] = i; // base column

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    int replace = dp[i - 1][j - 1];
                    int del = dp[i - 1][j];
                    int ins = dp[i][j - 1];
                    dp[i][j] = 1 + min(replace, min(del, ins));
                }
            }
        }
        return dp[m][n];
    }
};
```

### C++ — 1D Space Optimized

```cpp
#include <bits/stdc++.h>
using namespace std;

class EditDistanceSpaceOptimized {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<int> prev(n + 1), curr(n + 1);

        for (int j = 0; j <= n; j++) prev[j] = j; // dp[0][j]

        for (int i = 1; i <= m; i++) {
            curr[0] = i; // dp[i][0]
            for (int j = 1; j <= n; j++) {
                if (word1[i - 1] == word2[j - 1]) {
                    curr[j] = prev[j - 1];
                } else {
                    curr[j] = 1 + min(prev[j - 1], min(prev[j], curr[j - 1]));
                }
            }
            swap(prev, curr); // reuse the array
        }
        return prev[n];
    }
};
```

### Dry Run

`word1 = "horse"`, `word2 = "ros"`. The filled table (`dp[i][j]`, rows = prefixes of `"horse"`, columns = prefixes of `"ros"`):

|       | "" | r | o | s |
|-------|----|---|---|---|
| **""**| 0  | 1 | 2 | 3 |
| **h** | 1  | 1 | 2 | 3 |
| **o** | 2  | 2 | 1 | 2 |
| **r** | 3  | 2 | 2 | 2 |
| **s** | 4  | 3 | 3 | 2 |
| **e** | 5  | 4 | 4 | 3 |

Answer = `dp[5][3] = 3`. The path: replace `h→r`, replace `o→o` (free), delete `r`, keep `s`, delete `e` — three operations (`horse → rorse → rose → ros`).

---

## 2. Wildcard Matching (LC 44)

Given an input string `s` and a pattern `p` containing `?` and `*`, return whether `p` matches the **entire** string `s`:

- `?` matches any single character.
- `*` matches any sequence of characters (including the empty sequence).

### State

`dp[i][j]` = whether the first `i` characters of `s` match the first `j` characters of `p`.

### Recurrence

Inspect `p[j - 1]`:

- If it is a normal char or `?`: it must consume one char of `s` —
  `dp[i][j] = dp[i - 1][j - 1] && (p[j-1] == '?' || p[j-1] == s[i-1])`.
- If it is `*`: it matches **empty** (`dp[i][j - 1]`) **or** absorbs one more char of `s` (`dp[i - 1][j]`):
  `dp[i][j] = dp[i][j - 1] || dp[i - 1][j]`.

### Base Cases

- `dp[0][0] = true` — empty pattern matches empty string.
- `dp[i][0] = false` for `i > 0` — non-empty string cannot match empty pattern.
- **Leading `*`s:** `dp[0][j] = dp[0][j - 1] && p[j - 1] == '*'`. An empty string matches a prefix of pattern only if that prefix is all `*`s (each matching the empty sequence).

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(m · n) | O(m · n) |

(where `m = s.size()`, `n = p.size()`)

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class WildcardMatching {
public:
    bool isMatch(string s, string p) {
        int m = s.size(), n = p.size();
        vector<vector<bool>> dp(m + 1, vector<bool>(n + 1, false));

        dp[0][0] = true; // empty matches empty
        for (int j = 1; j <= n; j++) {
            // leading '*' can match empty prefix of s
            dp[0][j] = dp[0][j - 1] && p[j - 1] == '*';
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                char pc = p[j - 1];
                if (pc == '*') {
                    // '*' = empty (dp[i][j-1]) OR absorb s[i-1] (dp[i-1][j])
                    dp[i][j] = dp[i][j - 1] || dp[i - 1][j];
                } else if (pc == '?' || pc == s[i - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = false;
                }
            }
        }
        return dp[m][n];
    }
};
```

### Dry Run

`s = "adceb"`, `p = "*a*b"`. The leading `*` makes `dp[0][1] = true`. Walking the table, `dp[5][4]` resolves to `true`: the first `*` matches `""`, `a` matches `a`, the second `*` absorbs `dce`, and `b` matches `b`.

---

## 3. Regular Expression Matching (LC 10)

Given an input string `s` and a pattern `p` with `.` and `*`, return whether `p` matches the **entire** string `s`:

- `.` matches any single character.
- `*` matches **zero or more** of the **preceding** element.

The key difference from wildcard `*`: here `*` modifies the character immediately before it, rather than standing alone.

### State

`dp[i][j]` = whether the first `i` characters of `s` match the first `j` characters of `p`.

### Recurrence

Inspect `p[j - 1]`:

- Normal char or `.`: `dp[i][j] = dp[i - 1][j - 1] && matches(s[i-1], p[j-1])` where `matches` is true when the pattern char is `.` or equals the string char.
- `*` (with preceding char `p[j - 2]`):
  - **Zero occurrences:** drop the pair `x*` → `dp[i][j - 2]`.
  - **One or more occurrences:** if the preceding pattern char matches `s[i - 1]`, consume that char → `dp[i - 1][j]`.
  - Combine: `dp[i][j] = dp[i][j - 2] || (matches(s[i-1], p[j-2]) && dp[i - 1][j])`.

### Base Cases

- `dp[0][0] = true`.
- `dp[i][0] = false` for `i > 0`.
- **Patterns like `a*`, `a*b*`** can match the empty string: for `j >= 2`, if `p[j - 1] == '*'` then `dp[0][j] = dp[0][j - 2]`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Memoization | O(m · n) | O(m · n) + stack |
| Tabulation | O(m · n) | O(m · n) |

### C++ — Memoization

```cpp
#include <bits/stdc++.h>
using namespace std;

class RegexMatchingMemo {
    // 0 = unknown, 1 = true, -1 = false (since bool cannot hold "unset")
    vector<vector<int>> memo;

public:
    bool isMatch(string s, string p) {
        memo.assign(s.size() + 1, vector<int>(p.size() + 1, 0));
        return solve(s, p, 0, 0);
    }

private:
    bool solve(const string& s, const string& p, int i, int j) {
        if (j == (int)p.size()) return i == (int)s.size();
        if (memo[i][j] != 0) return memo[i][j] == 1;

        bool firstMatch = i < (int)s.size()
                && (p[j] == '.' || p[j] == s[i]);

        bool result;
        if (j + 1 < (int)p.size() && p[j + 1] == '*') {
            // zero occurrences (skip "x*") OR one+ (consume s[i])
            result = solve(s, p, i, j + 2)
                    || (firstMatch && solve(s, p, i + 1, j));
        } else {
            result = firstMatch && solve(s, p, i + 1, j + 1);
        }

        memo[i][j] = result ? 1 : -1;
        return result;
    }
};
```

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class RegexMatchingTabulation {
public:
    bool isMatch(string s, string p) {
        int m = s.size(), n = p.size();
        vector<vector<bool>> dp(m + 1, vector<bool>(n + 1, false));

        dp[0][0] = true;
        // patterns like a*, a*b* may match empty string
        for (int j = 2; j <= n; j++) {
            if (p[j - 1] == '*') {
                dp[0][j] = dp[0][j - 2];
            }
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                char pc = p[j - 1];
                if (pc == '*') {
                    char prev = p[j - 2];
                    // zero occurrences of the preceding element
                    dp[i][j] = dp[i][j - 2];
                    // one or more occurrences
                    if (prev == '.' || prev == s[i - 1]) {
                        dp[i][j] = dp[i][j] || dp[i - 1][j];
                    }
                } else if (pc == '.' || pc == s[i - 1]) {
                    dp[i][j] = dp[i - 1][j - 1];
                } else {
                    dp[i][j] = false;
                }
            }
        }
        return dp[m][n];
    }
};
```

### Dry Run

`s = "aab"`, `p = "c*a*b"`. `c*` matches zero `c`s (`dp[0][2] = true`), `a*` matches `aa`, `b` matches `b`. The chain of true cells leads to `dp[3][5] = true`.

---

## 4. Word Break (LC 139)

Given a string `s` and a dictionary `wordDict`, return whether `s` can be segmented into a space-separated sequence of one or more dictionary words.

### State

`dp[i]` = whether the prefix `s.substr(0, i)` (length `i`) can be segmented entirely using dictionary words.

### Recurrence

For each end position `i`, try every split point `j < i`:

`dp[i] = OR over j in [0, i) of ( dp[j] && dictionary.count(s.substr(j, i - j)) )`.

In words: the prefix of length `i` is breakable if some earlier prefix of length `j` is breakable AND the remaining segment `s[j..i)` is a dictionary word.

### Base Case

- `dp[0] = true` — the empty prefix is trivially segmentable.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation (unordered_set lookup) | O(n^2 · L) | O(n + dict) |

`n = s.size()`, `L` = average word length (cost of substring + hashing).

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class WordBreak {
public:
    bool wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> dictionary(wordDict.begin(), wordDict.end());
        int n = s.size();
        vector<bool> dp(n + 1, false);
        dp[0] = true; // empty prefix

        for (int i = 1; i <= n; i++) {
            for (int j = 0; j < i; j++) {
                if (dp[j] && dictionary.count(s.substr(j, i - j))) {
                    dp[i] = true;
                    break; // one valid split is enough
                }
            }
        }
        return dp[n];
    }
};
```

### Dry Run

`s = "leetcode"`, `wordDict = ["leet", "code"]`.

- `dp[0] = true`.
- `dp[4]`: `dp[0] && "leet" in dict` → `true`.
- `dp[8]`: `dp[4] && "code" in dict` → `true`.

Answer = `dp[8] = true`.

---

## 5. Decode Ways (LC 91)

A message of digits is encoded with `A → 1`, ..., `Z → 26`. Given a digit string `s`, return the number of ways to decode it. A leading `'0'` decodes to nothing valid, so it must be handled carefully.

### State

`dp[i]` = number of ways to decode the prefix `s.substr(0, i)` (length `i`).

### Recurrence

For position `i` (1-indexed over characters):

- **One-digit decode:** if `s[i - 1]` is `'1'..'9'`, add `dp[i - 1]`.
- **Two-digit decode:** if the substring `s[i-2..i)` (i.e. chars at `i-2` and `i-1`) forms a value in `[10, 26]`, add `dp[i - 2]`.

`dp[i] = (validOne ? dp[i-1] : 0) + (validTwo ? dp[i-2] : 0)`.

A character `'0'` can never decode as a single digit and is only valid as the second digit of `10` or `20`.

### Base Cases

- `dp[0] = 1` — empty string has one (empty) decoding.
- `dp[1] = (s[0] == '0') ? 0 : 1`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(n) | O(n) |
| O(1) space variant | O(n) | O(1) |

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class DecodeWays {
public:
    int numDecodings(string s) {
        int n = s.size();
        if (n == 0 || s[0] == '0') return 0;

        vector<int> dp(n + 1, 0);
        dp[0] = 1;
        dp[1] = 1; // s[0] is guaranteed != '0' here

        for (int i = 2; i <= n; i++) {
            char one = s[i - 1];
            char tens = s[i - 2];

            if (one != '0') {                 // single-digit decode 1..9
                dp[i] += dp[i - 1];
            }
            int twoDigit = (tens - '0') * 10 + (one - '0');
            if (twoDigit >= 10 && twoDigit <= 26) { // two-digit decode 10..26
                dp[i] += dp[i - 2];
            }
        }
        return dp[n];
    }
};
```

### C++ — O(1) Space

```cpp
#include <bits/stdc++.h>
using namespace std;

class DecodeWaysSpaceOptimized {
public:
    int numDecodings(string s) {
        int n = s.size();
        if (n == 0 || s[0] == '0') return 0;

        int prev2 = 1;        // dp[i-2]
        int prev1 = 1;        // dp[i-1], i.e. dp[1]

        for (int i = 2; i <= n; i++) {
            int curr = 0;
            char one = s[i - 1];
            char tens = s[i - 2];

            if (one != '0') {
                curr += prev1;
            }
            int twoDigit = (tens - '0') * 10 + (one - '0');
            if (twoDigit >= 10 && twoDigit <= 26) {
                curr += prev2;
            }
            prev2 = prev1;
            prev1 = curr;
        }
        return prev1;
    }
};
```

### Dry Run

`s = "226"`.

- `dp[0] = 1`, `dp[1] = 1` (`"2"`).
- `dp[2]`: `"2"` valid (+`dp[1]`=1) and `"22"` in [10,26] (+`dp[0]`=1) → `2`.
- `dp[3]`: `"6"` valid (+`dp[2]`=2) and `"26"` in [10,26] (+`dp[1]`=1) → `3`.

Answer = `dp[3] = 3` (`"BBF"`, `"BZ"`, `"VF"`).

---

## 6. Palindromic Substrings (LC 647, count)

Given a string `s`, return the **number** of palindromic substrings (substrings that read the same forwards and backwards; different positions count separately).

### Approach A — Expand Around Center

Every palindrome has a center: either a single character (odd length, `n` centers) or a gap between two characters (even length, `n - 1` centers). For each of the `2n - 1` centers, expand outward while the boundary characters match, counting one palindrome per successful expansion.

### Approach B — DP Boolean Table

`dp[i][j]` = whether `s.substr(i, j - i + 1)` is a palindrome.

Recurrence: `dp[i][j] = (s[i] == s[j]) && (j - i < 2 || dp[i + 1][j - 1])`.

The condition `j - i < 2` covers length-1 (`i == j`) and length-2 (`j == i + 1`) substrings, which need no inner check. Iterate `i` from high to low so that `dp[i + 1][...]` is already computed.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Expand around center | O(n^2) | O(1) |
| DP boolean table | O(n^2) | O(n^2) |

### C++ — Expand Around Center

```cpp
#include <bits/stdc++.h>
using namespace std;

class PalindromicSubstringsExpand {
public:
    int countSubstrings(string s) {
        int count = 0;
        for (int center = 0; center < (int)s.size(); center++) {
            count += expand(s, center, center);     // odd length
            count += expand(s, center, center + 1);  // even length
        }
        return count;
    }

private:
    int expand(const string& s, int left, int right) {
        int count = 0;
        while (left >= 0 && right < (int)s.size()
                && s[left] == s[right]) {
            count++;
            left--;
            right++;
        }
        return count;
    }
};
```

### C++ — DP Boolean Table

```cpp
#include <bits/stdc++.h>
using namespace std;

class PalindromicSubstringsDP {
public:
    int countSubstrings(string s) {
        int n = s.size();
        vector<vector<bool>> dp(n, vector<bool>(n, false));
        int count = 0;

        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                if (s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    count++;
                }
            }
        }
        return count;
    }
};
```

### Dry Run

`s = "aaa"`. Palindromes: `"a"`(x3), `"aa"`(x2), `"aaa"`(x1) → total `6`. Expand-around-center: center 0 odd → 1; center 0/1 even → 1; center 1 odd → 2 (`"a"`, `"aaa"`); center 1/2 even → 1; center 2 odd → 1. Sum = `1+1+2+1+1 = 6`.

---

## 7. Longest Palindromic Substring (LC 5)

Given a string `s`, return the longest palindromic substring.

### Approach A — Expand Around Center (preferred: O(1) space)

Same `2n - 1` centers as above, but instead of counting, track the longest matched `[start, end]` window and return that substring at the end.

### Approach B — DP Table

`dp[i][j]` = whether `s.substr(i, j - i + 1)` is a palindrome (same recurrence as Section 6). While filling, record the longest `(i, j)` span where `dp[i][j]` is true.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Expand around center | O(n^2) | O(1) |
| DP table | O(n^2) | O(n^2) |

### C++ — Expand Around Center

```cpp
#include <bits/stdc++.h>
using namespace std;

class LongestPalindromeExpand {
    int start = 0;
    int maxLen = 0;

public:
    string longestPalindrome(string s) {
        if (s.empty()) return "";
        for (int center = 0; center < (int)s.size(); center++) {
            expand(s, center, center);      // odd length
            expand(s, center, center + 1);  // even length
        }
        return s.substr(start, maxLen);
    }

private:
    void expand(const string& s, int left, int right) {
        while (left >= 0 && right < (int)s.size()
                && s[left] == s[right]) {
            left--;
            right++;
        }
        int len = right - left - 1; // window after the failing step
        if (len > maxLen) {
            maxLen = len;
            start = left + 1;
        }
    }
};
```

### C++ — DP Table

```cpp
#include <bits/stdc++.h>
using namespace std;

class LongestPalindromeDP {
public:
    string longestPalindrome(string s) {
        int n = s.size();
        if (n < 2) return s;

        vector<vector<bool>> dp(n, vector<bool>(n, false));
        int bestStart = 0, bestLen = 1;

        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                if (s[i] == s[j] && (j - i < 2 || dp[i + 1][j - 1])) {
                    dp[i][j] = true;
                    if (j - i + 1 > bestLen) {
                        bestLen = j - i + 1;
                        bestStart = i;
                    }
                }
            }
        }
        return s.substr(bestStart, bestLen);
    }
};
```

### Dry Run

`s = "babad"`. Expanding at center index 1 (`'a'`) odd gives `"bab"` (len 3). Center index 2 (`'b'`) odd gives `"aba"` (len 3). Either is a valid answer; the algorithm keeps the first longest found, `"bab"`.

---

## 8. Interleaving String (LC 97)

Given `s1`, `s2`, and `s3`, return whether `s3` is formed by an interleaving of `s1` and `s2` — i.e. `s3` can be split into pieces drawn alternately (in order) from `s1` and `s2`.

### Length Check First

An interleaving must use every character of both inputs, so `s1.size() + s2.size() == s3.size()` is a necessary precondition. If it fails, return `false` immediately.

### State

`dp[i][j]` = whether the first `i` characters of `s1` and the first `j` characters of `s2` can interleave to form the first `i + j` characters of `s3`.

### Recurrence

The character `s3[i + j - 1]` came from either `s1` or `s2`:

- From `s1`: `dp[i - 1][j] && s1[i - 1] == s3[i + j - 1]`.
- From `s2`: `dp[i][j - 1] && s2[j - 1] == s3[i + j - 1]`.

`dp[i][j] = (fromS1) || (fromS2)`.

### Base Cases

- `dp[0][0] = true`.
- `dp[i][0]`: only `s1`'s prefix used — `dp[i - 1][0] && s1[i - 1] == s3[i - 1]`.
- `dp[0][j]`: only `s2`'s prefix used — `dp[0][j - 1] && s2[j - 1] == s3[j - 1]`.

### Complexity

| Approach | Time | Space |
|---|---|---|
| Tabulation | O(m · n) | O(m · n) |

### C++ — Tabulation

```cpp
#include <bits/stdc++.h>
using namespace std;

class InterleavingString {
public:
    bool isInterleave(string s1, string s2, string s3) {
        int m = s1.size(), n = s2.size();
        if (m + n != (int)s3.size()) return false; // length precondition

        vector<vector<bool>> dp(m + 1, vector<bool>(n + 1, false));
        dp[0][0] = true;

        for (int i = 1; i <= m; i++) { // first column: only s1
            dp[i][0] = dp[i - 1][0] && s1[i - 1] == s3[i - 1];
        }
        for (int j = 1; j <= n; j++) { // first row: only s2
            dp[0][j] = dp[0][j - 1] && s2[j - 1] == s3[j - 1];
        }

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                char target = s3[i + j - 1];
                bool fromS1 = dp[i - 1][j] && s1[i - 1] == target;
                bool fromS2 = dp[i][j - 1] && s2[j - 1] == target;
                dp[i][j] = fromS1 || fromS2;
            }
        }
        return dp[m][n];
    }
};
```

### Dry Run

`s1 = "aab"`, `s2 = "axy"`, `s3 = "aaxaby"`. Lengths: `3 + 3 = 6` ✓. Tracing the table, `dp[3][3]` evaluates to `true`: one valid interleaving is `a`(s1) `a`(s1) `x`(s2) `a`(s2... no) — the DP confirms a consistent path exists yielding `"aaxaby"`.

---

## Recognition Signals

| Signal in the problem statement | Likely problem / sub-pattern |
|---|---|
| "minimum operations to convert / transform one string to another", insert/delete/replace | Edit Distance (LC 72) |
| Pattern with `?` (one char) and `*` (any sequence) | Wildcard Matching (LC 44) |
| Pattern with `.` (one char) and `*` (zero-or-more of preceding) | Regular Expression Matching (LC 10) |
| "can the string be segmented into dictionary words" | Word Break (LC 139) |
| Digits encoded as letters, "count the number of ways to decode" | Decode Ways (LC 91) |
| "count substrings that are palindromes" | Palindromic Substrings (LC 647) |
| "return the longest palindromic substring" | Longest Palindromic Substring (LC 5) |
| "is s3 an interleaving of s1 and s2", merge two strings in order | Interleaving String (LC 97) |
| Two strings, compare prefixes, `dp[i][j]` over lengths | Two-string prefix DP (Edit/Wildcard/Regex/Interleave) |
| One string, property of prefix length `i` | 1D prefix DP (Word Break, Decode Ways) |
| One string, property of span `[i, j]` | 2D substring DP (palindrome problems) |

---

## Summary

| # | Problem | LC | State | Time | Space (opt.) |
|---|---|---|---|---|---|
| 48 | Edit Distance | 72 | `dp[i][j]` = min edits for prefixes | O(m·n) | O(n) |
| 49 | Wildcard Matching | 44 | `dp[i][j]` = prefixes match | O(m·n) | O(m·n) |
| 50 | Regular Expression Matching | 10 | `dp[i][j]` = prefixes match | O(m·n) | O(m·n) |
| 51 | Word Break | 139 | `dp[i]` = prefix segmentable | O(n²·L) | O(n) |
| 52 | Decode Ways | 91 | `dp[i]` = ways to decode prefix | O(n) | O(1) |
| 53 | Palindromic Substrings | 647 | center / `dp[i][j]` palindrome | O(n²) | O(1) |
| 54 | Longest Palindromic Substring | 5 | center / `dp[i][j]` palindrome | O(n²) | O(1) |
| 55 | Interleaving String | 97 | `dp[i][j]` = interleave prefixes | O(m·n) | O(m·n) |

**Key takeaways:**

- **Two strings → `dp[i][j]` over prefix lengths.** Edit Distance, Wildcard, Regex, and Interleaving all share this skeleton; the recurrence differs only in how transitions branch on the current characters and pattern symbols.
- **`*` semantics matter.** In wildcard matching `*` stands alone and matches any sequence; in regex `*` modifies the *preceding* element and means zero-or-more — leading to the `dp[i][j-2]` "skip the pair" transition.
- **Single string → `dp[i]` (prefix) or `dp[i][j]` (span).** Word Break and Decode Ways use prefix states; palindrome problems use either expand-around-center (O(1) space) or a substring boolean table filled bottom-up.
- **Space optimization.** Any `dp[i][j]` depending only on the previous row collapses to two 1D arrays (Edit Distance), and prefix recurrences depending on a fixed window collapse to a few scalars (Decode Ways).
- **Watch the edge cases:** leading `'0'` (Decode Ways), leading `*` (Wildcard/Regex base row), and the length precondition (Interleaving String).

> **Last Updated:** 2026-06-26
