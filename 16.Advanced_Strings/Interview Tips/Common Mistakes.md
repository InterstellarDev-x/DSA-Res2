> **Topic:** [Advanced Strings](../README.md) · **Tips 2 of 4**

# Common Mistakes — Advanced Strings

Ten wrong/correct code pairs. Each is a bug that passes small tests and fails the hard ones.

---

### Mistake 1 — LPS off-by-one (advancing `i` on fallback)

```rust
// WRONG: advances i on the fallback branch -> wrong borders, sometimes infinite confusion
else if len > 0 { len = lps[len - 1]; i += 1; }
```
```rust
// CORRECT: on fallback, ONLY len changes; i stays put
else if len > 0 { len = lps[len - 1]; }
```

---

### Mistake 2 — KMP not resetting `j` via `lps` on mismatch

```rust
// WRONG: resets j to 0, throwing away matched border -> O(nm), can even miss matches with overlaps
} else { j = 0; i += 1; }
```
```rust
// CORRECT: fall back to the longest border; only advance i when j is already 0
} else if j > 0 { j = lps[j - 1]; }
else { i += 1; }
```

---

### Mistake 3 — Rolling hash overflow without `i64`

```rust
// WRONG: i32 overflows the moment BASE * hash exceeds ~2.1e9
let hash: i32 = (hash * BASE + s[i] as i32) % MOD;
```
```rust
// CORRECT: i64 throughout, prime mod
let hash: i64 = (hash * BASE + s[i] as i64) % MOD;   // MOD = 1_000_000_007i64
```

---

### Mistake 4 — Forgetting the negative-mod fix

```rust
// WRONG: subtraction can be negative; Rust % keeps the sign -> corrupt hash
window_hash = (window_hash - leaving) % MOD;
```
```rust
// CORRECT: normalize into [0, MOD)
window_hash = ((window_hash - leaving) % MOD + MOD) % MOD;
```

---

### Mistake 5 — Not verifying a hash collision

```rust
// WRONG: returns on hash equality alone -> false positive on a spurious hit
if window_hash == pattern_hash { return Some(i); }
```
```rust
// CORRECT: confirm characters (or use double hashing)
if window_hash == pattern_hash && &text[i..i + m] == pattern { return Some(i); }
```

---

### Mistake 6 — Manacher transformed-string indexing

```rust
// WRONG: forgets the separators, indexes the original string with transformed positions
let c = s.as_bytes()[center] as char;  // 'center' is an index into "^#a#b#a#$"
```
```rust
// CORRECT: build the transformed string and map back with (center - 1) / 2 for the original
let mut t = String::from("^");
for ch in s.chars() { t.push('#'); t.push(ch); }
t.push_str("#$");
// original start index of a palindrome of radius r centered at i in t:
let orig_start = (i - r) / 2;
```

---

### Mistake 7 — Expand-around-center missing even centers

```rust
// WRONG: only odd-length palindromes are found
for i in 0..n { best = best.max(expand(&s, i, i)); }
```
```rust
// CORRECT: both odd (i,i) and even (i,i+1) centers -> 2n-1 total
for i in 0..n {
    best = best.max(expand(&s, i, i).max(expand(&s, i, i + 1)));
}
```

---

### Mistake 8 — Repeated Substring Pattern missing the `lps[n-1] != 0` guard

```rust
// WRONG: "abc" has lps[n-1]=0, p=n, n%n==0 -> falsely returns true
return n % (n - lps[n - 1]) == 0;
```
```rust
// CORRECT: a string with no border is not a repetition
let k = lps[n - 1];
return k != 0 && n % (n - k) == 0;
```

---

### Mistake 9 — Shortest Palindrome wrong separator (collision)

```rust
// WRONG: no separator -> border can exceed |s|, giving a wrong palindromic-prefix length
let combined: String = s.chars().chain(s.chars().rev()).collect();
```
```rust
// CORRECT: a sentinel that cannot appear in s caps every border at |s|
let combined: String = s.chars().chain(std::iter::once('#')).chain(s.chars().rev()).collect();
```

---

### Mistake 10 — Z-array window L/R update bug

```rust
// WRONG: forgets to advance r, or updates l/r without the i+z[i] > r check -> breaks O(n)
for i in 1..n {
    while i + z[i] < n && s[z[i]] == s[i + z[i]] { z[i] += 1; }
}
```
```rust
// CORRECT: maintain the [l, r] z-box; reuse mirror values; advance r only when extended
let (mut l, mut r) = (0usize, 0usize);
for i in 1..n {
    if i < r { z[i] = (r - i).min(z[i - l]); }
    while i + z[i] < n && s[z[i]] == s[i + z[i]] { z[i] += 1; }
    if i + z[i] > r { l = i; r = i + z[i]; }
}
```

---

## Related

- [Coding Tips](Coding%20Tips.md) · [Algorithm Selection](Algorithm%20Selection.md) · [Complexity Analysis](Complexity%20Analysis.md)
- Back to [topic README](../README.md)

---

> **Last Updated:** 2026-06-26
