# Top K Elements

> **Topic:** [Heaps](../README.md) · **Pattern 2 of 5**
> **Problems:** Kth Largest in Array/Stream · K Closest Points · Top K Frequent Elements/Words · Kth Largest Integer

---

## Core Concept

**Goal:** Find the k largest (or smallest, or most frequent) elements efficiently.

**Key insight — min-heap of size k for "k largest":**
- Maintain a min-heap of the k largest elements seen so far
- The root of this heap = kth largest element
- When a new element arrives: push it; if heap size > k, pop (removes the smallest, keeping the k largest)

```
Finding k largest from [3,1,4,1,5,9,2,6], k=3:
Min-heap of size 3:
  After 3:     [3]
  After 1:     [1,3]
  After 4:     [1,3,4]
  After 1 (4th): push 1 → [1,1,3,4]; size>3 → pop 1 → [1,3,4]
  After 5:     push 5 → [1,3,4,5]; pop 1 → [3,4,5]
  After 9:     push 9 → [3,4,5,9]; pop 3 → [4,5,9]
  After 2:     push 2 → [2,4,5,9]; pop 2 → [4,5,9]
  After 6:     push 6 → [4,5,6,9]; pop 4 → [5,6,9]
  Root = 5 = kth largest ✓
```

**Template:**
```cpp
#include <bits/stdc++.h>
using namespace std;

priority_queue<int, vector<int>, greater<int>> minHeap;  // min-heap of size k
for (int num : nums) {
    minHeap.push(num);
    if (minHeap.size() > k) minHeap.pop();   // remove smallest; keep k largest
}
return minHeap.top();  // root = kth largest
```

---

## Problem 1: Kth Largest Element in a Stream — LC 703

Design class: `KthLargest(k, nums)` + `add(val)` returns current kth largest.

```cpp
#include <bits/stdc++.h>
using namespace std;

class KthLargest {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    int k;
public:
    KthLargest(int k, vector<int>& nums) : k(k) {
        for (int n : nums) add(n);
    }

    int add(int val) {
        minHeap.push(val);
        if (minHeap.size() > k) minHeap.pop();
        return minHeap.top();
    }
};
```

**Complexity:** O(log k) per `add` — heap stays size ≤ k.

---

## Problem 2: Kth Largest Element in an Array — LC 215

**Approach 1: Min-heap of size k — O(n log k) time, O(k) space**

```cpp
#include <bits/stdc++.h>
using namespace std;

int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    for (int n : nums) {
        minHeap.push(n);
        if (minHeap.size() > k) minHeap.pop();
    }
    return minHeap.top();
}
```

**Approach 2: Quickselect — O(n) average, O(n²) worst, O(1) space**

```cpp
#include <bits/stdc++.h>
using namespace std;

int partitionArr(vector<int>& nums, int left, int right) {
    int pivot = nums[right], i = left;
    for (int j = left; j < right; j++) {
        if (nums[j] <= pivot) swap(nums[i++], nums[j]);
    }
    swap(nums[i], nums[right]);
    return i;
}

int quickselect(vector<int>& nums, int left, int right, int kSmallest) {
    if (left == right) return nums[left];

    int pivotIdx = partitionArr(nums, left, right);
    if (pivotIdx == kSmallest) return nums[pivotIdx];
    else if (pivotIdx < kSmallest) return quickselect(nums, pivotIdx + 1, right, kSmallest);
    else return quickselect(nums, left, pivotIdx - 1, kSmallest);
}

int findKthLargest(vector<int>& nums, int k) {
    return quickselect(nums, 0, nums.size() - 1, nums.size() - k);
    // kth largest = (n-k)th smallest (0-indexed)
}
```

**Which to use?**
- Heap: guaranteed O(n log k), preferred when k << n or for streaming data
- Quickselect: O(n) average, but O(n²) worst case; preferred when modifying the array is OK and k ≈ n/2

---

## Problem 3: K Closest Points to Origin — LC 973

Distance² = x² + y² (no need to take square root — avoids floating point).

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
    // Max-heap of size k — keep k smallest distances; root = kth closest
    auto cmp = [](const vector<int>& a, const vector<int>& b) {
        return a[0]*a[0] + a[1]*a[1] < b[0]*b[0] + b[1]*b[1];
    };
    priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> maxHeap(cmp);

    for (auto& p : points) {
        maxHeap.push(p);
        if (maxHeap.size() > k) maxHeap.pop();   // remove farthest
    }

    vector<vector<int>> result(k);
    for (int i = k - 1; i >= 0; i--) {
        result[i] = maxHeap.top();
        maxHeap.pop();
    }
    return result;
}
```

**Why max-heap for k smallest?** To keep the k closest, maintain a max-heap — the root is always the farthest among the k closest. When a new point is closer than the root, we replace the root (pop max, push new). This keeps only the k closest.

**Simpler alternative — sort:**
```cpp
#include <bits/stdc++.h>
using namespace std;

sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
    return a[0]*a[0] + a[1]*a[1] < b[0]*b[0] + b[1]*b[1];
});
return vector<vector<int>>(points.begin(), points.begin() + k);  // O(n log n) time, O(1) space
```

Use heap for streaming/online variant; sort for offline.

---

## Problem 4: Top K Frequent Elements — LC 347

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int, int> freq;
    for (int n : nums) freq[n]++;

    // Min-heap by frequency: keep k most frequent
    auto cmp = [](const pair<int,int>& a, const pair<int,int>& b) {
        return a.second > b.second;
    };
    priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> minHeap(cmp);

    for (auto& [key, val] : freq) {
        minHeap.push({key, val});
        if (minHeap.size() > k) minHeap.pop();   // remove least frequent
    }

    vector<int> result(k);
    for (int i = k - 1; i >= 0; i--) {
        result[i] = minHeap.top().first;
        minHeap.pop();
    }
    return result;
}
```

**O(n log k) time, O(n) space** (for freq map + heap of size k)

**Alternative — Bucket Sort — O(n) time:**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int, int> freq;
    for (int n : nums) freq[n]++;

    // Buckets: bucket[i] = list of elements with frequency i
    vector<vector<int>> buckets(nums.size() + 1);
    for (auto& [key, f] : freq) {
        buckets[f].push_back(key);
    }

    vector<int> result;
    for (int f = (int)buckets.size() - 1; f >= 0 && (int)result.size() < k; f--) {
        for (int n : buckets[f]) {
            if ((int)result.size() < k) result.push_back(n);
        }
    }
    return result;
}
```

---

## Problem 5: Top K Frequent Words — LC 692

Same as Top K Frequent Elements but with lexicographic tie-breaking.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<string> topKFrequent(vector<string>& words, int k) {
    unordered_map<string, int> freq;
    for (auto& w : words) freq[w]++;

    // Min-heap: least frequent at top (to be evicted); lexicographically LATER breaks ties
    auto cmp = [&](const string& a, const string& b) {
        int fa = freq[a], fb = freq[b];
        if (fa != fb) return fa > fb;         // lower freq → evict first
        return a < b;                         // lexicographically later → evict first
    };
    priority_queue<string, vector<string>, decltype(cmp)> minHeap(cmp);

    for (auto& [w, _] : freq) {
        minHeap.push(w);
        if (minHeap.size() > k) minHeap.pop();
    }

    vector<string> result;
    while (!minHeap.empty()) {
        result.push_back(minHeap.top());
        minHeap.pop();
    }
    reverse(result.begin(), result.end());  // reverse order
    return result;
}
```

**Why the comparator for tie-breaking?** We want lexicographically smaller words to stay (appear first in answer). The min-heap evicts the "worst" element. For ties in frequency, "worse" = lexicographically larger. So `a < b` as the condition means: if a comes before b, a is "better" (kept); b gets evicted first.

---

## Top K — Decision Table

| Goal | Heap Type | Size | Root Meaning |
|------|----------|------|--------------|
| k largest elements | Min-heap | k | kth largest |
| k smallest elements | Max-heap | k | kth smallest |
| k most frequent elements | Min-heap (by freq) | k | kth most frequent |
| k closest points | Max-heap (by dist) | k | kth closest |

**Rule:** For "k [extreme]", use the **opposite** heap type and maintain size k. The root is the answer.

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [K-Way Merge](./K%20Way%20Merge.md)
- [Heap + Frequency](./Heap%20and%20Frequency.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)

> **Last Updated:** 2026-06-26
