> **Topic:** [Greedy Algorithms](../README.md) · **Interview Problems — Google**

# Google Interview — Greedy Algorithms Deep Dive

## Problem 1: Partition Labels (#763) — Deep Dive

### Problem Statement
You have a string `s`. Partition it into as many parts as possible so that each letter appears in at most one part. Return a list of integers representing the size of these parts.

### Two-Phase Greedy Strategy

**Phase 1:** Build `last[c]` = last index where character `c` appears. O(n).

**Phase 2:** Scan left to right. Maintain a window `[start, end]`. For each character, extend `end` to `max(end, last[c])` ("stretch window"). When `i == end`, the window is complete → record `end - start + 1`, advance `start`.

### "Stretch Window" Logic Explained
```
s = "ababcbacadefegdehijhklij"
     0123456789...

last['a'] = 8, last['b'] = 5, last['c'] = 7, last['d'] = 14, ...

i=0 (a): end = max(0, last[a]=8) = 8
i=1 (b): end = max(8, last[b]=5) = 8  ← b ends before a, no stretch needed
i=2 (a): end = max(8, last[a]=8) = 8
...
i=8 (a): end = max(8, 8) = 8
  → i == end! Partition [0,8] → size = 9

i=9 (d): start=9, end = max(9, last[d]=14) = 14
...
i=14(e): end = max(14, last[e]=15) = 15
  → continue stretching...
i=15: → i == end! Partition [9,15] → size = 7

i=16 (h): start=16, end = max(16, last[h]=19) = 19
...
i=23: → i == end! Partition [16,23] → size = 8
```

**Result: [9, 7, 8]** ✓

### Full C++ Solution
```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> partitionLabels(string s) {
    // Phase 1: Build last-occurrence map
    int last[26] = {};
    for (int i = 0; i < (int)s.length(); i++) {
        last[s[i] - 'a'] = i;
    }

    // Phase 2: Greedy window stretching
    vector<int> result;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.length(); i++) {
        end = max(end, last[s[i] - 'a']); // stretch window
        if (i == end) { // window complete
            result.push_back(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}
```

### Why Greedy Is Optimal
**Claim:** We cannot split the window `[start, end]` further.

**Proof:** Suppose there's a split point `m` where `start < m < end`. Then some character `c` appears at both some index `≤ m` and some index `> m` (otherwise `end` would not have been stretched past `m`). So `c` would appear in both partitions — violating the constraint. ∎

---

## Problem 2: Meeting Rooms II (#253) — Min-Heap Analysis

### Problem Statement
Given an array of meeting time intervals `[start, end]`, find the minimum number of conference rooms required.

### Why Min-Heap?
We need to know: "Is there a room that becomes free by the time this meeting starts?"
- A **min-heap of end times** lets us check the earliest-ending room in O(1).
- If `heap.top() <= newStart`: that room is free — reuse it (pop old end, push new end).
- Else: all rooms are busy — need a new room (just push new end).
- **Heap size at the end = number of rooms.**

### Full C++ Solution
```cpp
#include <bits/stdc++.h>
using namespace std;

int minMeetingRooms(vector<vector<int>>& intervals) {
    // Sort by start time to process meetings chronologically
    sort(intervals.begin(), intervals.end(), [](auto& a, auto& b){ return a[0] < b[0]; });

    // Min-heap of end times — top() gives earliest-ending meeting
    priority_queue<int, vector<int>, greater<int>> minHeap;

    for (auto& interval : intervals) {
        // If the earliest-ending room is free before this meeting starts, reuse it
        if (!minHeap.empty() && minHeap.top() <= interval[0]) {
            minHeap.pop(); // Free up the room
        }
        minHeap.push(interval[1]); // Assign this meeting a room (new or reused)
    }

    return (int)minHeap.size(); // Active rooms = total rooms needed
}
```

### Amortized Analysis

| Operation | Frequency | Cost | Total |
|-----------|-----------|------|-------|
| `push(end)` | n times | O(log n) | O(n log n) |
| `pop()` | ≤ n times | O(log n) | O(n log n) |
| `top()` | n times | O(1) | O(n) |
| `sort` | once | O(n log n) | O(n log n) |
| **Total** | | | **O(n log n)** |

**Space:** O(n) for the heap in worst case (all meetings overlap).

### Greedy vs DP Decision Framework

| Question | Choose Greedy | Choose DP |
|----------|--------------|-----------|
| "Minimum resources for non-conflicting tasks?" | ✓ Greedy (interval scheduling) | ✗ |
| "Minimum cost path with constraints?" | ✗ | ✓ DP |
| "Can we reach the target?" | ✓ Greedy (Jump Game) | ✓ also works |
| "Count ways to partition?" | ✗ | ✓ DP |
| "Maximize satisfied items with binary choice per item?" | ✓ Greedy (sort + scan) | ✗ |
| "Maximize satisfied items with complex dependencies?" | ✗ | ✓ DP |

**Greedy heuristic:** If "take the locally best choice now never hurts future choices" can be proven (exchange argument or greedy stays ahead), use greedy. If future choices depend on current choices in complex ways, use DP.

---

## Google Interview Progression

| Round | Expected | Greedy Focus |
|-------|----------|--------------|
| Phone Screen | Clean O(n log n) solution | Partition Labels or Meeting Rooms II |
| Coding Interview 1 | Optimal + complexity justification | Both problems + follow-ups |
| Coding Interview 2 | Alternative approaches | Sweep line for Meeting Rooms, Unicode for Partition Labels |
| System Design | Scale to distributed context | "Design a distributed meeting room scheduler" |
| Behavioral | Structured STAR answers | "Tell me about a time you simplified a complex algorithm" |

> **Last Updated:** 2026-06-26
