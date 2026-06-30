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

```java
public static int[] buildZ(String s) {
    int n = s.length();
    int[] z = new int[n];
    int L = 0, R = 0;            // current rightmost match window [L, R)
    for (int i = 1; i < n; i++) {
        if (i < R) {
            z[i] = Math.min(R - i, z[i - L]);   // reuse mirror, capped at window
        }
        // try to extend the match beyond what we reused
        while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) {
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

```java
public static List<Integer> zSearch(String text, String pattern) {
    String combined = pattern + '$' + text;
    int[] z = buildZ(combined);
    int m = pattern.length();
    List<Integer> result = new ArrayList<>();
    for (int i = m + 1; i < combined.length(); i++) {
        if (z[i] == m) {
            result.add(i - m - 1);   // offset back into the original text
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
- **Modulus `M`:** a large prime such as `1_000_000_007L` to keep values in `long` range and spread collisions.
- **Precompute powers** `B^0..B^(m)` once so a window hash updates in O(1).

```java
static final long MOD = 1_000_000_007L;
static final long BASE = 131L;

// Precompute base powers up to length n.
static long[] powers(int n) {
    long[] p = new long[n + 1];
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

```java
class Solution {
    public List<Integer> findAnagrams(String s, String p) {
        List<Integer> res = new ArrayList<>();
        int n = s.length(), m = p.length();
        if (n < m) return res;
        int[] need = new int[26], win = new int[26];
        for (char c : p.toCharArray()) need[c - 'a']++;
        for (int i = 0; i < n; i++) {
            win[s.charAt(i) - 'a']++;            // add right edge
            if (i >= m) win[s.charAt(i - m) - 'a']--;  // remove left edge
            if (i >= m - 1 && Arrays.equals(win, need)) {
                res.add(i - m + 1);
            }
        }
        return res;
    }
}
```

**Complexity:** O(n * 26) ≈ O(n) time, O(1) space. `Arrays.equals` over 26 ints is constant.

**Hashing alternative.** Instead of comparing arrays you can keep a *commutative* hash of the window — e.g. a fixed-length 26-vector difference counter, or a polynomial over the sorted multiset. Frequency comparison is simpler and exact, so prefer it for anagrams; rolling hash earns its keep when *order matters* (exact substring), not for anagrams.

---

## 5. Problem 9a — Longest Duplicate Substring (LC 1044, Hard)

> Find the longest substring that occurs at least twice (may overlap). Return any such substring (or "" if none).

**Strategy: binary search on the length + Rabin-Karp.** A duplicate of length `L` existing implies a duplicate of every length `< L` exists (a prefix of it), so the predicate "exists a duplicate of length `L`" is monotonic — binary-search `L` in `[1, n-1]`. For a fixed `L`, slide a rolling hash over all length-`L` windows and store hashes in a set; a repeat hash (verified) means a duplicate.

```java
class Solution {
    static final long MOD = (1L << 61) - 1;   // large Mersenne prime, room in long
    static final long BASE = 131L;

    public String longestDupSubstring(String s) {
        int n = s.length();
        int[] a = new int[n];
        for (int i = 0; i < n; i++) a[i] = s.charAt(i) - 'a' + 1;
        int lo = 1, hi = n - 1, start = -1, bestLen = 0;
        while (lo <= hi) {
            int mid = (lo + hi) >>> 1;
            int pos = search(a, mid);
            if (pos != -1) { start = pos; bestLen = mid; lo = mid + 1; }
            else { hi = mid - 1; }
        }
        return start == -1 ? "" : s.substring(start, start + bestLen);
    }

    // Returns the start index of some duplicated substring of length L, or -1.
    private int search(int[] a, int L) {
        long pow = 1;
        for (int i = 0; i < L; i++) pow = mulmod(pow, BASE);
        long h = 0;
        for (int i = 0; i < L; i++) h = (mulmod(h, BASE) + a[i]) % MOD;
        HashMap<Long, List<Integer>> seen = new HashMap<>();
        seen.computeIfAbsent(h, k -> new ArrayList<>()).add(0);
        for (int i = L; i < a.length; i++) {
            h = (mulmod(h, BASE) + a[i] - mulmod(a[i - L], pow) % MOD + MOD * MOD) % MOD;
            int startIdx = i - L + 1;
            List<Integer> cand = seen.get(h);
            if (cand != null) {
                // verify to defeat spurious hash hits (collision safety)
                for (int j : cand) if (equalsAt(a, j, startIdx, L)) return startIdx;
            }
            seen.computeIfAbsent(h, k -> new ArrayList<>()).add(startIdx);
        }
        return -1;
    }

    private boolean equalsAt(int[] a, int x, int y, int L) {
        for (int k = 0; k < L; k++) if (a[x + k] != a[y + k]) return false;
        return true;
    }

    private long mulmod(long x, long y) {
        return (x % MOD) * (y % MOD) % MOD;   // fits because MOD < 2^61, x,y < MOD
    }
}
```

**Complexity:** O(n log n) average — `log n` binary-search steps, each an O(n) rolling pass. **Double hashing** (two independent (BASE, MOD) pairs, compare both) makes a collision astronomically unlikely and lets you skip the explicit verification; we kept the verification here for a guaranteed-correct answer.

---

## 6. Problem 9b — Distinct Echo Substrings (LC 1316, Hard)

> Count the number of **distinct** substrings that are a concatenation of some string with itself (i.e. of the form `a + a`, an "echo").

A substring `s[i..i+2k-1]` is an echo iff its first half `s[i..i+k-1]` equals its second half `s[i+k..i+2k-1]`. With prefix hashes, comparing two halves is O(1); store every distinct echo as a (hash) to dedupe.

```java
class Solution {
    public int distinctEchoSubstrings(String text) {
        int n = text.length();
        long MOD = 1_000_000_007L, BASE = 131L;
        long[] h = new long[n + 1], pw = new long[n + 1];
        pw[0] = 1;
        for (int i = 0; i < n; i++) {
            h[i + 1] = (h[i] * BASE + text.charAt(i)) % MOD;
            pw[i + 1] = pw[i] * BASE % MOD;
        }
        Set<Long> distinct = new HashSet<>();
        for (int len = 1; 2 * len <= n; len++) {           // half-length
            for (int i = 0; i + 2 * len <= n; i++) {
                long first  = sub(h, pw, MOD, i, i + len);
                long second = sub(h, pw, MOD, i + len, i + 2 * len);
                if (first == second) distinct.add(first); // dedupe by half-hash
            }
        }
        return distinct.size();
    }

    // hash of text[l..r) from prefix hashes.
    private long sub(long[] h, long[] pw, long MOD, int l, int r) {
        return ((h[r] - h[l] * pw[r - l]) % MOD + MOD) % MOD;
    }
}
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
