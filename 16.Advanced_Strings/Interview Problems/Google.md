> **Topic:** [Advanced Strings](../README.md) · **Section:** Interview Problems

# Google — Advanced Strings (Interview Deep Dives)

Google is where the *hard* string algorithms actually earn their keep. At **L4** you are expected to reach for the prefix function or rolling hash without hand-holding; at **L5** the bar is deriving the trick (Shortest Palindrome) and handling adversarial collisions (Longest Duplicate Substring) cleanly. Google loves string algorithms because they reward turning an O(n²) brute force into O(n log n) or O(n) via a clever invariant — exactly the "algorithmic maturity" signal they grade.

---

## 1. Shortest Palindrome (LC 214) — KMP on `s + '#' + reverse(s)`

> Prepend the fewest characters to `s` to make it a palindrome. Return the result.

### The insight, derived

You may only **prepend**, so the suffix of `s` is fixed. To minimize additions, find the **longest prefix of `s` that is itself a palindrome** — call its length `p`. The remaining suffix `s[p..]` reversed is prepended. So the whole task reduces to: *what is the longest palindromic prefix of `s`?*

Now the KMP trick. Let `r = reverse(s)`. A prefix `s[0..k)` is a palindrome iff it equals its own reverse, i.e. iff `s[0..k)` equals the **suffix** of `r` of length `k`. So we want the longest prefix of `s` that is also a suffix of `r` — that is exactly the **prefix function** evaluated on the combined string:

```
combined = s + '#' + reverse(s)
```

The `'#'` is a separator **not appearing in `s`**, so a border of `combined` can never exceed `|s|` (it cannot straddle the `#`). The value `lps[combined.length() - 1]` is the length of the longest prefix of `s` that matches a suffix of `reverse(s)` — i.e. the longest palindromic prefix length `p`.

### Why the separator matters

Without `'#'`, the prefix-function fallback could match characters of `s` against characters of `s` (since `reverse(s)` begins with `s`'s last char), producing a border **longer than `|s|`** and a wrong answer. The separator caps every border at `|s|`. Picking a separator that *can* occur in the input is the classic bug — use a sentinel guaranteed absent (here inputs are lowercase letters, so `'#'` is safe).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    string shortestPalindrome(string s) {
        if (s.empty() || s.length() <= 1) return s;
        string rev = s;
        reverse(rev.begin(), rev.end());
        string combined = s + "#" + rev;
        vector<int> lps = buildLPS(combined);
        int p = lps[combined.length() - 1];      // longest palindromic prefix length
        string toPrepend = rev.substr(0, s.length() - p);
        return toPrepend + s;
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

**Complexity:** O(n) time and space. Full LPS construction in [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md); the palindrome side in [Palindrome Algorithms](../Patterns/Palindrome%20Algorithms.md).

---

## 2. Longest Duplicate Substring (LC 1044) — binary search on length + Rabin-Karp

> Return any longest substring of `s` that appears at least twice (possibly overlapping). Empty string if none.

### Two ideas stacked

1. **Monotonicity → binary search on length.** If a duplicate of length `L` exists, then so does one of length `L-1` (take any prefix of it). So `feasible(L)` is monotone, and we binary search the largest `L` in `[1, n-1]`.
2. **Rabin-Karp for `feasible(L)`.** Hash every length-`L` window; if a hash repeats, a duplicate of length `L` likely exists. Use an `std::unordered_set` of hashes for O(n) per check.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    static const long long MOD1 = 1000000007LL;
    static const long long MOD2 = 998244353LL;
    static const long long BASE = 131LL;

public:
    string longestDupSubstring(string s) {
        int n = s.length();
        vector<long long> h1(n + 1), h2(n + 1);
        vector<long long> p1(n + 1), p2(n + 1);
        p1[0] = p2[0] = 1;
        for (int i = 0; i < n; i++) {
            h1[i + 1] = (h1[i] * BASE + s[i]) % MOD1;
            h2[i + 1] = (h2[i] * BASE + s[i]) % MOD2;
            p1[i + 1] = (p1[i] * BASE) % MOD1;
            p2[i + 1] = (p2[i] * BASE) % MOD2;
        }

        int lo = 1, hi = n - 1, start = -1, bestLen = 0;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;
            int found = firstDup(s, mid, h1, h2, p1, p2);
            if (found != -1) {
                start = found; bestLen = mid; lo = mid + 1;
            } else {
                hi = mid - 1;
            }
        }
        return start == -1 ? "" : s.substr(start, bestLen);
    }

private:
    // Returns the start index of a duplicated length-L substring, or -1.
    int firstDup(string& s, int L, vector<long long>& h1, vector<long long>& h2,
                 vector<long long>& p1, vector<long long>& p2) {
        int n = s.length();
        unordered_set<long long> seen;
        for (int i = 0; i + L <= n; i++) {
            long long a = ((h1[i + L] - h1[i] * p1[L]) % MOD1 + MOD1) % MOD1;
            long long b = ((h2[i + L] - h2[i] * p2[L]) % MOD2 + MOD2) % MOD2;
            long long combined = a * MOD2 + b;   // double hash packed into one long
            if (!seen.insert(combined).second) return i;
        }
        return -1;
    }
};
```

### Double hashing for collisions

A **single** hash near `10^9` is provably breakable — LeetCode's hard test cases include adversarial collisions. Using **two** independent moduli (`MOD1`, `MOD2`) drops the collision probability to ~`10^-18`, packed into one `long` for O(1) set lookups. Note the negative-safe `(x % MOD + MOD) % MOD` on every subtraction. Full hashing reference: [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md).

**Complexity:** O(n log n) time, O(n) space.

---

## 3. Longest Happy Prefix (LC 1392) — prefix function, directly

> A *happy prefix* is a non-empty prefix that is also a proper suffix. Return the longest one.

This is the prefix function **by definition** — no extra work. `lps[n-1]` is the length of the longest proper prefix of the whole string that is also a suffix.

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
        return s.substr(0, lps[n - 1]);
    }
};
```

**Complexity:** O(n) time and space. The interview signal is recognizing that "prefix that is also a suffix" *is* the LPS — candidates who reinvent it with hashing miss the elegance.

---

## L4 / L5 context — why Google loves string algorithms

- **L4:** recognize the right engine fast, implement it cleanly, state complexity. Solving Longest Happy Prefix with the prefix function (not hashing) is the kind of "knows the canonical tool" signal expected.
- **L5:** *derive* the non-obvious reduction (Shortest Palindrome → prefix function on `s#reverse(s)`) and *defend correctness under adversarial input* (double hashing). The interviewer is grading whether you turn O(n²) into O(n log n)/O(n) with a justified invariant and reason about failure modes (collisions, separators) without being prompted.

---

## Related

- OA frequency: [OA-Qns → Google](../OA-Qns/Google.md)
- Other deep dives: [Amazon](Amazon.md) · [Microsoft](Microsoft.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
