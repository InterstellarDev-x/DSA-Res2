> **Topic:** [Advanced Strings](../README.md) · **Pattern 2 of 3**

# Z Algorithm & Polynomial Hashing

Two linear-time string tools that complement KMP. The **Z-array** is a per-index "how much of the prefix repeats here" function and gives an alternative O(n+m) matcher. **Polynomial rolling hash** turns any substring into an O(1)-comparable number, powering Rabin-Karp, duplicate detection, and binary-search-on-length problems.

---

## 1. The Z-array

For a string `s` of length `n`:

> `z[i]` = the length of the longest substring starting at index `i` that is **also a prefix** of `s`. By convention `z[0] = 0` (or `n`; we use 0 and treat the whole-string case separately).

Example: for `s = "aabxaab"`, `z = [_, 1, 0, 0, 3, 1, 0]` — at index 4 the substring `"aab"` matches the prefix `"aab"`, so `z[4] = 3`.

### Construction with the [L, R] window

We maintain a window `[L, R]` — the rightmost segment we have seen that is *known to match a prefix*. For a new `i`:

- If `i < R`, the mirror index `i - L` already has a `z` value we can reuse: `z[i] = min(R - i, z[i - L])`.
- Then we try to **extend** by comparing characters past `R`.
- If we extended past `R`, update the window `L = i, R = i + z[i]`.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> buildZ(const string& s) {
    int n = s.length();
    vector<int> z(n, 0);
    int L = 0, R = 0;            // current rightmost match window [L, R)
    for (int i = 1; i < n; i++) {
        if (i < R) {
            z[i] = min(R - i, z[i - L]);   // reuse mirror, capped at window
        }
        // try to extend the match beyond what we reused
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) {
            z[i]++;
        }
        if (i + z[i] > R) {       // extended past R -> slide the window
            L = i;
            R = i + z[i];
        }
    }
    return z;
}
```

The `min(R - i, z[i - L])` cap is the subtle part: the mirror value may claim more than fits inside the window, so we clamp to `R - i` and verify the rest by explicit comparison. Each character is compared O(1) amortized → O(n) total.

---

## 2. Z-based pattern matching

To find pattern `P` inside text `T`, build the Z-array of `P + '$' + T` where `$` does not occur in either string. Every index `i` (in the `T` region) with `z[i] == |P|` marks an occurrence.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> zSearch(const string& text, const string& pattern) {
    string combined = pattern + '$' + text;
    vector<int> z = buildZ(combined);
    int m = pattern.length();
    vector<int> result;
    for (int i = m + 1; i < (int)combined.length(); i++) {
        if (z[i] == m) {
            result.push_back(i - m - 1);   // offset back into the original text
        }
    }
    return result;
}
```

`z[i] == m` means the next `m` characters fully equal the pattern prefix — a match. The separator `$` guarantees no `z` value can spill across the boundary.

---

## 3. Polynomial Rolling Hash Fundamentals

Treat a string as a number in base `B`:

> `hash(s) = (s[0]*B^(m-1) + s[1]*B^(m-2) + ... + s[m-1]*B^0) mod M`

- **Base `B`:** a value larger than the alphabet, commonly `31`, `131`, or `256`.
- **Modulus `M`:** a large prime such as `1'000'000'007LL` to keep values in `long long` range and spread collisions.
- **Precompute powers** `B^0..B^(m)` once so a window hash updates in O(1).

```cpp
#include <bits/stdc++.h>
using namespace std;

const long long MOD = 1000000007LL;
const long long BASE = 131LL;

// Precompute base powers up to length n.
vector<long long> powers(int n) {
    vector<long long> p(n + 1);
    p[0] = 1;
    for (int i = 1; i <= n; i++) p[i] = p[i - 1] * BASE % MOD;
    return p;
}
```

Rolling forward by one character (drop leftmost, append rightmost):
`newHash = ((oldHash - s[i] * B^(m-1)) * B + s[i+m]) mod M`, always re-normalizing with `(x % MOD + MOD) % MOD` to kill negatives. Full mechanics live in [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md).

---

## 4. Problem 7 — Find All Anagrams in a String (LC 438, Medium)

> Return the start indices of all substrings of `s` that are anagrams of `p`.

An anagram is purely a **multiset of characters**, so a 26-slot frequency window is the natural, collision-free tool. This is fundamentally a fixed-size sliding window — see the [Sliding Window topic](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md).

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> findAnagrams(string s, string p) {
        vector<int> res;
        int n = s.length(), m = p.length();
        if (n < m) return res;
        vector<int> need(26, 0), win(26, 0);
        for (auto& c : p) need[c - 'a']++;
        for (int i = 0; i < n; i++) {
            win[s[i] - 'a']++;            // add right edge
            if (i >= m) win[s[i - m] - 'a']--;  // remove left edge
            if (i >= m - 1 && win == need) {
                res.push_back(i - m + 1);
            }
        }
        return res;
    }
};
```

**Complexity:** O(n * 26) ≈ O(n) time, O(1) space. `==` on two `vector<int>` of size 26 is constant.

**Hashing alternative.** Instead of comparing arrays you can keep a *commutative* hash of the window — e.g. a fixed-length 26-vector difference counter, or a polynomial over the sorted multiset. Frequency comparison is simpler and exact, so prefer it for anagrams; rolling hash earns its keep when *order matters* (exact substring), not for anagrams.

---

## 5. Problem 9a — Longest Duplicate Substring (LC 1044, Hard)

> Find the longest substring that occurs at least twice (may overlap). Return any such substring (or "" if none).

**Strategy: binary search on the length + Rabin-Karp.** A duplicate of length `L` existing implies a duplicate of every length `< L` exists (a prefix of it), so the predicate "exists a duplicate of length `L`" is monotonic — binary-search `L` in `[1, n-1]`. For a fixed `L`, slide a rolling hash over all length-`L` windows and store hashes in a set; a repeat hash (verified) means a duplicate.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
    static const long long MOD = (1LL << 61) - 1;   // large Mersenne prime, room in long long
    static const long long BASE = 131LL;

public:
    string longestDupSubstring(string s) {
        int n = s.length();
        vector<int> a(n);
        for (int i = 0; i < n; i++) a[i] = s[i] - 'a' + 1;
        int lo = 1, hi = n - 1, start = -1, bestLen = 0;
        while (lo <= hi) {
            int mid = (lo + hi) >> 1;
            int pos = search(a, mid);
            if (pos != -1) { start = pos; bestLen = mid; lo = mid + 1; }
            else { hi = mid - 1; }
        }
        return start == -1 ? "" : s.substr(start, bestLen);
    }

private:
    // Returns the start index of some duplicated substring of length L, or -1.
    int search(const vector<int>& a, int L) {
        long long pow = 1;
        for (int i = 0; i < L; i++) pow = mulmod(pow, BASE);
        long long h = 0;
        for (int i = 0; i < L; i++) h = (mulmod(h, BASE) + a[i]) % MOD;
        unordered_map<long long, vector<int>> seen;
        seen[h].push_back(0);
        for (int i = L; i < (int)a.size(); i++) {
            h = (mulmod(h, BASE) + a[i] - mulmod(a[i - L], pow) % MOD + MOD * MOD) % MOD;
            int startIdx = i - L + 1;
            auto it = seen.find(h);
            if (it != seen.end()) {
                // verify to defeat spurious hash hits (collision safety)
                for (int j : it->second) if (equalsAt(a, j, startIdx, L)) return startIdx;
            }
            seen[h].push_back(startIdx);
        }
        return -1;
    }

    bool equalsAt(const vector<int>& a, int x, int y, int L) {
        for (int k = 0; k < L; k++) if (a[x + k] != a[y + k]) return false;
        return true;
    }

    long long mulmod(long long x, long long y) {
        return (x % MOD) * (y % MOD) % MOD;   // fits because MOD < 2^61, x,y < MOD
    }
};
```

**Complexity:** O(n log n) average — `log n` binary-search steps, each an O(n) rolling pass. **Double hashing** (two independent (BASE, MOD) pairs, compare both) makes a collision astronomically unlikely and lets you skip the explicit verification; we kept the verification here for a guaranteed-correct answer.

---

## 6. Problem 9b — Distinct Echo Substrings (LC 1316, Hard)

> Count the number of **distinct** substrings that are a concatenation of some string with itself (i.e. of the form `a + a`, an "echo").

A substring `s[i..i+2k-1]` is an echo iff its first half `s[i..i+k-1]` equals its second half `s[i+k..i+2k-1]`. With prefix hashes, comparing two halves is O(1); store every distinct echo as a (hash) to dedupe.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int distinctEchoSubstrings(string text) {
        int n = text.length();
        const long long MOD = 1000000007LL, BASE = 131LL;
        vector<long long> h(n + 1, 0), pw(n + 1, 0);
        pw[0] = 1;
        for (int i = 0; i < n; i++) {
            h[i + 1] = (h[i] * BASE + text[i]) % MOD;
            pw[i + 1] = pw[i] * BASE % MOD;
        }
        unordered_set<long long> distinct;
        for (int len = 1; 2 * len <= n; len++) {           // half-length
            for (int i = 0; i + 2 * len <= n; i++) {
                long long first  = sub(h, pw, MOD, i, i + len);
                long long second = sub(h, pw, MOD, i + len, i + 2 * len);
                if (first == second) distinct.insert(first); // dedupe by half-hash
            }
        }
        return distinct.size();
    }

private:
    // hash of text[l..r) from prefix hashes.
    long long sub(const vector<long long>& h, const vector<long long>& pw, long long MOD, int l, int r) {
        return ((h[r] - h[l] * pw[r - l]) % MOD + MOD) % MOD;
    }
};
```

**Complexity:** O(n^2) hashing comparisons, O(n^2) distinct entries worst-case. Note the `(... + MOD) % MOD` to fix the negative produced by `h[r] - h[l]*pw[]`. For full collision safety in interviews, mention double hashing on the stored key.

---

## 7. Dry-run — Z-array of `"aabxaabxcaabxaabxay"`

Indices: `a a b x a a b x c a a b x a a b x a y` (length 19).

| i | reuse `min(R-i, z[i-L])` | extend? | z[i] | window [L,R] |
|---|--------------------------|---------|------|--------------|
| 1 | i≥R → 0 | s[0]='a'==s[1]='a', then s[1]='a'≠s[2]='b' → 1 | 1 | [1,2] |
| 2 | i≥R → 0 | s[0]='a'≠'b' | 0 | unchanged |
| 3 | 0 | 'a'≠'x' | 0 | unchanged |
| 4 | 0 | "aabx" matches prefix "aabx", stops at index 8 'c' | 4 | [4,8] |
| 5 | i<R: min(8-5, z[1]=1)=1 | s[1]='a'≠s[6]='b' | 1 | unchanged |
| 6 | min(8-6, z[2]=0)=0 | 'a'≠'b' | 0 | unchanged |
| 7 | min(8-7, z[3]=0)=0 | 'a'≠'x' | 0 | unchanged |
| 9 | i≥R → 0 | "aabxaabxa" matches the prefix for 9, stops at end-ish | 9 | [9,18] |
| 10..17 | reused from mirror inside [9,18] | clamp + verify | mirrors z[1..8] | — |

The big wins are `z[4]=4` (`"aabx"` repeats) and `z[9]=9` (`"aabxaabxa"` echoes the prefix). Inside the window `[9,18]` every later index copies its mirror in O(1), which is exactly why the algorithm is linear.

---

## Recognition Signals

| Signal in the prompt | Reach for |
|----------------------|-----------|
| "match prefix at each position", linear pattern search | Z-array on `P + '$' + T` |
| "anagrams", "permutation of p", order-independent | frequency window (see [Sliding Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md)) |
| "longest duplicated substring", overlapping allowed | binary search on length + Rabin-Karp |
| "echo" / "concatenation of a string with itself" | prefix-hash half comparison |
| "compare many substrings for equality fast" | polynomial prefix hashes, O(1) per compare |
| huge text, need average linear matching, hashing OK | rolling hash (Rabin-Karp) |

---

## Cross-links

- [String Matching (KMP)](String%20Matching%20(KMP).md) — Z and LPS are interchangeable linear matchers.
- [Rolling Hash (Rabin-Karp)](../Design%20Data%20Structure%20Problems/Rolling%20Hash%20(Rabin-Karp).md) — the hashing engine these problems lean on.
- [Sliding Window — Fixed Size Window](../../8.Sliding_Window/Patterns/Fixed%20Size%20Window.md) — the canonical home of Find All Anagrams.

> **Last Updated:** 2026-06-26
