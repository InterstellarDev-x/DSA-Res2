> **Topic:** [Advanced Strings](../README.md) · **Pattern 3 of 3**

# Palindrome Algorithms

Palindrome problems split into three tiers of technique:

1. **Expand around center** — O(n²), the interview default; simple and almost always accepted.
2. **Manacher's algorithm** — O(n), the optimal longest-palindromic-substring solver; mention/derive it only when O(n) is demanded.
3. **KMP on `s + reverse(s)`** — turns the *shortest palindrome by prepending* problem into a prefix-function computation.

---

## 1. Expand Around Center

Every palindrome has a center. A string of length `n` has **2n − 1** centers: `n` single-character centers (odd-length palindromes) and `n − 1` between-character centers (even-length palindromes). Expanding outward from each center while characters match finds the longest palindrome with that center.

### Problem 5 — Longest Palindromic Substring (LC 5, Medium)

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s == null || s.length() < 1) return "";
        int start = 0, end = 0;
        for (int i = 0; i < s.length(); i++) {
            int odd  = expand(s, i, i);       // odd-length, center at i
            int even = expand(s, i, i + 1);   // even-length, center between i, i+1
            int len = Math.max(odd, even);
            if (len > end - start + 1) {
                start = i - (len - 1) / 2;     // recover left boundary from length
                end = i + len / 2;
            }
        }
        return s.substring(start, end + 1);
    }

    // Returns the length of the palindrome expanding from (left, right).
    private int expand(String s, int left, int right) {
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            left--; right++;
        }
        return right - left - 1;   // chars between the over-stepped bounds
    }
}
```

**Complexity:** O(n²) time, O(1) space. The two `expand` calls (odd + even) per index are exactly the 2n−1 centers. The boundary recovery `start = i - (len-1)/2` works for both parities — memorize it.

### Problem 6 — Palindromic Substrings, count (LC 647, Medium)

> Count how many substrings are palindromes (different positions count separately).

Each successful single-character extension during expansion is one more palindrome, so count expansions instead of tracking the longest.

```java
class Solution {
    public int countSubstrings(String s) {
        int count = 0;
        for (int i = 0; i < s.length(); i++) {
            count += countFrom(s, i, i);       // odd centers
            count += countFrom(s, i, i + 1);   // even centers
        }
        return count;
    }

    private int countFrom(String s, int left, int right) {
        int count = 0;
        while (left >= 0 && right < s.length() && s.charAt(left) == s.charAt(right)) {
            count++; left--; right++;
        }
        return count;
    }
}
```

**Complexity:** O(n²) time, O(1) space.

> **DP alternative.** Both problems also have a classic O(n²) time / O(n²) space DP: `dp[i][j] = (s[i]==s[j]) && (j-i<2 || dp[i+1][j-1])`. Expand-around-center is preferred because it is O(1) space and just as fast. See [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) for the tabulation form.

---

## 2. Manacher's Algorithm — O(n) Longest Palindrome

Manacher removes the odd/even split by **transforming** the string: insert a separator (`'#'`) between every character and at both ends. `"aba"` → `"^#a#b#a#$"` (sentinels `^`, `$` avoid bounds checks). Now *every* palindrome in the transformed string has odd length and a single center, and its radius in the transformed string equals the palindrome length in the original.

We keep a `center C` and `right boundary R` of the rightmost palindrome found, plus an array `p[i]` = radius at `i`. The **mirror trick**: for `i < R`, its mirror is `mirror = 2*C - i`, and `p[i]` can be seeded from `p[mirror]`, capped by `R - i`.

```java
class Solution {
    public String longestPalindrome(String s) {
        if (s.isEmpty()) return "";
        // Build transformed string: ^ # a # b # ... $
        StringBuilder t = new StringBuilder("^");
        for (char c : s.toCharArray()) t.append('#').append(c);
        t.append("#$");
        String tt = t.toString();
        int n = tt.length();
        int[] p = new int[n];     // p[i] = palindrome radius centered at i
        int C = 0, R = 0;         // current center and right boundary
        for (int i = 1; i < n - 1; i++) {
            int mirror = 2 * C - i;
            if (i < R) p[i] = Math.min(R - i, p[mirror]);   // mirror seed, capped
            // expand around i past the seeded radius
            while (tt.charAt(i + p[i] + 1) == tt.charAt(i - p[i] - 1)) p[i]++;
            if (i + p[i] > R) { C = i; R = i + p[i]; }       // slide the window
        }
        // find max radius and map back to original indices
        int maxLen = 0, centerIndex = 0;
        for (int i = 1; i < n - 1; i++) {
            if (p[i] > maxLen) { maxLen = p[i]; centerIndex = i; }
        }
        int start = (centerIndex - maxLen) / 2;   // convert transformed -> original
        return s.substring(start, start + maxLen);
    }
}
```

**Complexity:** O(n) time, O(n) space. The sentinels `^`/`$` mean the inner `while` never reads out of bounds (they never match `#` or a real char). The mapping `start = (centerIndex - maxLen) / 2` undoes the `#`-insertion. **Interview note:** code this only if asked for linear time — it is fiddly and easy to get wrong under pressure; expand-around-center is the safe default.

---

## 3. Problem 3 — Shortest Palindrome (LC 214, Hard)

> By prepending characters in front of `s`, form the shortest palindrome and return it.

**Insight.** We want the **longest palindromic prefix** of `s`. The characters after that prefix must be reversed and prepended. Finding the longest palindromic prefix reduces to a prefix-function (KMP) computation:

> Build `combined = s + '#' + reverse(s)`. Compute its LPS array. `lps[last]` = length of the longest prefix of `s` that equals a suffix of `reverse(s)` — which is precisely the **longest palindromic prefix** of `s`.

The `'#'` separator (a char not in `s`) is essential: without it the border could spill from `s` into `reverse(s)` and overcount, producing a wrong (and possibly non-palindromic) answer.

```java
class Solution {
    public String shortestPalindrome(String s) {
        if (s.isEmpty()) return s;
        String rev = new StringBuilder(s).reverse().toString();
        String combined = s + "#" + rev;
        int[] lps = buildLPS(combined);
        int palPrefixLen = lps[combined.length() - 1];   // longest palindromic prefix
        // characters of s after the palindromic prefix, reversed, go in front
        String suffix = s.substring(palPrefixLen);
        return new StringBuilder(suffix).reverse().toString() + s;
    }

    private int[] buildLPS(String t) {
        int n = t.length();
        int[] lps = new int[n];
        int len = 0, i = 1;
        while (i < n) {
            if (t.charAt(i) == t.charAt(len)) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        return lps;
    }
}
```

**Complexity:** O(n) time, O(n) space. Example: `s = "aacecaaa"`, `rev = "aaacecaa"`, `combined = "aacecaaa#aaacecaa"`. The LPS ends at 7 (`"aacecaa"` is the longest palindromic prefix), leaving the trailing `"a"`; prepend `reverse("a") = "a"` → `"aaacecaaa"`.

---

## 4. Dry-run — Expand Around Center on `"babad"`

Indices: `b(0) a(1) b(2) a(3) d(4)`.

| Center | Type | Expansion | Palindrome | Length |
|--------|------|-----------|------------|--------|
| i=0 | odd | `b` only (b vs nothing left) | `b` | 1 |
| i=0/1 | even | `b`≠`a` | — | 0 |
| i=1 | odd | `a`; then s[0]=b == s[2]=b → `bab`; then out of... s[-1] | `bab` | 3 |
| i=2 | odd | `b`; then s[1]=a == s[3]=a → `aba`; then s[0]=b ≠ s[4]=d | `aba` | 3 |
| i=3 | odd | `a`; then s[2]=b ≠ s[4]=d | `a` | 1 |
| i=4 | odd | `d` | `d` | 1 |

First maximal length 3 is found at center 1 (`"bab"`); center 2 ties with `"aba"`. We keep the first, so the answer is `"bab"` (either is accepted by LC 5). The even centers all fail immediately here because no two adjacent characters are equal.

---

## Recognition Signals

| Signal in the prompt | Reach for |
|----------------------|-----------|
| "longest palindromic substring" | expand around center (Manacher if O(n) required) |
| "count palindromic substrings" | expand around center, sum extensions |
| "shortest palindrome by prepending / appending" | KMP/LPS on `s + '#' + reverse(s)` |
| "is it a palindrome", basic check | two-pointer compare (no advanced tool needed) |
| explicitly "in linear time" for longest palindrome | Manacher's algorithm |
| "palindrome partitioning / min cuts" | DP — see [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) |

---

## Cross-links

- [String Matching (KMP)](String%20Matching%20(KMP).md) — Shortest Palindrome is built directly on the prefix function.
- [DP on Strings](../../14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) — the DP tabulation alternative for palindromic substrings and partitioning.

> **Last Updated:** 2026-06-26
