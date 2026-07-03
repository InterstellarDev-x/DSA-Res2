> **Topic:** [Advanced Strings](../README.md) · **Tips 1 of 4**

# Coding Tips — Advanced Strings

Eight battle-tested tips for writing correct string-algorithm code under interview pressure.

---

### Tip 1 — KMP vs just using `find`

`haystack.find(needle)` already does O(n+m) substring search in C++. Reach for **hand-written KMP** only when: library calls are forbidden, the interviewer asks for the algorithm, you need guaranteed worst-case linear, or you search the *same* pattern across many texts (precompute `lps` once). State the pragmatic answer first, then offer KMP.

```cpp
#include <bits/stdc++.h>
using namespace std;
size_t idx = haystack.find(needle);   // O(n+m), use this unless told otherwise
```

---

### Tip 2 — Memorize the LPS construction template

The prefix function is the spine of this whole topic. Drill this until it is muscle memory:

```cpp
#include <bits/stdc++.h>
using namespace std;
vector<int> lps(n);
int len = 0, i = 1;
while (i < n) {
    if (s[i] == s[len]) { lps[i++] = ++len; }
    else if (len > 0) { len = lps[len - 1]; }   // fall back; do NOT advance i
    else { lps[i++] = 0; }
}
```

The single most common bug is advancing `i` on the fallback branch. On a mismatch with `len > 0`, only `len` changes.

---

### Tip 3 — Expand-around-center is usually enough

For Longest Palindromic Substring / Palindromic Substrings, **expand-around-center** (O(n²), O(1) space) is the right default — short and obviously correct. Mention **Manacher (O(n))** only if the interviewer explicitly asks for linear time. Implementing Manacher unprompted wastes time and invites bugs.

```cpp
#include <bits/stdc++.h>
using namespace std;
int expand(const string& s, int l, int r) {
    while (l >= 0 && r < (int)s.length() && s[l] == s[r]) { l--; r++; }
    return r - l - 1;
}
```

---

### Tip 4 — Rolling hash: always `long long` + prime mod

A polynomial hash overflows 32-bit `int` instantly. Use `long long`, a prime modulus, and reduce after every multiply.

```cpp
#include <bits/stdc++.h>
using namespace std;
const long long MOD = 1000000007LL;
const long long BASE = 131LL;            // or 31 / 256
hash = (hash * BASE + s[i]) % MOD;
```

---

### Tip 5 — `(x % m + m) % m` for negative remainders

Rolling-hash updates **subtract** the leaving character, which can go negative. C++'s `%` keeps the sign of the dividend, so normalize:

```cpp
windowHash = ((windowHash - leaving) % MOD + MOD) % MOD;   // negative-safe
```

Forgetting this gives sporadic wrong answers that pass small tests and fail large ones.

---

### Tip 6 — Verify on a hash collision

A hash match is **necessary, not sufficient**. Two distinct substrings can share a hash (spurious hit). On every hash match, either confirm the characters or use double hashing.

```cpp
#include <bits/stdc++.h>
using namespace std;
if (windowHash == patternHash && text.compare(i, m, pattern) == 0) return i;
```

---

### Tip 7 — Z-array and LPS carry the same information

`lps[i]` = longest border *ending* at `i`; `z[i]` = longest prefix-match *starting* at `i`. Either solves single-pattern matching in O(n+m) — pick whichever coordinate system the problem makes natural (concatenate `pattern + '#' + text` for Z; build `lps` on the pattern for KMP). Don't waste energy converting between them mid-interview.

---

### Tip 8 — A string of length `n` has `2n-1` palindrome centers

`n` odd-length centers (each character) plus `n-1` even-length centers (each gap). When counting or searching palindromes, iterate *both* per index:

```cpp
for (int i = 0; i < n; i++) {
    count += expand(s, i, i);       // odd center
    count += expand(s, i, i + 1);   // even center
}
```

Missing the even centers silently drops every even-length palindrome — a classic off-by-one bug.

---

## Related

- [Common Mistakes](Common%20Mistakes.md) · [Algorithm Selection](Algorithm%20Selection.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
