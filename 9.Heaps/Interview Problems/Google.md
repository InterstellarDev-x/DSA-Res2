# Google — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Google

---

## Problem 1: Find Median from Data Stream — Deep Dive

**LC 295** · Hard

```java
class MedianFinder {
    private PriorityQueue<Integer> lo = new PriorityQueue<>(Comparator.reverseOrder());
    private PriorityQueue<Integer> hi = new PriorityQueue<>();

    public void addNum(int num) {
        if (lo.isEmpty() || num <= lo.peek()) lo.offer(num);
        else hi.offer(num);
        if (lo.size() > hi.size() + 1) hi.offer(lo.poll());
        else if (hi.size() > lo.size()) lo.offer(hi.poll());
    }

    public double findMedian() {
        return lo.size() > hi.size() ? lo.peek() : (lo.peek() + hi.peek()) / 2.0;
    }
}
```

**Q: What if the stream only contains integers in [0, 100]?**
A: Use a frequency array of size 101. Track total count. For `findMedian`, walk the array finding the position(s) of the median. O(1) addNum, O(100) = O(1) findMedian, but truly O(1) space (not O(n)).

**Q: What if 99% of numbers are in [0, 100] with occasional outliers?**
A: Hybrid approach: bucket array for [0..100], two small heaps for outliers. Median calculation walks the appropriate structure based on total count.

**Q: What is the invariant that guarantees correctness?**
A: Two invariants: (1) Size: `|lo.size - hi.size| ≤ 1`. (2) Order: `lo.peek() ≤ hi.peek()`. If both hold, the median is either `lo.peek()` (odd total) or `(lo.peek() + hi.peek()) / 2` (even total).

---

## Problem 2: IPO — Deep Dive

**LC 502** · Hard

```java
public int findMaximizedCapital(int k, int w, int[] profits, int[] capital) {
    int n = profits.length;
    PriorityQueue<int[]> locked    = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    PriorityQueue<int[]> available = new PriorityQueue<>((a, b) -> b[1] - a[1]);

    for (int i = 0; i < n; i++) locked.offer(new int[]{capital[i], profits[i]});

    for (int i = 0; i < k; i++) {
        while (!locked.isEmpty() && locked.peek()[0] <= w)
            available.offer(locked.poll());
        if (available.isEmpty()) break;
        w += available.poll()[1];
    }
    return w;
}
```

**Q: Prove the greedy choice is optimal.**
A: At each step, we can afford any project with `capital ≤ w`. Among these, picking the one with the highest profit maximizes our capital gain — any other choice leads to equal or lower capital for all future steps. This is a classic exchange argument: swapping any non-maximum profit project with the maximum always improves or maintains the result.

**Q: Google follow-up — what if we have a budget (total capital we can spend)?**
A: Modify: track total spent capital separately, and `available` heap now also checks if we can afford the cost (if projects have both profit AND cost).

**Q: Google follow-up — what if k is very large?**
A: Once `available` is empty (can't afford anything more), break early. With `k = n`, we might complete all projects.

---

## Problem 3: Smallest Range from K Lists — Deep Dive

**LC 632** · Hard

```java
public int[] smallestRange(List<List<Integer>> nums) {
    PriorityQueue<int[]> heap = new PriorityQueue<>(Comparator.comparingInt(a -> a[0]));
    int maxVal = Integer.MIN_VALUE;

    for (int i = 0; i < nums.size(); i++) {
        heap.offer(new int[]{nums.get(i).get(0), i, 0});
        maxVal = Math.max(maxVal, nums.get(i).get(0));
    }

    int[] result = {heap.peek()[0], maxVal};

    while (true) {
        int[] curr = heap.poll();
        int listIdx = curr[1], elemIdx = curr[2];
        if (elemIdx + 1 == nums.get(listIdx).size()) break;

        int next = nums.get(listIdx).get(elemIdx + 1);
        heap.offer(new int[]{next, listIdx, elemIdx + 1});
        maxVal = Math.max(maxVal, next);
        if (maxVal - heap.peek()[0] < result[1] - result[0]) {
            result[0] = heap.peek()[0]; result[1] = maxVal;
        }
    }
    return result;
}
```

**Q: Why does `maxVal` only increase?**
A: We only add elements from lists, and each list is sorted ascending. When we advance to the next element of a list, the new value is ≥ the old value. So `maxVal` is non-decreasing. The minimum of the heap (= `heap.peek()[0]`) is the range's lower bound; we minimize the range by advancing the minimum.

**Q: Why stop when a list is exhausted?**
A: We need exactly one element from each list. If list `i` is exhausted, the range's minimum is now determined by the second-smallest across lists. We can't include list `i` anymore — there's no valid range.

---

## Google L3/L4/L5 Problem Mapping

| Level | Typical Problem |
|-------|----------------|
| L3 | Kth Largest, Top K Frequent, K Closest |
| L4 | Find Median, IPO, K Pairs |
| L5+ | Sliding Window Median, Smallest Range, Design Twitter |

---

## Related Files

- [Google OA](../OA-Qns/Google.md)
- [Amazon Interview Problems](./Amazon.md)
- [Two Heaps Pattern](../Patterns/Two%20Heaps.md)
- [K-Way Merge Pattern](../Patterns/K%20Way%20Merge.md)

> **Last Updated:** 2026-06-26
