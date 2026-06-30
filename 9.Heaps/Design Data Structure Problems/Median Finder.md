# MedianFinder

> **Topic:** [Heaps](../README.md) · **Design 1 of 2**
> **Problems:** Find Median from Data Stream (LC 295)

---

## Problem Statement

Design a data structure that supports:
- `addNum(num)` — add integer to the data structure
- `findMedian()` — return the median of all added numbers

All operations must be efficient.

---

## Solution: Two Heaps

```java
class MedianFinder {
    // lo: max-heap, holds lower half
    private PriorityQueue<Integer> lo = new PriorityQueue<>(Comparator.reverseOrder());
    // hi: min-heap, holds upper half
    private PriorityQueue<Integer> hi = new PriorityQueue<>();

    public void addNum(int num) {
        // 1. Route to correct half
        if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
        else hi.offer(num);

        // 2. Rebalance: lo can have at most 1 more than hi
        if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
        else if (hi.size() > lo.size()) lo.offer(hi.poll());
    }

    public double findMedian() {
        return lo.size() > hi.size()
            ? lo.peek()
            : (lo.peek() + hi.peek()) / 2.0;
    }
}
```

**Complexity:** `addNum` O(log n), `findMedian` O(1), space O(n)

---

## Dry Run: [1, 2, 3, 4, 5]

```
addNum(1): lo=[1], hi=[]              → median = 1.0
addNum(2): 2>1 → hi=[2]. hi.size>lo.size → lo.offer(hi.poll()=2) → lo=[2,1], hi=[]
           Wait: 2 > lo.peek()=1 → hi.offer(2) → lo=[1], hi=[2]. lo.size=hi.size=1 → OK
           findMedian: lo.size=hi.size=1 → (1+2)/2=1.5
addNum(3): 3 > lo.peek()=1 → hi.offer(3) → lo=[1], hi=[2,3]. hi.size>lo.size → lo.offer(hi.poll()=2) → lo=[2,1], hi=[3]
           findMedian: lo.size=2>hi.size=1 → lo.peek()=2
addNum(4): 4>2 → hi.offer(4) → lo=[2,1], hi=[3,4]. lo.size=hi.size=2 → OK
           findMedian: (2+3)/2=2.5
addNum(5): 5>2 → hi.offer(5) → lo=[2,1], hi=[3,4,5]. hi.size=3>lo.size=2 → lo.offer(hi.poll()=3) → lo=[3,2,1], hi=[4,5]
           findMedian: lo.size=3>hi.size=2 → lo.peek()=3
```

---

## Follow-up 1: Handle Large Even Counts — Integer Overflow in Median

```java
public double findMedian() {
    if (lo.size() > hi.size()) return lo.peek();
    return lo.peek() / 2.0 + hi.peek() / 2.0;  // avoids overflow vs (a+b)/2
}
```

---

## Follow-up 2: Find Median of Integer Stream with Constraint

**"99% of all numbers are in range [0, 100]"** — Use a bucket array for common values, heap for outliers.

```java
// Optimization: O(1) amortized via bucket counts for [0..100]
// Heap only for values outside that range
// findMedian: walk bucket array from 0, counting until median position
```

---

## Follow-up 3: Thread Safety

```java
class ThreadSafeMedianFinder {
    private final PriorityQueue<Integer> lo = new PriorityQueue<>(Comparator.reverseOrder());
    private final PriorityQueue<Integer> hi = new PriorityQueue<>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();

    public void addNum(int num) {
        lock.writeLock().lock();
        try {
            if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
            else hi.offer(num);
            if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
            else if (hi.size() > lo.size()) lo.offer(hi.poll());
        } finally { lock.writeLock().unlock(); }
    }

    public double findMedian() {
        lock.readLock().lock();
        try {
            return lo.size() > hi.size() ? lo.peek() : (lo.peek() + hi.peek()) / 2.0;
        } finally { lock.readLock().unlock(); }
    }
}
```

**ReadWriteLock:** Multiple concurrent `findMedian` readers allowed; only one `addNum` writer at a time.

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Which heap holds extra element? | `lo` (max-heap) | `lo.peek()` is the median for odd count |
| How to route new element? | Route to `lo` first if ≤ lo.peek, else to `hi` | Maintains order property |
| Rebalance direction? | Balance so `lo.size ≤ hi.size + 1` | `lo` is allowed 1 extra |
| Integer arithmetic in findMedian? | Use `/2.0` division | Avoids integer truncation |

---

## Related Files

- [Two Heaps Pattern](../Patterns/Two%20Heaps.md)
- [Twitter Feed Design](./Twitter%20Feed.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
