> **Topic:** [Advanced Strings](../README.md) · **Tips 2 of 4**

# Common Mistakes — Advanced Strings

Ten wrong/correct code pairs. Each is a bug that passes small tests and fails the hard ones.

---

### Mistake 1 — LPS off-by-one (advancing `i` on fallback)

```java
// WRONG: advances i on the fallback branch -> wrong borders, sometimes infinite confusion
else if (len > 0) { len = lps[len - 1]; i++; }
```
```java
// CORRECT: on fallback, ONLY len changes; i stays put
else if (len > 0) { len = lps[len - 1]; }
```

---

### Mistake 2 — KMP not resetting `j` via `lps` on mismatch

```java
// WRONG: resets j to 0, throwing away matched border -> O(nm), can even miss matches with overlaps
} else { j = 0; i++; }
```
```java
// CORRECT: fall back to the longest border; only advance i when j is already 0
} else if (j > 0) { j = lps[j - 1]; }
else { i++; }
```

---

### Mistake 3 — Rolling hash overflow without `long`

```java
// WRONG: int overflows the moment BASE * hash exceeds ~2.1e9
int hash = (hash * BASE + s.charAt(i)) % MOD;
```
```java
// CORRECT: long throughout, prime mod
long hash = (hash * BASE + s.charAt(i)) % MOD;   // MOD = 1_000_000_007L
```

---

### Mistake 4 — Forgetting the negative-mod fix

```java
// WRONG: subtraction can be negative; Java % keeps the sign -> corrupt hash
windowHash = (windowHash - leaving) % MOD;
```
```java
// CORRECT: normalize into [0, MOD)
windowHash = ((windowHash - leaving) % MOD + MOD) % MOD;
```

---

### Mistake 5 — Not verifying a hash collision

```java
// WRONG: returns on hash equality alone -> false positive on a spurious hit
if (windowHash == patternHash) return i;
```
```java
// CORRECT: confirm characters (or use double hashing)
if (windowHash == patternHash && text.regionMatches(i, pattern, 0, m)) return i;
```

---

### Mistake 6 — Manacher transformed-string indexing

```java
// WRONG: forgets the separators, indexes the original string with transformed positions
char c = s.charAt(center);           // 'center' is an index into "^#a#b#a#$"
```
```java
// CORRECT: build the transformed string and map back with (center - 1) / 2 for the original
StringBuilder t = new StringBuilder("^");
for (char ch : s.toCharArray()) { t.append('#').append(ch); }
t.append("#$");
// original start index of a palindrome of radius r centered at i in t:
int origStart = (i - r) / 2;
```

---

### Mistake 7 — Expand-around-center missing even centers

```java
// WRONG: only odd-length palindromes are found
for (int i = 0; i < n; i++) best = Math.max(best, expand(s, i, i));
```
```java
// CORRECT: both odd (i,i) and even (i,i+1) centers -> 2n-1 total
for (int i = 0; i < n; i++) {
    best = Math.max(best, Math.max(expand(s, i, i), expand(s, i, i + 1)));
}
```

---

### Mistake 8 — Repeated Substring Pattern missing the `lps[n-1] != 0` guard

```java
// WRONG: "abc" has lps[n-1]=0, p=n, n%n==0 -> falsely returns true
return n % (n - lps[n - 1]) == 0;
```
```java
// CORRECT: a string with no border is not a repetition
int k = lps[n - 1];
return k != 0 && n % (n - k) == 0;
```

---

### Mistake 9 — Shortest Palindrome wrong separator (collision)

```java
// WRONG: no separator -> border can exceed |s|, giving a wrong palindromic-prefix length
String combined = s + new StringBuilder(s).reverse();
```
```java
// CORRECT: a sentinel that cannot appear in s caps every border at |s|
String combined = s + "#" + new StringBuilder(s).reverse();
```

---

### Mistake 10 — Z-array window L/R update bug

```java
// WRONG: forgets to advance r, or updates l/r without the i+z[i] > r check -> breaks O(n)
for (int i = 1; i < n; i++) {
    while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
}
```
```java
// CORRECT: maintain the [l, r] z-box; reuse mirror values; advance r only when extended
int l = 0, r = 0;
for (int i = 1; i < n; i++) {
    if (i < r) z[i] = Math.min(r - i, z[i - l]);
    while (i + z[i] < n && s.charAt(z[i]) == s.charAt(i + z[i])) z[i]++;
    if (i + z[i] > r) { l = i; r = i + z[i]; }
}
```

---

## Related

- [Coding Tips](Coding%20Tips.md) · [Algorithm Selection](Algorithm%20Selection.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
