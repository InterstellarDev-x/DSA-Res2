# Google — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Google

---

## Problem 1: Find Median from Data Stream — Deep Dive

**LC 295** · Hard

```cpp
#include <bits/stdc++.h>
using namespace std;

class MedianFinder {
    priority_queue<int> lo; // max-heap
    priority_queue<int, vector<int>, greater<int>> hi; // min-heap

public:
    void addNum(int num) {
        if (lo.empty() || num <= lo.top()) lo.push(num);
        else hi.push(num);
        if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
        else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }

    double findMedian() {
        return lo.size() > hi.size() ? lo.top() : (lo.top() + hi.top()) / 2.0;
    }
};
```

**Q: What if the stream only contains integers in [0, 100]?**
A: Use a frequency array of size 101. Track total count. For `findMedian`, walk the array finding the position(s) of the median. O(1) addNum, O(100) = O(1) findMedian, but truly O(1) space (not O(n)).

**Q: What if 99% of numbers are in [0, 100] with occasional outliers?**
A: Hybrid approach: bucket array for [0..100], two small heaps for outliers. Median calculation walks the appropriate structure based on total count.

**Q: What is the invariant that guarantees correctness?**
A: Two invariants: (1) Size: `|lo.size - hi.size| ≤ 1`. (2) Order: `lo.top() ≤ hi.top()`. If both hold, the median is either `lo.top()` (odd total) or `(lo.top() + hi.top()) / 2` (even total).

---

## Problem 2: IPO — Deep Dive

**LC 502** · Hard

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximizedCapital(int k, int w, vector<int>& profits, vector<int>& capital) {
    int n = profits.size();
    // min-heap by capital: {capital, profit}
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> locked;
    // max-heap by profit: {profit, capital}
    priority_queue<pair<int,int>> available;

    for (int i = 0; i < n; i++) locked.push({capital[i], profits[i]});

    for (int i = 0; i < k; i++) {
        while (!locked.empty() && locked.top().first <= w) {
            auto [cap, prof] = locked.top(); locked.pop();
            available.push({prof, cap});
        }
        if (available.empty()) break;
        w += available.top().first;
        available.pop();
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

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> smallestRange(vector<vector<int>>& nums) {
    // min-heap: {value, listIdx, elemIdx}
    using T = tuple<int,int,int>;
    priority_queue<T, vector<T>, greater<T>> heap;
    int maxVal = INT_MIN;

    for (int i = 0; i < (int)nums.size(); i++) {
        heap.push({nums[i][0], i, 0});
        maxVal = max(maxVal, nums[i][0]);
    }

    vector<int> result = {get<0>(heap.top()), maxVal};

    while (true) {
        auto [val, listIdx, elemIdx] = heap.top(); heap.pop();
        if (elemIdx + 1 == (int)nums[listIdx].size()) break;

        int next = nums[listIdx][elemIdx + 1];
        heap.push({next, listIdx, elemIdx + 1});
        maxVal = max(maxVal, next);
        if (maxVal - get<0>(heap.top()) < result[1] - result[0]) {
            result[0] = get<0>(heap.top());
            result[1] = maxVal;
        }
    }
    return result;
}
```

**Q: Why does `maxVal` only increase?**
A: We only add elements from lists, and each list is sorted ascending. When we advance to the next element of a list, the new value is ≥ the old value. So `maxVal` is non-decreasing. The minimum of the heap (= `heap.top()`) is the range's lower bound; we minimize the range by advancing the minimum.

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
