# Microsoft Interview Problems — Bit Manipulation

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Interview Problems
> **Last Updated:** 2026-06-26

---

## Question Bank

| # | Problem | Level | Difficulty | Frequency | Pattern | LeetCode |
|---|---------|-------|-----------|----------|---------|---------|
| 1 | Divide Two Integers | SDE II | Medium | ⭐⭐⭐⭐ | [Bit Basics](../Patterns/Bit%20Basics.md) | [29](https://leetcode.com/problems/divide-two-integers/) |
| 2 | Reverse Bits | SDE I | Easy | ⭐⭐⭐⭐ | [Bit Basics](../Patterns/Bit%20Basics.md) | [190](https://leetcode.com/problems/reverse-bits/) |
| 3 | Single Number | SDE I | Easy | ⭐⭐⭐ | [XOR Tricks](../Patterns/XOR%20Tricks.md) | [136](https://leetcode.com/problems/single-number/) |
| 4 | Counting Bits | SDE I | Easy | ⭐⭐⭐ | [Counting Bits](../Patterns/Counting%20Bits.md) | [338](https://leetcode.com/problems/counting-bits/) |

---

## Deep Dive: Divide Two Integers

### Key Considerations

1. **No `*`, `/`, `%`** — use bit shifts
2. **Overflow:** `INT_MIN / -1 = 2^31` — exceeds `INT_MAX`, return `INT_MAX`
3. **Signs:** Compute absolute values; apply sign at end
4. **Use `long`** — `abs(INT_MIN)` overflows `int`

```cpp
#include <bits/stdc++.h>
using namespace std;

int divide(int dividend, int divisor) {
    if (dividend == INT_MIN && divisor == -1) return INT_MAX;
    long dvd = abs((long) dividend);
    long dvs = abs((long) divisor);
    int sign = (dividend > 0) == (divisor > 0) ? 1 : -1;
    long result = 0;

    while (dvd >= dvs) {
        long tmp = dvs, multiple = 1;
        while (dvd >= (tmp << 1)) { tmp <<= 1; multiple <<= 1; }
        dvd -= tmp;
        result += multiple;
    }
    return (int)(sign * result);
}
```

**Complexity:** O(log² n) — outer loop runs log(n) times, inner loop runs log(n) times.

### Microsoft Follow-ups

**Q: What if you could only use shifts, not multiplication in the inner loop check?**
`tmp << 1` IS a shift — this is already compliant.

**Q: Handle when dvd = LONG_MAX?**
`tmp << 1` could overflow `long`. Guard: `dvd >= (tmp << 1) && (tmp << 1) > 0`.

---

## Related Files

- [Microsoft OA-Qns](../OA-Qns/Microsoft.md)
- [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
