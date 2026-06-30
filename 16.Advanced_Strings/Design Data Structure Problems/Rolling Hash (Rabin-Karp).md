> **Topic:** [Advanced Strings](../README.md) · **Section:** Design

# Rolling Hash (Rabin-Karp)

A **rolling hash** turns a string into a number so that two substrings can be compared in **O(1)** after O(n) preprocessing. The classic use is **Rabin-Karp** substring matching, but the same primitive powers Longest Duplicate Substring, Distinct Echo Substrings, plagiarism detection, and substring deduplication.

The design question almost always reduces to: *"compare many substrings for equality, fast."* Hashing answers it — provided you handle overflow, negative remainders, and spurious collisions correctly.

---

## 1. Polynomial Hash

Treat a string `s = s[0] s[1] ... s[m-1]` as a base-`B` number under a prime modulus `MOD`:

```
hash(s) = (s[0]·B^(m-1) + s[1]·B^(m-2) + ... + s[m-1]·B^0) mod MOD
```

- **Base `B`:** any value larger than the alphabet works. `31` is the canonical small base for lowercase strings; `256` treats each byte as a digit. Both are fine — pick one and stay consistent.
- **Modulus `MOD`:** a large prime, `1_000_000_007L`. A prime modulus spreads hashes uniformly and keeps the probability of a random collision near `1/MOD`.
- **`long` everywhere:** `B·hash` can reach `31 · 10^9 ≈ 3.1·10^10`, which overflows a 32-bit `int`. Use `long` and reduce mod `MOD` after every multiply.

---

## 2. Rolling Update Formula

The "rolling" part: when the window slides one character to the right (drop `s[i]`, add `s[i+m]`), recompute the hash in O(1) instead of O(m):

```
newHash = ( (oldHash - s[i]·B^(m-1)) · B + s[i+m] ) mod MOD
```

- Subtract the contribution of the leaving character (its weight is the top power `B^(m-1)`).
- Multiply the whole window by `B` (every remaining character shifts up one power).
- Add the entering character (weight `B^0`).

Because subtraction can produce a **negative** value under modular arithmetic, always normalize:

```java
hash = ((hash % MOD) + MOD) % MOD;
```

---

## 3. Full Rabin-Karp — Single Hash

```java
public class RabinKarp {

    private static final long MOD = 1_000_000_007L;
    private static final long BASE = 256L; // each char treated as a base-256 digit

    /** Returns the first index where pattern occurs in text, or -1. */
    public int search(String text, String pattern) {
        int n = text.length(), m = pattern.length();
        if (m == 0) return 0;
        if (m > n) return -1;

        // highPow = BASE^(m-1) mod MOD  -> weight of the leftmost character.
        long highPow = 1L;
        for (int i = 0; i < m - 1; i++) {
            highPow = (highPow * BASE) % MOD;
        }

        long patternHash = 0L;
        long windowHash = 0L;
        for (int i = 0; i < m; i++) {
            patternHash = (patternHash * BASE + pattern.charAt(i)) % MOD;
            windowHash = (windowHash * BASE + text.charAt(i)) % MOD;
        }

        for (int i = 0; i + m <= n; i++) {
            // Hash match -> verify the actual characters (guard against spurious hits).
            if (windowHash == patternHash && matches(text, i, pattern)) {
                return i;
            }
            // Roll the window forward by one, unless we are at the last position.
            if (i + m < n) {
                long leaving = (text.charAt(i) * highPow) % MOD;
                windowHash = ((windowHash - leaving) % MOD + MOD) % MOD; // negative-safe
                windowHash = (windowHash * BASE + text.charAt(i + m)) % MOD;
            }
        }
        return -1;
    }

    /** Character-by-character verification; only called when hashes collide. */
    private boolean matches(String text, int start, String pattern) {
        for (int k = 0; k < pattern.length(); k++) {
            if (text.charAt(start + k) != pattern.charAt(k)) return false;
        }
        return true;
    }
}
```

**Why `matches`?** Two distinct substrings can share a hash (a **spurious hit** / collision). A hash match is *necessary* but not *sufficient* — never return on a hash match alone. The verification keeps Rabin-Karp **correct**; it stays O(n+m) on average because collisions are rare.

---

## 4. Spurious Hits and Collision Probability

With a single prime modulus near `10^9`, the chance any one comparison collides is about `1/MOD ≈ 10^-9`. Over `q` comparisons the expected number of false hits is about `q/MOD` — negligible for normal inputs, which is why verification is cheap.

But adversarial inputs (or anti-hash test cases on Codeforces, and occasionally on hard LeetCode problems) can be crafted to collide a single fixed `(BASE, MOD)`. The defense is **double hashing**.

---

## 5. Double Hashing

Maintain two independent hashes with different `(BASE, MOD)` pairs and treat two substrings as equal only when **both** match. The effective collision probability becomes roughly `1/(MOD1·MOD2) ≈ 10^-18`, small enough to skip character verification in many competitive settings (though verifying is still safest in interviews).

```java
public class DoubleHash {

    private static final long MOD1 = 1_000_000_007L;
    private static final long MOD2 = 998_244_353L;
    private static final long BASE1 = 131L;
    private static final long BASE2 = 137L;

    private final long[] h1, h2;   // prefix hashes
    private final long[] p1, p2;   // base powers
    private final int n;

    public DoubleHash(String s) {
        n = s.length();
        h1 = new long[n + 1];
        h2 = new long[n + 1];
        p1 = new long[n + 1];
        p2 = new long[n + 1];
        p1[0] = 1L;
        p2[0] = 1L;
        for (int i = 0; i < n; i++) {
            h1[i + 1] = (h1[i] * BASE1 + s.charAt(i)) % MOD1;
            h2[i + 1] = (h2[i] * BASE2 + s.charAt(i)) % MOD2;
            p1[i + 1] = (p1[i] * BASE1) % MOD1;
            p2[i + 1] = (p2[i] * BASE2) % MOD2;
        }
    }

    /** Packed hash of substring s[l..r) (half-open), 0 <= l <= r <= n. */
    public long hash(int l, int r) {
        long a = ((h1[r] - h1[l] * p1[r - l]) % MOD1 + MOD1) % MOD1;
        long b = ((h2[r] - h2[l] * p2[r - l]) % MOD2 + MOD2) % MOD2;
        return a * MOD2 + b; // pack two 32-bit-ish values into one comparable long
    }

    /** True if s[l1..l1+len) equals s[l2..l2+len). */
    public boolean equal(int l1, int l2, int len) {
        return hash(l1, l1 + len) == hash(l2, l2 + len);
    }
}
```

The **prefix-hash** form above (`h[r] - h[l]·B^(r-l)`) is the most reusable shape: build once in O(n), then query the hash of *any* substring in O(1). This is exactly what Longest Duplicate Substring needs.

---

## 6. Complexity

| Operation | Average | Worst |
|-----------|---------|-------|
| Rabin-Karp search | O(n + m) | O(nm) (all hashes collide, every window verified fully) |
| Prefix-hash build | O(n) | O(n) |
| Substring hash query | O(1) | O(1) |
| Space | O(1) (rolling) or O(n) (prefix array) | same |

The O(nm) worst case is the reason double hashing matters: it makes adversarial all-collide inputs astronomically unlikely, so the average case holds in practice.

---

## 7. Follow-ups

### Substring deduplication ("count distinct substrings of length k")
Slide a window of length `k`, push each window's hash into a `HashSet<Long>`, and the set size is the count of distinct length-`k` substrings. Use double hashing to avoid counting two different substrings as one. This generalizes to Distinct Echo Substrings (LC 1316), where you hash both halves and require them equal.

```java
public int distinctSubstringsOfLength(String s, int k) {
    int n = s.length();
    if (k > n) return 0;
    DoubleHash dh = new DoubleHash(s);
    java.util.HashSet<Long> seen = new java.util.HashSet<>();
    for (int i = 0; i + k <= n; i++) {
        seen.add(dh.hash(i, i + k));
    }
    return seen.size();
}
```

### Longest Duplicate Substring (LC 1044) — binary search on length + Rabin-Karp
The answer length is **monotone**: if a duplicate of length `L` exists, one of length `L-1` exists too. Binary search `L` in `[1, n-1]`; for each `L`, hash every length-`L` window and check a `HashSet` for a repeat in O(n). Total O(n log n). Double hashing is essential here because LeetCode's hard cases probe single-hash collisions.

### Plagiarism / fingerprinting framing
Break each document into overlapping `k`-grams, hash each with a rolling hash, and compare the resulting sets of fingerprints (Jaccard similarity). Winnowing selects a deterministic subset of fingerprints so two documents that share a passage share fingerprints regardless of alignment — the production form of "compare many substrings fast."

---

## Cross-links

- [String Matching (KMP)](../Patterns/String%20Matching%20(KMP).md) — the deterministic O(n+m) alternative to hashing for single-pattern search.
- [Z Algorithm and Hashing](../Patterns/Z%20Algorithm%20and%20Hashing.md) — uses these exact prefix hashes for Longest Duplicate Substring and Distinct Echo Substrings.
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md) — average vs worst case for all string algorithms.

---

> **Last Updated:** 2026-06-26
