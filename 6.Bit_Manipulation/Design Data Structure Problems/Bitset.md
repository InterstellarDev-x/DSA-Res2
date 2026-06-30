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

Use a `long[]` array — each `long` holds 64 bits. Key challenge: `flip()` must be O(1), not O(n).

**Lazy flip:** Maintain a `flipped` boolean. Flipping twice = no-op. On any read, XOR the actual bit with `flipped` to get the logical value. Maintain a separate `setCount` that adjusts on flip: `setCount = size - setCount`.

```java
class Bitset {
    private final long[] words;
    private final int size;
    private boolean flipped;
    private int setCount;

    public Bitset(int size) {
        this.size = size;
        this.words = new long[(size + 63) / 64]; // ceiling division
        this.flipped = false;
        this.setCount = 0;
    }

    public void fix(int idx) {
        int wi = idx / 64, bi = idx % 64;
        boolean isSet = ((words[wi] >> bi) & 1) == 1;
        // Logical value = isSet XOR flipped
        if (!(isSet ^ flipped)) { // currently logically 0
            if (!flipped) {
                words[wi] |= (1L << bi);  // set physical bit
            } else {
                words[wi] &= ~(1L << bi); // physical 1 means logical 0 when flipped; clear it
            }
            setCount++;
        }
    }

    public void unfix(int idx) {
        int wi = idx / 64, bi = idx % 64;
        boolean isSet = ((words[wi] >> bi) & 1) == 1;
        if (isSet ^ flipped) { // currently logically 1
            if (!flipped) {
                words[wi] &= ~(1L << bi);
            } else {
                words[wi] |= (1L << bi);
            }
            setCount--;
        }
    }

    public void flip() {
        flipped = !flipped;
        setCount = size - setCount;
    }

    public boolean all()  { return setCount == size; }
    public boolean one()  { return setCount > 0; }
    public int count()    { return setCount; }

    public String toString() {
        StringBuilder sb = new StringBuilder(size);
        for (int i = 0; i < size; i++) {
            int wi = i / 64, bi = i % 64;
            int physical = (int)((words[wi] >> bi) & 1);
            sb.append((char)('0' + (physical ^ (flipped ? 1 : 0))));
        }
        return sb.toString();
    }
}
```

---

## Simpler Implementation (Java BitSet wrapper)

If the interviewer allows Java's built-in `java.util.BitSet`:

```java
class Bitset {
    private final java.util.BitSet bs;
    private final java.util.BitSet flippedBs;
    private final int size;

    public Bitset(int size) {
        this.size = size;
        bs = new java.util.BitSet(size);
        flippedBs = new java.util.BitSet(size);
        flippedBs.set(0, size); // flippedBs = complement
    }

    public void fix(int idx)   { bs.set(idx); flippedBs.clear(idx); }
    public void unfix(int idx) { bs.clear(idx); flippedBs.set(idx); }
    public void flip()         { java.util.BitSet tmp = (java.util.BitSet) bs.clone(); bs.xor(flippedBs); flippedBs.xor(tmp); }
    public boolean all()       { return bs.cardinality() == size; }
    public boolean one()       { return !bs.isEmpty(); }
    public int count()         { return bs.cardinality(); }
    public String toString()   {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < size; i++) sb.append(bs.get(i) ? '1' : '0');
        return sb.toString();
    }
}
```

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Storage | `long[]` or `java.util.BitSet` | Compact 64x space saving vs boolean[] |
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
