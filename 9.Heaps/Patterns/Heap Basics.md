# Heap Basics

> **Topic:** [Heaps](../README.md) · **Pattern 1 of 5**
> **Problems:** Last Stone Weight · heap fundamentals

---

## Core Concept

A **heap** is a complete binary tree satisfying the heap property:
- **Min-heap:** parent ≤ children → root = minimum element
- **Max-heap:** parent ≥ children → root = maximum element

Java's `PriorityQueue` is a **min-heap** by default. To get a max-heap, reverse the comparator.

---

## Java PriorityQueue API

```java
// Min-heap (default)
PriorityQueue<Integer> minHeap = new PriorityQueue<>();

// Max-heap
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
// equivalent: new PriorityQueue<>((a, b) -> b - a)   ← UNSAFE (subtraction overflow)
// SAFE:       new PriorityQueue<>((a, b) -> Integer.compare(b, a))

// Custom comparator — sort by absolute value
PriorityQueue<Integer> pq = new PriorityQueue<>(Comparator.comparingInt(Math::abs));

// Operations
pq.offer(x);      // insert — O(log n)
pq.poll();        // remove top (min or max) — O(log n); returns null if empty
pq.peek();        // view top without removing — O(1); returns null if empty
pq.size();        // O(1)
pq.isEmpty();     // O(1)
pq.contains(x);  // O(n) — linear scan; avoid in time-critical code
pq.remove(x);    // O(n) — linear search + O(log n) heapify; expensive
```

**Critical:** Never use subtraction `b - a` in comparators — it overflows for extreme values like `Integer.MIN_VALUE`. Always use `Integer.compare(b, a)`.

---

## Heap Internals

| Operation | Min-Heap | Max-Heap |
|-----------|---------|---------|
| Insert | O(log n) — bubble up | O(log n) — bubble up |
| Delete top | O(log n) — bubble down | O(log n) — bubble down |
| Peek top | O(1) | O(1) |
| Build heap | O(n) — heapify all | O(n) — heapify all |
| Find kth | O(k log n) — poll k times | O(k log n) |

**Build heap is O(n), not O(n log n):** Bottom-up heapification; nodes at lower levels have minimal work; by amortized analysis total is O(n).

---

## Problem 1: Last Stone Weight — LC 1046

Repeatedly smash the two heaviest stones. If equal, both destroyed; otherwise `|y - x|` remains.

```java
public int lastStoneWeight(int[] stones) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    for (int s : stones) maxHeap.offer(s);

    while (maxHeap.size() > 1) {
        int y = maxHeap.poll();   // largest
        int x = maxHeap.poll();   // second largest
        if (y != x) maxHeap.offer(y - x);   // remainder
    }

    return maxHeap.isEmpty() ? 0 : maxHeap.poll();
}
```

**Complexity:** O(n log n) — n stones, each operation O(log n)

**Edge cases:**
- All stones equal → all destroyed → return 0 (heap becomes empty)
- Single stone → while loop never runs → return that stone

---

## Min-Heap vs Max-Heap Selection

| Goal | Heap Type | Why |
|------|----------|-----|
| Always access the minimum | Min-heap | Root = min |
| Always access the maximum | Max-heap | Root = max |
| Find kth largest | Min-heap of size k | Root = kth largest; pop when size > k |
| Find kth smallest | Max-heap of size k | Root = kth smallest; pop when size > k |
| Median (lower half) | Max-heap | Root = middle of lower half |
| Median (upper half) | Min-heap | Root = middle of upper half |

---

## Heap with Custom Objects

```java
// Sort by distance, then by index for ties
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> {
    if (a[0] != b[0]) return Integer.compare(a[0], b[0]);  // primary: distance
    return Integer.compare(a[1], b[1]);                     // tie-break: index
});
pq.offer(new int[]{distance, index});
```

**For Top K Frequent Words** (string comparison + frequency):
```java
PriorityQueue<Map.Entry<String, Integer>> pq = new PriorityQueue<>((a, b) -> {
    if (!a.getValue().equals(b.getValue()))
        return a.getValue() - b.getValue();  // fewer freq first (min-heap of freq)
    return b.getKey().compareTo(a.getKey()); // more lexicographic first for same freq
});
```

---

## Heapify — Building from Array

```java
// Java: PriorityQueue constructor accepts Collection
int[] arr = {3, 1, 4, 1, 5, 9, 2, 6};
PriorityQueue<Integer> pq = new PriorityQueue<>();
for (int x : arr) pq.offer(x);   // O(n log n) via repeated inserts

// For true O(n) heapify, use Arrays-based approach (not directly available in Java PQ)
// In practice, the loop above is fine for interviews
```

---

## Related Files

- [Top K Elements](./Top%20K%20Elements.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Two Heaps](./Two%20Heaps.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
