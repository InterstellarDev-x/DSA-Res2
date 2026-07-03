# Bitset

> **Topic:** [Bit Manipulation](../README.md) · **Section:** Design Data Structure Problems
> **LeetCode:** [2166 — Design Bitset](https://leetcode.com/problems/design-bitset/)

---

## Problem

Design a `Bitset` with a fixed size. Supports:
- `fix(idx)` — set bit idx to 1
- `unfix(idx)` — set bit idx to 0
- `flip()` — invert all bits
- `all()` — true if all bits are 1
- `one()` — true if at least one bit is 1
- `count()` — number of 1-bits
- `toString()` — binary string representation

All operations O(1) except `toString()` O(n).

---

## Design

Use a `vector<long long>` array — each `long long` holds 64 bits. Key challenge: `flip()` must be O(1), not O(n).

**Lazy flip:** Maintain a `flipped` boolean. Flipping twice = no-op. On any read, XOR the actual bit with `flipped` to get the logical value. Maintain a separate `setCount` that adjusts on flip: `setCount = size - setCount`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Bitset {
    vector<long long> words;
    int size;
    bool flipped;
    int setCount;

public:
    Bitset(int size) : size(size), words((size + 63) / 64, 0LL), flipped(false), setCount(0) {}

    void fix(int idx) {
        int wi = idx / 64, bi = idx % 64;
        bool isSet = ((words[wi] >> bi) & 1) == 1;
        // Logical value = isSet XOR flipped
        if (!(isSet ^ flipped)) { // currently logically 0
            if (!flipped) {
                words[wi] |= (1LL << bi);  // set physical bit
            } else {
                words[wi] &= ~(1LL << bi); // physical 1 means logical 0 when flipped; clear it
            }
            setCount++;
        }
    }

    void unfix(int idx) {
        int wi = idx / 64, bi = idx % 64;
        bool isSet = ((words[wi] >> bi) & 1) == 1;
        if (isSet ^ flipped) { // currently logically 1
            if (!flipped) {
                words[wi] &= ~(1LL << bi);
            } else {
                words[wi] |= (1LL << bi);
            }
            setCount--;
        }
    }

    void flip() {
        flipped = !flipped;
        setCount = size - setCount;
    }

    bool all()  { return setCount == size; }
    bool one()  { return setCount > 0; }
    int count() { return setCount; }

    string toString() {
        string result;
        result.reserve(size);
        for (int i = 0; i < size; i++) {
            int wi = i / 64, bi = i % 64;
            int physical = (int)((words[wi] >> bi) & 1);
            result += (char)('0' + (physical ^ (flipped ? 1 : 0)));
        }
        return result;
    }
};
```

---

## Simpler Implementation (using vector\<bool\>)

If a simpler approach is acceptable, use C++'s `vector<bool>` (which is bit-packed internally):

```cpp
#include <bits/stdc++.h>
using namespace std;

class Bitset {
    vector<bool> bs;
    vector<bool> flippedBs;
    int sz;

    void xorBitsets(vector<bool>& a, const vector<bool>& b) {
        for (int i = 0; i < (int)a.size(); i++) a[i] = a[i] ^ b[i];
    }

public:
    Bitset(int size) : sz(size), bs(size, false), flippedBs(size, true) {}

    void fix(int idx)   { bs[idx] = true; flippedBs[idx] = false; }
    void unfix(int idx) { bs[idx] = false; flippedBs[idx] = true; }
    void flip() {
        vector<bool> tmp = bs;
        xorBitsets(bs, flippedBs);
        xorBitsets(flippedBs, tmp);
    }
    bool all() {
        for (int i = 0; i < sz; i++) if (!bs[i]) return false;
        return true;
    }
    bool one() {
        for (int i = 0; i < sz; i++) if (bs[i]) return true;
        return false;
    }
    int count() {
        int cnt = 0;
        for (int i = 0; i < sz; i++) if (bs[i]) cnt++;
        return cnt;
    }
    string toString() {
        string result;
        for (int i = 0; i < sz; i++) result += (bs[i] ? '1' : '0');
        return result;
    }
};
```

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Storage | `vector<long long>` or `vector<bool>` | Compact 64x space saving vs bool[] |
| `flip()` O(1) | Lazy `flipped` flag | Avoids O(n) bit inversion |
| `setCount` maintenance | Updated on fix/unfix/flip | O(1) `all()`/`one()`/`count()` |
| `toString()` | O(n) always | Must read each bit |

---

## Complexity

| Operation | Time | Space |
|-----------|------|-------|
| `fix`, `unfix` | O(1) | — |
| `flip` | O(1) | — |
| `all`, `one`, `count` | O(1) | — |
| `toString` | O(n) | O(n) |
| Total space | — | O(n/64) = O(n) |

---

## Related Files

- [XOR Linked List](./XOR%20Linked%20List.md)
- [Bit Basics](../Patterns/Bit%20Basics.md)

---

**Back:** [Bit Manipulation README](../README.md)

> **Last Updated:** 2026-06-26
