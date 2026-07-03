# String Hashing (Rabin-Karp)

> **Topic:** [Strings](../README.md) · **Difficulty:** Hard
> **Tags:** `rabin-karp` `rolling-hash` `polynomial-hash` `substring-search`

---

## Table of Contents

1. [Pattern Overview](#pattern-overview)
2. [When to Use](#when-to-use)
3. [Recognition Cues](#recognition-cues)
4. [Complexity](#complexity)
5. [C++ Templates](#c-templates)
6. [Common Mistakes](#common-mistakes)
7. [Applications](#applications)
8. [Practice Problems](#practice-problems)
9. [Related Patterns](#related-patterns)

---

## Pattern Overview

String Hashing maps a string to a numeric fingerprint (hash) such that:
- Equal strings → same hash (guaranteed)
- Different strings → different hash (with very high probability if hash chosen carefully)

**Polynomial rolling hash:**
```
hash(s[0..n-1]) = s[0]*B^(n-1) + s[1]*B^(n-2) + ... + s[n-1]*B^0   (mod M)
```

**Rolling hash** enables O(1) hash of any substring `s[l..r]` after O(n) preprocessing:
```
hash(s[l..r]) = (prefix[r+1] - prefix[l] * B^(r-l+1)) mod M
```

**Rabin-Karp algorithm** uses rolling hashes to find pattern occurrences in O(n + m) average time.

---

## When to Use

- Find all occurrences of pattern in text (same power as KMP)
- Count **distinct substrings**
- Find **longest common substring** between two strings
- Check if two large strings are equal after transformation
- Binary search on answer length + hash verification

---

## Recognition Cues

| Cue | Approach |
|-----|----------|
| "count distinct substrings" | Hash all substrings, count unique hashes |
| "longest repeated substring" | Binary search on length + rolling hash set |
| "find pattern, avoid worst case" | Rabin-Karp |
| "longest common substring" | Binary search + hashing |
| "check if rotation" | Hash of s == hash of any rotation |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build prefix hash | O(n) | O(n) |
| Substring hash query | O(1) | — |
| Rabin-Karp search | O(n + m) average, O(nm) worst | O(m) |
| Count distinct substrings | O(n²) | O(n²) hashes stored |
| Longest repeated substring | O(n log n) | O(n) |

---

## C++ Templates

### 1. Polynomial Prefix Hash (Build + Query)

```cpp
#include <bits/stdc++.h>
using namespace std;

struct StringHasher {
    static const long long BASE = 131;
    static const long long MOD  = (long long)1e9 + 7;

    vector<long long> prefix;
    vector<long long> power;

    StringHasher(const string& s) {
        int n = s.length();
        prefix.resize(n + 1);
        power.resize(n + 1);
        power[0] = 1;

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = (prefix[i] * BASE + s[i]) % MOD;
            power[i + 1]  = power[i] * BASE % MOD;
        }
    }

    // Hash of s[l..r] inclusive, 0-indexed
    long long hash(int l, int r) {
        return (prefix[r + 1] - prefix[l] * power[r - l + 1] % MOD + MOD * MOD) % MOD;
    }
};
// Build: O(n) | Query: O(1)
```

### 2. Rabin-Karp Pattern Search

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> rabinKarp(const string& text, const string& pattern) {
    vector<int> result;
    int n = text.length(), m = pattern.length();
    if (m > n) return result;

    StringHasher th(text);
    StringHasher ph(pattern);
    long long patHash = ph.hash(0, m - 1);

    for (int i = 0; i <= n - m; i++) {
        if (th.hash(i, i + m - 1) == patHash) {
            // Verify to handle hash collisions
            if (text.substr(i, m) == pattern) result.push_back(i);
        }
    }
    return result;
}
// Time: O(n + m) average | O(nm) worst (hash collisions)
```

### 3. Count Distinct Substrings

```cpp
#include <bits/stdc++.h>
using namespace std;

int countDistinctSubstrings(const string& s) {
    int n = s.length();
    unordered_set<long long> seen;
    StringHasher hasher(s);

    for (int len = 1; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            seen.insert(hasher.hash(i, i + len - 1));
        }
    }
    return seen.size();
    // Time: O(n²) | Space: O(n²)
}
```

### 4. Longest Repeated Substring (Binary Search + Hash)

```cpp
#include <bits/stdc++.h>
using namespace std;

int findRepeated(StringHasher& h, const string& s, int n, int len) {
    unordered_set<long long> seen;
    for (int i = 0; i <= n - len; i++) {
        long long hash = h.hash(i, i + len - 1);
        if (!seen.insert(hash).second) return i - 1; // collision possible; accept for now
    }
    return -1;
}

string longestRepeatedSubstring(const string& s) {
    int n = s.length();
    StringHasher hasher(s);
    int lo = 1, hi = n - 1, bestStart = -1, bestLen = 0;

    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        int start = findRepeated(hasher, s, n, mid);
        if (start != -1) { bestStart = start; bestLen = mid; lo = mid + 1; }
        else hi = mid - 1;
    }
    return bestStart == -1 ? "" : s.substr(bestStart, bestLen);
}
// Time: O(n log n) | Space: O(n)
```

### 5. Double Hashing (Reduce Collision Probability)

```cpp
#include <bits/stdc++.h>
using namespace std;

// Use two independent hash functions to reduce false positive probability
pair<long long, long long> doubleHash(const string& s, int l, int r) {
    long long h1 = hashWith(s, l, r, 131, (long long)1e9+7);
    long long h2 = hashWith(s, l, r, 137, (long long)1e9+9);
    return {h1, h2};
}
// Collision probability ≈ 1/(1e9+7) × 1/(1e9+9) ≈ 10^{-18}
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Negative hash values | Add `MOD` before taking `% MOD`: `(val % MOD + MOD) % MOD` |
| Using `int` for hash values | Use `long long` — intermediate products overflow `int` |
| Not verifying on hash match | Always do `==` check when hash matches (collision guard) |
| Single hash with small MOD | Use double hashing or MOD ≈ 10^9+7 |
| `power` array not initialized | `power[0] = 1` then `power[i+1] = power[i] * BASE % MOD` |
| Substring bounds: `.substr(l, len)` vs `.substr(l, r)` | C++'s `.substr(pos, len)` takes position and length; use `.substr(l, r - l + 1)` for inclusive `r` |

---

## Applications

| Application | Description |
|-------------|-------------|
| Plagiarism detection | Hash all substrings; find matching windows |
| Longest common substring | Binary search length + hash sets |
| Count distinct palindromes | Combine with palindrome expansion |
| Substring equality in O(1) | Two string hashers, compare queries |
| Rolling hash in competitive programming | Combine with other algorithms |

---

## Practice Problems

| Problem | Difficulty | LeetCode |
|---------|-----------|----------|
| [Implement strStr()](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Easy | LC 28 |
| [Longest Duplicate Substring](https://leetcode.com/problems/longest-duplicate-substring/) | Hard | LC 1044 |
| [Repeated String Match](https://leetcode.com/problems/repeated-string-match/) | Medium | LC 686 |
| [Longest Repeating Substring](https://leetcode.com/problems/longest-repeating-substring/) | Medium | LC 1062 |
| Count Distinct Substrings | Medium | GFG |

---

## Related Patterns

- [KMP and Z-Algorithm](./KMP%20and%20Z%20Algorithm.md) — deterministic O(n+m) alternative
- [Sliding Window on Strings](./Sliding%20Window%20on%20Strings.md) — fixed-length window
- [Frequency Count](./Frequency%20Count.md) — when char counts identify strings

---

> **Interview Tip:** In product-company interviews, Rabin-Karp is rarely asked directly. The rolling hash technique appears as a building block for "longest duplicate substring" or "count distinct substrings" — problems that require O(1) substring comparison. Know the `(prefix[r+1] - prefix[l] * power[r-l+1]) % MOD` formula and the negative-modulo fix.

> **Last Updated:** 2026-06-26
