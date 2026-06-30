> **Topic:** [Advanced Strings](../README.md) · **Tips 1 of 4**

# Coding Tips — Advanced Strings

Eight battle-tested tips for writing correct string-algorithm code under interview pressure.

---

### Tip 1 — KMP vs just using `indexOf`

`haystack.indexOf(needle)` already does O(n+m) substring search in Java. Reach for **hand-written KMP** only when: library calls are forbidden, the interviewer asks for the algorithm, you need guaranteed worst-case linear, or you search the *same* pattern across many texts (precompute `lps` once). State the pragmatic answer first, then offer KMP.

```java
int idx = haystack.indexOf(needle);   // O(n+m), use this unless told otherwise
```

---

### Tip 2 — Memorize the LPS construction template

The prefix function is the spine of this whole topic. Drill this until it is muscle memory:

```java
int[] lps = new int[n];
int len = 0, i = 1;
while (i < n) {
    if (s.charAt(i) == s.charAt(len)) { lps[i++] = ++len; }
    else if (len > 0) { len = lps[len - 1]; }   // fall back; do NOT advance i
    else { lps[i++] = 0; }
}
```

The single most common bug is advancing `i` on the fallback branch. On a mismatch with `len > 0`, only `len` changes.

---

### Tip 3 — Expand-around-center is usually enough

For Longest Palindromic Substring / Palindromic Substrings, **expand-around-center** (O(n²), O(1) space) is the right default — short and obviously correct. Mention **Manacher (O(n))** only if the interviewer explicitly asks for linear time. Implementing Manacher unprompted wastes time and invites bugs.

```java
private int expand(String s, int l, int r) {
    while (l >= 0 && r < s.length() && s.charAt(l) == s.charAt(r)) { l--; r++; }
    return r - l - 1;
}
```

---

### Tip 4 — Rolling hash: always `long` + prime mod

A polynomial hash overflows 32-bit `int` instantly. Use `long`, a prime modulus, and reduce after every multiply.

```java
static final long MOD = 1_000_000_007L;
static final long BASE = 131L;            // or 31 / 256
hash = (hash * BASE + s.charAt(i)) % MOD;
```

---

### Tip 5 — `(x % m + m) % m` for negative remainders

Rolling-hash updates **subtract** the leaving character, which can go negative. Java's `%` keeps the sign of the dividend, so normalize:

```java
windowHash = ((windowHash - leaving) % MOD + MOD) % MOD;   // negative-safe
```

Forgetting this gives sporadic wrong answers that pass small tests and fail large ones.

---

### Tip 6 — Verify on a hash collision

A hash match is **necessary, not sufficient**. Two distinct substrings can share a hash (spurious hit). On every hash match, either confirm the characters or use double hashing.

```java
if (windowHash == patternHash && text.regionMatches(i, pattern, 0, m)) return i;
```

---

### Tip 7 — Z-array and LPS carry the same information

`lps[i]` = longest border *ending* at `i`; `z[i]` = longest prefix-match *starting* at `i`. Either solves single-pattern matching in O(n+m) — pick whichever coordinate system the problem makes natural (concatenate `pattern + '#' + text` for Z; build `lps` on the pattern for KMP). Don't waste energy converting between them mid-interview.

---

### Tip 8 — A string of length `n` has `2n-1` palindrome centers

`n` odd-length centers (each character) plus `n-1` even-length centers (each gap). When counting or searching palindromes, iterate *both* per index:

```java
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
