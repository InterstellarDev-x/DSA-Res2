# Design: Dynamic Array (ArrayList)

> **Topic:** [Arrays](../README.md) · **Category:** Design Data Structure
> **Difficulty:** Medium · **Tags:** `design` `amortized` `resizing`

---

## Table of Contents

1. [Problem Statement](#problem-statement)
2. [Interview Expectations](#interview-expectations)
3. [Brute Force Approach](#brute-force-approach)
4. [Optimal Approach](#optimal-approach)
5. [Java Implementation](#java-implementation)
6. [Complexity Analysis](#complexity-analysis)
7. [Edge Cases](#edge-cases)
8. [Similar Problems](#similar-problems)
9. [Follow-up Questions](#follow-up-questions)
10. [Company Tags](#company-tags)

---

## Problem Statement

Design a dynamic array (similar to Java's `ArrayList`) that supports:

- `void add(int val)` — append element; resize if needed
- `int get(int index)` — return element at index
- `void set(int index, int val)` — update element at index
- `void remove(int index)` — remove element, shift left
- `int size()` — return current number of elements

The internal array should double in capacity when full and halve when less than 25% full (to avoid thrashing).

---

## Interview Expectations

| Expectation | Detail |
|-------------|--------|
| Core concept | Amortized O(1) append via doubling strategy |
| Key insight | Why doubling (not +1) gives O(1) amortized cost |
| Must explain | Amortized analysis — total cost over n operations = O(n) |
| Bonus | Shrinking policy and why threshold = 25% (not 50%) |

---

## Brute Force Approach

Create a new array of size `n+1` every time an element is added.

- **Time:** O(n) per `add` (copy everything)
- **Space:** O(n²) total over n insertions

---

## Optimal Approach

**Doubling strategy:**
- Start with capacity `c = 1`
- When `size == capacity`: allocate new array of size `2 * capacity`, copy all elements
- When `size < capacity / 4`: shrink to `capacity / 2`

**Why doubling works (amortized analysis):**
After `n` insertions into a fresh array, total copy work = 1 + 2 + 4 + ... + n = **2n** = O(n).
So amortized cost per insertion = O(1).

---

## Java Implementation

```java
public class DynamicArray {
    private int[] data;
    private int size;
    private int capacity;

    public DynamicArray() {
        capacity = 2;
        data = new int[capacity];
        size = 0;
    }

    public void add(int val) {
        if (size == capacity) resize(capacity * 2);
        data[size++] = val;
    }

    public int get(int index) {
        checkBounds(index);
        return data[index];
    }

    public void set(int index, int val) {
        checkBounds(index);
        data[index] = val;
    }

    public void remove(int index) {
        checkBounds(index);
        System.arraycopy(data, index + 1, data, index, size - index - 1);
        size--;
        // Shrink if using less than 25% capacity (avoid thrash: don't shrink at 50%)
        if (size > 0 && size == capacity / 4) resize(capacity / 2);
    }

    public int size() { return size; }

    private void resize(int newCapacity) {
        int[] newData = new int[newCapacity];
        System.arraycopy(data, 0, newData, 0, size);
        data = newData;
        capacity = newCapacity;
    }

    private void checkBounds(int index) {
        if (index < 0 || index >= size)
            throw new IndexOutOfBoundsException("Index: " + index + ", Size: " + size);
    }
}
```

---

## Complexity Analysis

| Operation | Time (amortized) | Space |
|-----------|-----------------|-------|
| `add` | O(1) amortized, O(n) worst | O(1) amortized |
| `get` | O(1) | O(1) |
| `set` | O(1) | O(1) |
| `remove` (middle) | O(n) | O(1) |
| `remove` (end) | O(1) | O(1) |
| Overall space | — | O(n) |

---

## Edge Cases

- `get(-1)` or `get(size)` → must throw `IndexOutOfBoundsException`
- Remove last element → no shift needed, just `size--`
- Repeated remove → verify shrinking doesn't go below capacity 1
- Add after remove → reuse existing space correctly
- Capacity can never go below 1 (guard the shrink)

---

## Similar Problems

| Problem | Link |
|---------|------|
| [Design Circular Queue](./Circular%20Queue.md) | Fixed-size circular buffer |
| [Design HashMap](./HashMap.md) | Dynamic resizing with load factor |
| [Design Stack using Array](../../7.Stacks_and_Queues/Design%20Data%20Structure%20Problems/Stack.md) | Stack over dynamic array |

---

## Follow-up Questions

1. **Why shrink at 25% and not 50%?** — 50% causes thrashing: add one element (resize up), remove one (resize down), repeat. 25% gives a hysteresis buffer.
2. **How does Java's `ArrayList` differ?** — It grows by 50% (`capacity + capacity/2`), never shrinks automatically.
3. **Thread safety?** — Need `synchronized` or `CopyOnWriteArrayList` for concurrent access.
4. **Generic version?** — Replace `int[]` with `Object[]`, use unchecked casts, store type via `Class<T>`.
5. **Memory fragmentation?** — Each resize allocates a new contiguous block; old block is GC'd.

---

## Company Tags

`Amazon` `Google` `Microsoft` `Adobe`

---

## Navigation

| Related |
|---------|
| [Arrays README](../README.md) |
| [Interview Tips — Coding Tips](../Interview%20Tips/Coding%20Tips.md) |

> **Last Updated:** 2026-06-26
