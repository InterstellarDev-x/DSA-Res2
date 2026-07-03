# K-Way Merge

> **Topic:** [Heaps](../README.md) · **Pattern 3 of 5**
> **Problems:** Merge K Sorted Lists · Kth Smallest in Matrix · K Pairs Smallest Sums · Sort K-Sorted Array · Smallest Range from K Lists

---

## Core Concept

**K-way merge** efficiently merges k sorted sequences or finds the kth element across k sorted sequences using a min-heap.

**Template:**
1. Initialize heap with the **first element** from each of the k lists
2. Poll the minimum element (current global minimum)
3. Push the **next element** from the same list that was just polled
4. Repeat until heap is empty or target is found

Each element enters and exits the heap at most once → **O(n log k)** total, where n = total elements.

---

## Problem 1: Merge K Sorted Lists — LC 23

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode { int val; ListNode* next; ListNode(int x): val(x), next(nullptr){} };

ListNode* mergeKLists(vector<ListNode*>& lists) {
    // Heap stores nodes; compare by val
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> heap(cmp);

    // Initialize: push head of each non-empty list
    for (auto node : lists) {
        if (node != nullptr) heap.push(node);
    }

    ListNode* dummy = new ListNode(0), *tail = dummy;

    while (!heap.empty()) {
        ListNode* node = heap.top(); heap.pop();
        tail->next = node;
        tail = tail->next;
        if (node->next != nullptr) heap.push(node->next);
    }

    return dummy->next;
}
```

**Complexity:** O(n log k) time, O(k) space (heap holds at most one node per list)

**Why comparator on `n->val` not subtraction?** Node values could be `INT_MIN` / `INT_MAX` — subtraction overflows. Use a lambda `[](ListNode* a, ListNode* b){ return a->val > b->val; }` or explicit compare.

---

## Problem 2: Sort an Almost Sorted Array (Sort K-Sorted Array)

Each element is at most k positions away from its correct position.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> sortKSortedArray(vector<int>& arr, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    vector<int> result(arr.size());
    int idx = 0;

    // Add first k+1 elements to the heap
    for (int i = 0; i < min(k + 1, (int)arr.size()); i++) {
        minHeap.push(arr[i]);
    }

    // Slide window: for each new element, extract minimum
    for (int i = k + 1; i < (int)arr.size(); i++) {
        result[idx++] = minHeap.top(); minHeap.pop();
        minHeap.push(arr[i]);
    }

    // Drain the heap
    while (!minHeap.empty()) { result[idx++] = minHeap.top(); minHeap.pop(); }
    return result;
}
```

**Why window of k+1?** An element at position i can be at most k positions ahead in the sorted array, meaning it's at most at position i+k. To place element correctly at position i, we must consider elements from position i to i+k — that's k+1 elements.

**Complexity:** O(n log k) time, O(k) space

---

## Problem 3: Find K Pairs with Smallest Sums — LC 373

Given sorted arrays `nums1`, `nums2`, find k pairs `(nums1[i], nums2[j])` with smallest sums.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> kSmallestPairs(vector<int>& nums1, vector<int>& nums2, int k) {
    vector<vector<int>> result;
    if (nums1.empty() || nums2.empty()) return result;

    // Min-heap: [sum, i, j] — initially push (nums1[i], nums2[0]) for all i
    auto cmp = [](vector<int>& a, vector<int>& b) { return a[0] > b[0]; };
    priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> heap(cmp);

    // Key optimization: only push first min(k, nums1.size()) pairs initially
    for (int i = 0; i < min(k, (int)nums1.size()); i++) {
        heap.push({nums1[i] + nums2[0], i, 0});
    }

    while (!heap.empty() && (int)result.size() < k) {
        auto curr = heap.top(); heap.pop();
        int i = curr[1], j = curr[2];
        result.push_back({nums1[i], nums2[j]});
        if (j + 1 < (int)nums2.size()) {
            heap.push({nums1[i] + nums2[j + 1], i, j + 1});
        }
    }
    return result;
}
```

**Key insight:** For each row `i` in the conceptual matrix of all `(i,j)` pairs, only advance `j` (columns in sorted order). Initialize with all `(i, 0)` pairs (first column). When we pick `(i, j)`, push `(i, j+1)`.

**Why not push all pairs initially?** O(n²) space. We only need O(k) pairs.

---

## Problem 4: Kth Smallest Element in Sorted Matrix — LC 378

Matrix where each row and column is sorted.

```cpp
#include <bits/stdc++.h>
using namespace std;

int kthSmallest(vector<vector<int>>& matrix, int k) {
    int n = matrix.size();
    // Min-heap: [value, row, col]
    auto cmp = [](vector<int>& a, vector<int>& b) { return a[0] > b[0]; };
    priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> heap(cmp);

    // Push first element of each row (first column)
    for (int r = 0; r < min(n, k); r++) {
        heap.push({matrix[r][0], r, 0});
    }

    int result = 0;
    for (int i = 0; i < k; i++) {
        auto curr = heap.top(); heap.pop();
        result = curr[0];
        int r = curr[1], c = curr[2];
        if (c + 1 < n) heap.push({matrix[r][c + 1], r, c + 1});
    }
    return result;
}
```

**Complexity:** O(k log k) time for k polls — better than O(n² log n) sort if k is small.

**Alternative — Binary Search — O(n log(max-min)):**
Binary search on the answer value; for each mid, count elements ≤ mid using the staircase search (O(n) per count). Use when k is large.

---

## Problem 5: Smallest Range Covering Elements from K Lists — LC 632

Find the smallest range `[a, b]` such that each of the k lists has at least one element in `[a, b]`.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> smallestRange(vector<vector<int>>& nums) {
    // Min-heap: [value, list-index, element-index]
    auto cmp = [](vector<int>& a, vector<int>& b) { return a[0] > b[0]; };
    priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> heap(cmp);
    int maxVal = INT_MIN;

    // Push first element from each list
    for (int i = 0; i < (int)nums.size(); i++) {
        heap.push({nums[i][0], i, 0});
        maxVal = max(maxVal, nums[i][0]);
    }

    vector<int> result = {heap.top()[0], maxVal};

    while (true) {
        auto curr = heap.top(); heap.pop();
        int val = curr[0], listIdx = curr[1], elemIdx = curr[2];

        if (elemIdx + 1 == (int)nums[listIdx].size()) break;  // exhausted a list

        int nextVal = nums[listIdx][elemIdx + 1];
        heap.push({nextVal, listIdx, elemIdx + 1});
        maxVal = max(maxVal, nextVal);

        int minVal = heap.top()[0];
        if (maxVal - minVal < result[1] - result[0]) {
            result[0] = minVal; result[1] = maxVal;
        }
    }
    return result;
}
```

**Invariant:** At all times, the heap contains exactly one element from each list. The range is `[heap.top(), maxVal]`. We minimize this range by advancing the minimum (popping heap root and pushing next from same list). We track `maxVal` separately as it can only increase.

**Why stop when a list is exhausted?** We need one element from each list. If any list has no more elements, the current minimum cannot be replaced — the range can only get worse.

---

## K-Way Merge — Heap Entry Design

| Problem | Heap Entry | Compare By |
|---------|-----------|------------|
| Merge K Sorted Lists | `ListNode*` | `node->val` |
| K Pairs Smallest Sums | `[sum, i, j]` | `sum` |
| Kth Smallest in Matrix | `[val, row, col]` | `val` |
| Smallest Range | `[val, listIdx, elemIdx]` | `val` |
| Sort K-Sorted | `int` | natural order |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [Two Heaps](./Two%20Heaps.md)
- [Merge Linked Lists (Linked List topic)](../../4.Linked_List/Patterns/Merge%20Linked%20Lists.md)
- [Complexity Analysis](../Interview%20Tips/Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
