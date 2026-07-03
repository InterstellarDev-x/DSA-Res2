> **Topic:** [Advanced Strings](../README.md) · **Tips 2 of 4**

# Common Mistakes — Advanced Strings

Ten wrong/correct code pairs. Each is a bug that passes small tests and fails the hard ones.

---

### Mistake 1 — LPS off-by-one (advancing `i` on fallback)

```cpp
// WRONG: advances i on the fallback branch -> wrong borders, sometimes infinite confusion
else if (len > 0) { len = lps[len - 1]; i++; }
```
```cpp
// CORRECT: on fallback, ONLY len changes; i stays put
else if (len > 0) { len = lps[len - 1]; }
```

---

### Mistake 2 — KMP not resetting `j` via `lps` on mismatch

```cpp
// WRONG: resets j to 0, throwing away matched border -> O(nm), can even miss matches with overlaps
} else { j = 0; i++; }
```
```cpp
// CORRECT: fall back to the longest border; only advance i when j is already 0
} else if (j > 0) { j = lps[j - 1]; }
else { i++; }
```

---

### Mistake 3 — Rolling hash overflow without `long long`

```cpp
// WRONG: int overflows the moment BASE * hash exceeds ~2.1e9
int hash = (hash * BASE + s[i]) % MOD;
```
```cpp
// CORRECT: long long throughout, prime mod
long long hash = (hash * BASE + s[i]) % MOD;   // MOD = 1'000'000'007LL
```

---

### Mistake 4 — Forgetting the negative-mod fix

```cpp
// WRONG: subtraction can be negative; C++ % keeps the sign -> corrupt hash
windowHash = (windowHash - leaving) % MOD;
```
```cpp
// CORRECT: normalize into [0, MOD)
windowHash = ((windowHash - leaving) % MOD + MOD) % MOD;
```

---

### Mistake 5 — Not verifying a hash collision

```cpp
// WRONG: returns on hash equality alone -> false positive on a spurious hit
if (windowHash == patternHash) return i;
```
```cpp
// CORRECT: confirm characters (or use double hashing)
if (windowHash == patternHash && text.compare(i, m, pattern) == 0) return i;
```

---

### Mistake 6 — Manacher transformed-string indexing

```cpp
// WRONG: forgets the separators, indexes the original string with transformed positions
char c = s[center];           // 'center' is an index into "^#a#b#a#$"
```
```cpp
// CORRECT: build the transformed string and map back with (center - 1) / 2 for the original
string t = "^";
for (char ch : s) { t += '#'; t += ch; }
t += "#$";
// original start index of a palindrome of radius r centered at i in t:
int origStart = (i - r) / 2;
```

---

### Mistake 7 — Expand-around-center missing even centers

```cpp
// WRONG: only odd-length palindromes are found
for (int i = 0; i < n; i++) best = max(best, expand(s, i, i));
```
```cpp
// CORRECT: both odd (i,i) and even (i,i+1) centers -> 2n-1 total
for (int i = 0; i < n; i++) {
    best = max(best, max(expand(s, i, i), expand(s, i, i + 1)));
}
```

---

### Mistake 8 — Repeated Substring Pattern missing the `lps[n-1] != 0` guard

```cpp
// WRONG: "abc" has lps[n-1]=0, p=n, n%n==0 -> falsely returns true
return n % (n - lps[n - 1]) == 0;
```
```cpp
// CORRECT: a string with no border is not a repetition
int k = lps[n - 1];
return k != 0 && n % (n - k) == 0;
```

---

### Mistake 9 — Shortest Palindrome wrong separator (collision)

```cpp
// WRONG: no separator -> border can exceed |s|, giving a wrong palindromic-prefix length
string combined = s + string(s.rbegin(), s.rend());
```
```cpp
// CORRECT: a sentinel that cannot appear in s caps every border at |s|
string combined = s + "#" + string(s.rbegin(), s.rend());
```

---

### Mistake 10 — Z-array window L/R update bug

```cpp
// WRONG: forgets to advance r, or updates l/r without the i+z[i] > r check -> breaks O(n)
for (int i = 1; i < n; i++) {
    while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
}
```
```cpp
// CORRECT: maintain the [l, r] z-box; reuse mirror values; advance r only when extended
int l = 0, r = 0;
for (int i = 1; i < n; i++) {
    if (i < r) z[i] = min(r - i, z[i - l]);
    while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
    if (i + z[i] > r) { l = i; r = i + z[i]; }
}
```

---

## Related

- [Coding Tips](Coding%20Tips.md) · [Algorithm Selection](Algorithm%20Selection.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
