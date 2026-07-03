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

```cpp
#include <bits/stdc++.h>
using namespace std;

class MedianFinder {
    // lo: max-heap, holds lower half
    priority_queue<int> lo;
    // hi: min-heap, holds upper half
    priority_queue<int, vector<int>, greater<int>> hi;

public:
    void addNum(int num) {
        // 1. Route to correct half
        if (lo.empty() || num <= lo.top()) lo.push(num);
        else hi.push(num);

        // 2. Rebalance: lo can have at most 1 more than hi
        if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
        else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }

    double findMedian() {
        return lo.size() > hi.size()
            ? lo.top()
            : (lo.top() + hi.top()) / 2.0;
    }
};
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

```cpp
double findMedian() {
    if (lo.size() > hi.size()) return lo.top();
    return lo.top() / 2.0 + hi.top() / 2.0;  // avoids overflow vs (a+b)/2
}
```

---

## Follow-up 2: Find Median of Integer Stream with Constraint

**"99% of all numbers are in range [0, 100]"** — Use a bucket array for common values, heap for outliers.

```cpp
// Optimization: O(1) amortized via bucket counts for [0..100]
// Heap only for values outside that range
// findMedian: walk bucket array from 0, counting until median position
```

---

## Follow-up 3: Thread Safety

```cpp
#include <bits/stdc++.h>
#include <shared_mutex>
using namespace std;

class ThreadSafeMedianFinder {
    priority_queue<int> lo;
    priority_queue<int, vector<int>, greater<int>> hi;
    mutable shared_mutex mtx;

public:
    void addNum(int num) {
        unique_lock<shared_mutex> lk(mtx);
        if (lo.empty() || num <= lo.top()) lo.push(num);
        else hi.push(num);
        if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
        else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }

    double findMedian() const {
        shared_lock<shared_mutex> lk(mtx);
        return lo.size() > hi.size() ? lo.top() : (lo.top() + hi.top()) / 2.0;
    }
};
```

**`shared_mutex`:** Multiple concurrent `findMedian` readers allowed (`shared_lock`); only one `addNum` writer at a time (`unique_lock`).

---

## Key Design Decisions

| Decision | Choice | Reason |
|----------|--------|--------|
| Which heap holds extra element? | `lo` (max-heap) | `lo.top()` is the median for odd count |
| How to route new element? | Route to `lo` first if ≤ lo.top(), else to `hi` | Maintains order property |
| Rebalance direction? | Balance so `lo.size ≤ hi.size + 1` | `lo` is allowed 1 extra |
| Integer arithmetic in findMedian? | Use `/2.0` division | Avoids integer truncation |

---

## Related Files

- [Two Heaps Pattern](../Patterns/Two%20Heaps.md)
- [Twitter Feed Design](./Twitter%20Feed.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
