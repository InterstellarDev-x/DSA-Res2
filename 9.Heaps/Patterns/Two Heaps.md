# Two Heaps

> **Topic:** [Heaps](../README.md) · **Pattern 4 of 5**
> **Problems:** Find Median from Data Stream · Sliding Window Median · IPO

---

## Core Concept

**Two-heap pattern** maintains two heaps to efficiently track the median or balance elements:
- **Max-heap (`lo`):** lower half of all elements; root = largest in lower half
- **Min-heap (`hi`):** upper half of all elements; root = smallest in upper half

**Invariants to maintain after every operation:**
1. **Size balance:** `lo.size() == hi.size()` OR `lo.size() == hi.size() + 1` (lo can have one extra)
2. **Order property:** `lo.peek() ≤ hi.peek()` (every lower-half element ≤ every upper-half element)

**Median:**
- If `lo.size() == hi.size()`: median = `(lo.peek() + hi.peek()) / 2.0`
- If `lo.size() > hi.size()`: median = `lo.peek()`

---

## Problem 1: Find Median from Data Stream — LC 295

```java
class MedianFinder {
    private PriorityQueue<Integer> lo = new PriorityQueue<>(Comparator.reverseOrder()); // max-heap
    private PriorityQueue<Integer> hi = new PriorityQueue<>();                           // min-heap

    public void addNum(int num) {
        // Step 1: Route to correct heap
        if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
        else hi.offer(num);

        // Step 2: Rebalance so |lo.size() - hi.size()| <= 1
        if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
        else if (hi.size() > lo.size()) lo.offer(hi.poll());
    }

    public double findMedian() {
        if (lo.size() == hi.size()) return (lo.peek() + hi.peek()) / 2.0;
        return lo.peek();   // lo always has the extra element
    }
}
```

**Trace for addNum sequence [5, 3, 8, 1]:**
```
addNum(5): lo=[5], hi=[]
addNum(3): 3<=5 → lo=[5,3]. lo.size=2 > hi.size+1=1 → move to hi. lo=[5], hi=[3]
           Wait — rebalance: lo.size(2) > hi.size(0)+1=1 → hi.offer(lo.poll()=5) → lo=[3], hi=[5]
           Actually: lo.size=2 > hi.size+1=1, rebalance → hi.offer(lo.poll()) → lo=[3], hi=[5]
           Hmm, 3 should be in lo (lower half). Let me retrace:
           addNum(3): 3 <= lo.peek()=5 → lo.offer(3) → lo=[5,3]. Rebalance: lo.size=2 > hi.size(0)+1=1 → hi.offer(lo.poll()=5) → lo=[3], hi=[5]
           findMedian: lo.size=hi.size=1 → (3+5)/2=4
addNum(8): 8 > lo.peek()=3 → hi.offer(8) → hi=[5,8]. Rebalance: hi.size=2 > lo.size=1 → lo.offer(hi.poll()=5) → lo=[5,3], hi=[8]
           findMedian: lo.size=2 > hi.size=1 → lo.peek()=5
addNum(1): 1 <= lo.peek()=5 → lo.offer(1) → lo=[5,3,1]. Rebalance: lo.size=3 > hi.size+1=2 → hi.offer(lo.poll()=5) → lo=[3,1], hi=[5,8]
           findMedian: lo.size=hi.size=2 → (3+5)/2=4
```

**Complexity:** O(log n) addNum, O(1) findMedian, O(n) space

---

## Problem 2: Sliding Window Median — LC 480

Find the median of each window of size k as it slides.

**Challenge:** Standard two-heap doesn't support removal of arbitrary elements. Solution: lazy deletion using a `toRemove` counter map.

```java
public double[] medianSlidingWindow(int[] nums, int k) {
    // TreeMap acts as ordered multiset — supports O(log n) add/remove
    TreeMap<Integer, Integer> lo = new TreeMap<>(Comparator.reverseOrder()); // lower half
    TreeMap<Integer, Integer> hi = new TreeMap<>();                           // upper half
    int loSize = 0, hiSize = 0;

    double[] result = new double[nums.length - k + 1];

    for (int i = 0; i < nums.length; i++) {
        // Add incoming element
        if (lo.isEmpty() || nums[i] <= lo.firstKey()) {
            lo.merge(nums[i], 1, Integer::sum); loSize++;
        } else {
            hi.merge(nums[i], 1, Integer::sum); hiSize++;
        }

        // Rebalance
        if (loSize > hiSize + 1) {
            int top = lo.firstKey();
            hi.merge(top, 1, Integer::sum); hiSize++;
            lo.merge(top, -1, Integer::sum);
            if (lo.get(top) == 0) lo.remove(top);
            loSize--;
        } else if (hiSize > loSize) {
            int top = hi.firstKey();
            lo.merge(top, 1, Integer::sum); loSize++;
            hi.merge(top, -1, Integer::sum);
            if (hi.get(top) == 0) hi.remove(top);
            hiSize--;
        }

        // Window is full starting at i == k-1
        if (i >= k - 1) {
            result[i - k + 1] = (loSize == hiSize)
                ? ((double) lo.firstKey() + hi.firstKey()) / 2.0
                : lo.firstKey();

            // Remove outgoing element (leftmost of window)
            int out = nums[i - k + 1];
            if (lo.containsKey(out)) {
                lo.merge(out, -1, Integer::sum);
                if (lo.get(out) == 0) lo.remove(out);
                loSize--;
            } else {
                hi.merge(out, -1, Integer::sum);
                if (hi.get(out) == 0) hi.remove(out);
                hiSize--;
            }

            // Rebalance after removal
            if (loSize > hiSize + 1) {
                int top = lo.firstKey();
                hi.merge(top, 1, Integer::sum); hiSize++;
                lo.merge(top, -1, Integer::sum);
                if (lo.get(top) == 0) lo.remove(top);
                loSize--;
            } else if (hiSize > loSize) {
                int top = hi.firstKey();
                lo.merge(top, 1, Integer::sum); loSize++;
                hi.merge(top, -1, Integer::sum);
                if (hi.get(top) == 0) hi.remove(top);
                hiSize--;
            }
        }
    }
    return result;
}
```

**Why TreeMap instead of PriorityQueue?** `PriorityQueue.remove(x)` is O(n). `TreeMap.remove(x)` is O(log n). For a sliding window, O(log n) removal is essential.

---

## Problem 3: IPO — LC 502

Maximize capital after k projects. Each project has profit and capital requirement.

```java
public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
    int n = profits.length;
    // Min-heap by capital requirement — projects we can't afford yet
    PriorityQueue<int[]> locked = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    // Max-heap by profit — projects we can afford; pick most profitable
    PriorityQueue<int[]> available = new PriorityQueue<>((a, b) -> b[1] - a[1]);

    for (int i = 0; i < n; i++) locked.offer(new int[]{capital[i], profits[i]});

    for (int i = 0; i < k; i++) {
        // Unlock all projects we can now afford
        while (!locked.isEmpty() && locked.peek()[0] <= w) {
            available.offer(locked.poll());
        }
        if (available.isEmpty()) break;   // no affordable projects
        w += available.poll()[1];          // pick most profitable
    }
    return w;
}
```

**Two-heap insight:** One min-heap sorted by capital (locked projects) and one max-heap sorted by profit (available projects). This models a "greedy unlock + greedy pick" strategy.

**Complexity:** O(n log n + k log n) time, O(n) space

---

## Two Heaps — Invariant Summary

| Problem | lo (max-heap) | hi (min-heap) | Invariant |
|---------|--------------|--------------|-----------|
| Median Stream | lower half | upper half | \|lo\|-\|hi\| ≤ 1; lo.peek ≤ hi.peek |
| Sliding Window Median | lower half of window | upper half of window | Same + need removal |
| IPO | available projects (by profit) | locked projects (by capital) | Different semantic — unlock to available on each step |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [MedianFinder Design](../Design%20Data%20Structure%20Problems/Median%20Finder.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
