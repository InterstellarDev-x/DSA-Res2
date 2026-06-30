> **Topic:** [Advanced Strings](../README.md) · **Section:** Interview Problems

# Microsoft — Advanced Strings (Interview Deep Dives)

Microsoft interviews weight **communication** heavily: a working solution you cannot explain scores poorly. The headline problems here — Repeated Substring Pattern and `strStr` — are both excuses to make you *teach* the KMP failure function to a non-expert.

---

## 1. Repeated Substring Pattern (LC 459) — period via LPS, proven

> Return `true` if `s` can be constructed by concatenating multiple copies of some substring.

### The lemma and its proof

Let `n = s.length()` and `k = lps[n-1]` (the longest border of the whole string). Define the candidate period `p = n - k`.

**Claim:** `s` is built from repeated copies of a substring **iff** `k != 0` and `n % p == 0`.

**Why `p` is a period.** `lps[n-1] = k` means the length-`k` prefix equals the length-`k` suffix:
```
s[0..k)   ==  s[n-k..n)
```
Aligning these two equal blocks forces `s[j] == s[j + p]` for every valid `j` (the suffix is the prefix shifted right by `p = n - k`). By definition that makes `p` a period of `s`.

**Why `n % p == 0` completes it.** If `p` divides `n`, the block `s[0..p)` tiles `s` exactly `n / p` times — `s` is that block repeated. If `p` does **not** divide `n`, the period does not align to a clean tiling, so `s` is *not* a pure repetition (e.g. `"abcab"`: `p = 3`, `5 % 3 != 0`).

**Why the `k != 0` guard.** A string with no border at all (e.g. `"abc"`) has `k = 0`, so `p = n`, and `n % n == 0` would falsely say "yes." The guard rejects the trivial single-copy case.

```java
class Solution {
    public boolean repeatedSubstringPattern(String s) {
        int n = s.length();
        int[] lps = buildLPS(s);
        int k = lps[n - 1];
        return k != 0 && n % (n - k) == 0;
    }

    private int[] buildLPS(String s) {
        int n = s.length();
        int[] lps = new int[n];
        int len = 0, i = 1;
        while (i < n) {
            if (s.charAt(i) == s.charAt(len)) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        return lps;
    }
}
```

**Complexity:** O(n) time and space.

**The slick alternative.** `(s + s).indexOf(s, 1) != s.length()`. Concatenating `s` with itself and searching for `s` starting at index 1: if `s` is periodic, a copy reappears before the natural index `n`. Be ready to justify it — Microsoft will ask *why* it works (the period must reappear within the doubled string before position `n`).

---

## 2. Implement strStr() (LC 28) — design and explanation

> Return the first index of `needle` in `haystack`, or `-1`.

```java
class Solution {
    public int strStr(String haystack, String needle) {
        if (needle.isEmpty()) return 0;
        int[] lps = buildLPS(needle);
        int i = 0, j = 0, n = haystack.length(), m = needle.length();
        while (i < n) {
            if (haystack.charAt(i) == needle.charAt(j)) {
                i++; j++;
                if (j == m) return i - j;
            } else if (j > 0) {
                j = lps[j - 1];   // fall back using the border; text pointer stays
            } else {
                i++;
            }
        }
        return -1;
    }

    private int[] buildLPS(String s) {
        int n = s.length();
        int[] lps = new int[n];
        int len = 0, i = 1;
        while (i < n) {
            if (s.charAt(i) == s.charAt(len)) { lps[i++] = ++len; }
            else if (len > 0) { len = lps[len - 1]; }
            else { lps[i++] = 0; }
        }
        return lps;
    }
}
```

**Design notes to verbalize:** handle the empty needle (return 0), note that `haystack.indexOf(needle)` is the production answer, and explain that KMP's win is *never moving the text pointer `i` backward* — on a mismatch we only rewind `j` within the pattern.

---

## 3. Explaining the KMP failure function to a non-expert

Microsoft loves "explain it to me like I don't know algorithms." Here is a script that works:

> "Imagine I'm matching the word `ABABC` against a long text and I get 4 letters in — `ABAB` matched — but the 5th fails. The naive thing is to throw away all that progress and restart one character over. But `ABAB` ends in `AB`, which is also how the pattern *begins*. So I already know the next two characters of the pattern (`AB`) are matched — I don't re-check them. The **failure function** is just a precomputed table that says, for each position, 'if you fail here, how many characters at the front are already guaranteed to match.' That number is the length of the longest piece that is both a **prefix and a suffix** of what matched so far."

Then show the table for `ABABC`:

| index | 0 | 1 | 2 | 3 | 4 |
|-------|---|---|---|---|---|
| char  | A | B | A | B | C |
| lps   | 0 | 0 | 1 | 2 | 0 |

> "When `C` (index 4) fails, `lps[3] = 2` tells me 2 front characters are already matched, so I resume comparing at pattern index 2 instead of 0 — and crucially I never move backward in the *text*. That's why it runs in O(n + m): each text character is examined a constant number of times."

The full step-by-step construction and a longer dry-run live in [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md).

---

## Edge cases across all three

- Empty needle / empty `s` → return 0 / handle `n == 0`.
- Single-character pattern → degenerates to a linear scan, still correct.
- Repeated Substring Pattern on a single char `"a"` → `k = 0`, returns `false` (correct; no repetition of a *proper* substring).

---

## Related

- OA frequency: [OA-Qns → Microsoft](../OA-Qns/Microsoft.md)
- Other deep dives: [Amazon](Amazon.md) · [Google](Google.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
