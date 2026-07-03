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
2. **Order property:** `lo.top() ≤ hi.top()` (every lower-half element ≤ every upper-half element)

**Median:**
- If `lo.size() == hi.size()`: median = `(lo.top() + hi.top()) / 2.0`
- If `lo.size() > hi.size()`: median = `lo.top()`

---

## Problem 1: Find Median from Data Stream — LC 295

```cpp
#include <bits/stdc++.h>
using namespace std;

class MedianFinder {
    priority_queue<int> lo;                            // max-heap (default)
    priority_queue<int, vector<int>, greater<int>> hi; // min-heap

public:
    void addNum(int num) {
        // Step 1: Route to correct heap
        if (lo.empty() || num <= lo.top()) lo.push(num);
        else hi.push(num);

        // Step 2: Rebalance so |lo.size() - hi.size()| <= 1
        if (lo.size() > hi.size() + 1) { hi.push(lo.top()); lo.pop(); }
        else if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
    }

    double findMedian() {
        if (lo.size() == hi.size()) return (lo.top() + hi.top()) / 2.0;
        return lo.top();   // lo always has the extra element
    }
};
```

**Trace for addNum sequence [5, 3, 8, 1]:**
```
addNum(5): lo=[5], hi=[]
addNum(3): 3<=5 → lo=[5,3]. lo.size=2 > hi.size+1=1 → move to hi. lo=[5], hi=[3]
           Wait — rebalance: lo.size(2) > hi.size(0)+1=1 → hi.offer(lo.poll()=5) → lo=[3], hi=[5]
           Actually: lo.size=2 > hi.size+1=1, rebalance → hi.offer(lo.poll()) → lo=[3], hi=[5]
           Hmm, 3 should be in lo (lower half). Let me retrace:
           addNum(3): 3 <= lo.top()=5 → lo.push(3) → lo=[5,3]. Rebalance: lo.size=2 > hi.size(0)+1=1 → hi.push(lo.top()=5); lo.pop() → lo=[3], hi=[5]
           findMedian: lo.size=hi.size=1 → (3+5)/2=4
addNum(8): 8 > lo.top()=3 → hi.push(8) → hi=[5,8]. Rebalance: hi.size=2 > lo.size=1 → lo.push(hi.top()=5); hi.pop() → lo=[5,3], hi=[8]
           findMedian: lo.size=2 > hi.size=1 → lo.top()=5
addNum(1): 1 <= lo.top()=5 → lo.push(1) → lo=[5,3,1]. Rebalance: lo.size=3 > hi.size+1=2 → hi.push(lo.top()=5); lo.pop() → lo=[3,1], hi=[5,8]
           findMedian: lo.size=hi.size=2 → (3+5)/2=4
```

**Complexity:** O(log n) addNum, O(1) findMedian, O(n) space

---

## Problem 2: Sliding Window Median — LC 480

Find the median of each window of size k as it slides.

**Challenge:** Standard two-heap doesn't support removal of arbitrary elements. Solution: lazy deletion using a `toRemove` counter map.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<double> medianSlidingWindow(vector<int>& nums, int k) {
    // map acts as ordered multiset with counts
    map<int,int> lo; // lower half — use rbegin() for max
    map<int,int> hi; // upper half — use begin() for min
    int loSize = 0, hiSize = 0;

    vector<double> result(nums.size() - k + 1);

    for (int i = 0; i < (int)nums.size(); i++) {
        // Add incoming element
        if (lo.empty() || nums[i] <= lo.rbegin()->first) {
            lo[nums[i]]++; loSize++;
        } else {
            hi[nums[i]]++; hiSize++;
        }

        // Rebalance
        if (loSize > hiSize + 1) {
            int top = lo.rbegin()->first;
            hi[top]++; hiSize++;
            if (--lo[top] == 0) lo.erase(top);
            loSize--;
        } else if (hiSize > loSize) {
            int top = hi.begin()->first;
            lo[top]++; loSize++;
            if (--hi[top] == 0) hi.erase(top);
            hiSize--;
        }

        // Window is full starting at i == k-1
        if (i >= k - 1) {
            result[i - k + 1] = (loSize == hiSize)
                ? ((double)lo.rbegin()->first + hi.begin()->first) / 2.0
                : lo.rbegin()->first;

            // Remove outgoing element (leftmost of window)
            int out = nums[i - k + 1];
            if (lo.count(out)) {
                if (--lo[out] == 0) lo.erase(out);
                loSize--;
            } else {
                if (--hi[out] == 0) hi.erase(out);
                hiSize--;
            }

            // Rebalance after removal
            if (loSize > hiSize + 1) {
                int top = lo.rbegin()->first;
                hi[top]++; hiSize++;
                if (--lo[top] == 0) lo.erase(top);
                loSize--;
            } else if (hiSize > loSize) {
                int top = hi.begin()->first;
                lo[top]++; loSize++;
                if (--hi[top] == 0) hi.erase(top);
                hiSize--;
            }
        }
    }
    return result;
}
```

**Why `std::map` instead of `std::priority_queue`?** `priority_queue` does not support O(log n) arbitrary removal. `std::map` with erase is O(log n). For a sliding window, O(log n) removal is essential.

---

## Problem 3: IPO — LC 502

Maximize capital after k projects. Each project has profit and capital requirement.

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMaximizedCapital(int k, int w, vector<int>& profits, vector<int>& capital) {
    int n = profits.size();
    // Min-heap by capital requirement — projects we can't afford yet
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> locked;
    // Max-heap by profit — projects we can afford; pick most profitable
    priority_queue<int> available;

    for (int i = 0; i < n; i++) locked.push({capital[i], profits[i]});

    for (int i = 0; i < k; i++) {
        // Unlock all projects we can now afford
        while (!locked.empty() && locked.top().first <= w) {
            available.push(locked.top().second);
            locked.pop();
        }
        if (available.empty()) break;   // no affordable projects
        w += available.top();           // pick most profitable
        available.pop();
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
| Median Stream | lower half | upper half | \|lo\|-\|hi\| ≤ 1; lo.top ≤ hi.top |
| Sliding Window Median | lower half of window | upper half of window | Same + need removal |
| IPO | available projects (by profit) | locked projects (by capital) | Different semantic — unlock to available on each step |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [MedianFinder Design](../Design%20Data%20Structure%20Problems/Median%20Finder.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
