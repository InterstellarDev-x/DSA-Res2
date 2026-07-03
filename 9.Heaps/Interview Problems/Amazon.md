# Amazon — Heaps Interview Problems

> **Topic:** [Heaps](../README.md) · **Company:** Amazon

---

## Problem 1: Top K Frequent Elements — Deep Dive

**LC 347** · Medium

### Two Approaches

**Approach 1: Min-heap of size k — O(n log k)**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int n : nums) freq[n]++;

    // min-heap by frequency: pair<freq, num>
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<pair<int,int>>> minHeap;

    for (auto& [num, f] : freq) {
        minHeap.push({f, num});
        if ((int)minHeap.size() > k) minHeap.pop();
    }

    vector<int> result(k);
    for (int i = k - 1; i >= 0; i--) {
        result[i] = minHeap.top().second;
        minHeap.pop();
    }
    return result;
}
```

**Approach 2: Bucket sort — O(n)**
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int n : nums) freq[n]++;

    vector<vector<int>> buckets(nums.size() + 1);
    for (auto& [num, f] : freq) {
        buckets[f].push_back(num);
    }

    vector<int> result;
    for (int f = (int)buckets.size() - 1; f >= 0 && (int)result.size() < k; f--)
        for (int n : buckets[f])
            if ((int)result.size() < k) result.push_back(n);
    return result;
}
```

**Q: When would you use bucket sort over heap?**
A: When the input values are bounded (nums are integers in a known range). Bucket sort is O(n) but uses O(n) space for the bucket array. For floating-point frequencies or very sparse data, heap is more appropriate.

**Q: Amazon follow-up — Top K Frequent Words with lexicographic tie-breaking?**
A: Add a second comparator condition: for equal frequencies, lexicographically later strings should be evicted first from the min-heap.

---

## Problem 2: Task Scheduler — Amazon Deep Dive

**LC 621** · Medium

**Both approaches must be known:**

```cpp
#include <bits/stdc++.h>
using namespace std;

// Formula — O(26 log 26) = O(1) effectively
int leastInterval(vector<char>& tasks, int n) {
    vector<int> freq(26, 0);
    for (char t : tasks) freq[t - 'A']++;
    sort(freq.begin(), freq.end());
    int maxFreq = freq[25];
    int maxCount = 0;
    for (int f : freq) if (f == maxFreq) maxCount++;
    return max((maxFreq - 1) * (n + 1) + maxCount, (int)tasks.size());
}
```

**Q: Explain the formula.**
A: Build "frames" of size `n+1`. The most frequent task (`maxFreq`) anchors each frame. There are `maxFreq - 1` full frames, each of size `n+1`. The final slot holds all tasks with max frequency (`maxCount`). Total = `(maxFreq-1)*(n+1) + maxCount`. But if tasks fill all slots with no idle time, the answer is just `tasks.size()`.

**Q: Amazon variant — what if n is very large?**
A: For large n, idle time dominates. The formula still handles this correctly — `(maxFreq-1)*(n+1) + maxCount` grows with n.

**Q: Amazon variant — different cooldown per task type?**
A: The heap simulation approach generalizes to this. The cooldown queue can track per-task cooldowns. The formula approach doesn't generalize.

---

## Problem 3: Merge K Sorted Lists — Amazon Phone Screen

**LC 23** · Hard · O(n log k)

```cpp
#include <bits/stdc++.h>
using namespace std;

struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x): val(x), next(nullptr){}
};

ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> heap(cmp);
    for (ListNode* node : lists) if (node != nullptr) heap.push(node);

    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (!heap.empty()) {
        tail->next = heap.top(); heap.pop();
        tail = tail->next;
        if (tail->next != nullptr) heap.push(tail->next);
    }
    return dummy.next;
}
```

**Q: Why not just concatenate and sort?**
A: O(n log n) where n = total nodes. K-way merge with heap is O(n log k) — better when k << n (many short lists).

**Q: What if k is very large (k = n lists, each length 1)?**
A: Building the heap costs O(k) = O(n). Each poll is O(log k) = O(log n). Total O(n log n) — same as sort. For this degenerate case, sorting is simpler.

---

## Amazon LP Alignment

| LP | Connection |
|----|-----------|
| Dive Deep | Know both heap and bucket sort for Top K; formula and simulation for Task Scheduler |
| Deliver Results | O(n log k) beats O(n log n) for Merge K Lists when k is small |
| Invent & Simplify | Bucket sort O(n) simplifies Top K when input range is bounded |

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Top K Elements](../Patterns/Top%20K%20Elements.md)
- [Heap + Frequency](../Patterns/Heap%20and%20Frequency.md)

> **Last Updated:** 2026-06-26
