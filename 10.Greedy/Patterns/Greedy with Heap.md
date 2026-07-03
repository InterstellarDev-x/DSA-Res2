> **Topic:** [Greedy Algorithms](../README.md) · **Pattern 4 of 4**

# Greedy with Heap

---

## 1. Core Concept

Some greedy problems require efficiently finding the "best available" choice at each step — one that cannot be determined with a simple sort + scan. These problems need a **priority queue (heap)** to maintain the relevant ordering dynamically as items are consumed and added.

The combination works as follows:
1. **Greedy strategy** defines which item to pick next (smallest end time, most frequent task, smallest card, etc.).
2. **Heap** makes retrieving that item O(log n) rather than O(n) per step.

**When to use Greedy + Heap (vs. plain greedy):**

| Condition | Use plain greedy | Use greedy + heap |
|-----------|-----------------|-------------------|
| Selection order fixed (sort once) | Yes | No |
| Dynamic insertion/removal of candidates | No | Yes |
| Need to query min/max of evolving set | No | Yes |
| std::map (sorted map) for frequency/order needed | No | Yes |

---

## 2. Pattern Recognition Signals

| Signal | Approach |
|--------|----------|
| "Minimum rooms / workers / machines needed" | Sort by start; min-heap of end times |
| "Group consecutive elements into fixed-size groups" | std::map to process smallest key first |
| "Schedule tasks with cooldown" | Frequency array + formula, or max-heap simulation |
| "Assign tasks to available workers" | Min-heap of (availability_time) |
| "Process in sorted order while dynamically updating counts" | std::map (navigable sorted map) |

---

## 3. Problems

---

### 1. Hand of Straights (#846)

**Problem:** Given an array `hand` of cards and a `groupSize`, determine if you can rearrange all cards into groups of `groupSize` consecutive cards.

**Key Insight:** Use a `std::map<int, int>` to count card frequencies, keeping keys in sorted order. At each step, take the smallest card (`begin()->first`) and try to form a consecutive group of length `groupSize` starting from it. If any required card is missing, return false. Decrement counts (removing key when count reaches 0) and repeat until the map is empty.

**Why std::map (not std::unordered_map):** We must always start a new group with the current smallest card. An `std::unordered_map` has no `begin()->first` in sorted order. A `std::map` maintains sorted order and provides O(log n) `begin()->first`, `count()`, and `erase()`.

**Why explicit decrement and erase:** We decrement `count[i]` and then check `if (count[i] == 0) count.erase(i)` to remove the key cleanly, ensuring `begin()->first` always returns the true minimum.

```cpp
#include <bits/stdc++.h>
using namespace std;

class HandOfStraights {
public:
    bool isNStraightHand(vector<int>& hand, int groupSize) {
        if (hand.size() % groupSize != 0) return false;
        map<int, int> count;
        for (int card : hand) count[card]++;
        while (!count.empty()) {
            int first = count.begin()->first;
            for (int i = first; i < first + groupSize; i++) {
                if (!count.count(i)) return false;
                count[i]--;
                if (count[i] == 0) count.erase(i);
            }
        }
        return true;
    }
};

int main() {
    HandOfStraights sol;
    vector<int> hand1 = {1,2,3,6,2,3,4,7,8};
    vector<int> hand2 = {1,2,3,4,5};
    cout << sol.isNStraightHand(hand1, 3) << "\n"; // true (1)
    cout << sol.isNStraightHand(hand2, 4) << "\n"; // false (0)
    return 0;
}
```

**Complexity:**
- Time: O(n log n) — each card is inserted into the std::map once (O(log n)) and processed once (O(log n)). Total O(n log n).
- Space: O(k) where k = number of distinct card values.

**Dry Run (hand=[1,2,3,6,2,3,4,7,8], groupSize=3):**
```
count = {1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}

Iteration 1: first=1
  i=1: count={1:1}  exists, count[1]--  {1:0}  erase  count={2:2,3:2,4:1,6:1,7:1,8:1}
  i=2: count={2:2}  exists, count[2]--  {2:1}          count={2:1,3:2,4:1,6:1,7:1,8:1}
  i=3: count={3:2}  exists, count[3]--  {3:1}          count={2:1,3:1,4:1,6:1,7:1,8:1}

Iteration 2: first=2
  i=2: erase 2  count={3:1,4:1,6:1,7:1,8:1}
  i=3: erase 3  count={4:1,6:1,7:1,8:1}
  i=4: erase 4  count={6:1,7:1,8:1}

Iteration 3: first=6
  i=6: erase 6  count={7:1,8:1}
  i=7: erase 7  count={8:1}
  i=8: erase 8  count={}

count.empty()  return true
```

---

### 2. Meeting Rooms II (#253)

**Problem:** Given an array of meeting intervals `[start, end]`, find the minimum number of conference rooms required.

**Key Insight:** Sort meetings by start time. Use a min-heap of end times to represent "rooms currently in use." For each new meeting:
- If the heap's minimum end time <= new meeting's start time, the earliest-ending meeting has finished — reuse that room (pop from heap).
- Add the new meeting's end time to the heap.

After processing all meetings, the heap size equals the number of rooms needed.

**Why min-heap on end times:** At each step, we want to know "is there any room whose meeting has ended by the time this new meeting starts?" The min-heap answers this in O(log n): just check the top (earliest end time). If even the earliest-ending meeting hasn't finished, we need a new room.

**Amortized analysis:** Each meeting is pushed once and popped at most once. Total operations: O(n log n) for sort + O(n log n) for heap operations (n pushes and at most n pops, each O(log n)).

```cpp
#include <bits/stdc++.h>
using namespace std;

class MeetingRoomsII {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
            return a[0] < b[0];
        });
        priority_queue<int, vector<int>, greater<int>> minHeap;
        for (auto& interval : intervals) {
            if (!minHeap.empty() && minHeap.top() <= interval[0]) minHeap.pop();
            minHeap.push(interval[1]);
        }
        return minHeap.size();
    }
};

int main() {
    MeetingRoomsII sol;
    vector<vector<int>> intervals1 = {{0,30},{5,10},{15,20}};
    vector<vector<int>> intervals2 = {{7,10},{2,4}};
    cout << sol.minMeetingRooms(intervals1) << "\n"; // 2
    cout << sol.minMeetingRooms(intervals2) << "\n"; // 1
    return 0;
}
```

**Complexity:**
- Time: O(n log n) — sort O(n log n) + heap operations O(n log n).
- Space: O(n) — heap can hold up to n end times in the worst case (all meetings overlap).

---

#### Detailed Dry Run — Meeting Rooms II: [[0,30],[5,10],[15,20]]

```
intervals sorted by start: [[0,30],[5,10],[15,20]]

minHeap = []

interval=[0,30]:
  heap empty  no pop
  push 30  minHeap=[30]
  rooms in use: {Room A ends at 30}

interval=[5,10]:
  top=30, interval[0]=5  30 <= 5? No  no pop (Room A still busy)
  push 10  minHeap=[10, 30]
  rooms in use: {Room A ends at 30, Room B ends at 10}

interval=[15,20]:
  top=10, interval[0]=15  10 <= 15? Yes  pop 10 (Room B is free)
  push 20  minHeap=[20, 30]
  rooms in use: {Room A ends at 30, Room B re-used ends at 20}

minHeap.size() = 2  return 2
```

**Visualization:**
```
Timeline:  0----5----10---15---20---25---30
Meeting 1: [============================]   Room A
Meeting 2:      [====]                      Room B (freed at 10)
Meeting 3:                [=====]           Room B (reused from 15)

Max concurrent at any time: 2 (at t=5 to t=10)
```

---

### 3. Task Scheduler (#621)

**Problem:** Given `tasks[]` (each is a letter 'A'-'Z') and a cooldown `n`, schedule tasks such that the same task type must be at least `n` intervals apart. Idle slots are allowed. Return the minimum number of intervals to finish all tasks.

**Key Insight — Formula approach:**

Let `maxFreq` = maximum frequency of any task (e.g., if 'A' appears 3 times, `maxFreq = 3`).
Let `maxCount` = number of tasks that appear `maxFreq` times.

The most frequent tasks create a "skeleton" of `(maxFreq - 1)` frames of size `(n + 1)`, plus a final slot for all tasks with max frequency:

```
Minimum intervals = max(tasks.length, (maxFreq - 1) * (n + 1) + maxCount)
```

- `(maxFreq - 1) * (n + 1)`: total slots in all complete frames.
- `+ maxCount`: final frame only needs the most-frequent tasks (no cooldown needed after the last occurrence).
- `max(..., tasks.length)`: if there are enough other tasks to fill all idle slots, no idles are needed and the answer is simply `tasks.length`.

**Formula derivation with example (tasks=[A,A,A,B,B,B,C,C], n=2):**
```
Frequencies: A=3, B=3, C=2
maxFreq=3, maxCount=2 (A and B both have freq 3)

Frame structure (each frame = n+1 = 3 slots):
Frame 1: [A, B, C]
Frame 2: [A, B, C]
Final:   [A, B]       <- only max-frequency tasks

Total = (3-1)*(2+1) + 2 = 2*3 + 2 = 8

tasks.length = 8, max(8, 8) = 8
```

If there are more varied tasks: suppose tasks=[A,A,A,B,C,D,E,F,G], n=2:
```
maxFreq=3, maxCount=1
Formula: (3-1)*(3)+1 = 7
tasks.length=9 > 7  answer=9 (no idles needed, other tasks fill all gaps)
```

```cpp
#include <bits/stdc++.h>
using namespace std;

class TaskScheduler {
public:
    int leastInterval(string tasks, int n) {
        int freq[26] = {};
        for (char t : tasks) freq[t - 'A']++;
        sort(freq, freq + 26);
        int maxFreq = freq[25];
        int maxCount = 0;
        for (int f : freq) if (f == maxFreq) maxCount++;
        return max((int)tasks.size(), (maxFreq - 1) * (n + 1) + maxCount);
    }
};

int main() {
    TaskScheduler sol;
    cout << sol.leastInterval("AAABBB", 2) << "\n"; // 8
    cout << sol.leastInterval("AAABBB", 0) << "\n"; // 6
    cout << sol.leastInterval("AAAABBBCCCDDDEEEFFF", 2) << "\n"; // 19
    return 0;
}
```

**Complexity:**
- Time: O(n + 26 log 26) = O(n). Sorting a fixed-size array of 26 is O(1).
- Space: O(1) — `freq[]` array is size 26.

**Cross-reference:** For the max-heap simulation approach (useful when you need to output the actual schedule), see the [Heaps topic](../../9.Heaps/README.md). The heap simulation processes tasks greedily by always executing the most frequent available task, inserting idle slots when no task is available in cooldown. Both approaches give the same answer; the formula is O(1) space while the heap simulation is O(26) = O(1) space but O(n log 26) time with cleaner extensibility.

---

## 4. Complexity Summary

| Problem | Time | Space | Data Structure |
|---------|------|-------|----------------|
| Hand of Straights (#846) | O(n log n) | O(k) distinct values | `std::map<int,int>` |
| Meeting Rooms II (#253) | O(n log n) | O(n) worst case | `std::priority_queue<int>` (min-heap) |
| Task Scheduler (#621) | O(n) | O(1) | `int[26]` freq array |

---

## 5. Pattern Comparison

| Problem | Heap Role | Key Greedy Decision |
|---------|-----------|---------------------|
| Hand of Straights | std::map processes smallest-first; ensures consecutive groups formed left-to-right | Always start a new group from the minimum available card |
| Meeting Rooms II | Min-heap of end times; O(log n) check for earliest-freed room | Reuse room with earliest end if it finishes before new meeting starts |
| Task Scheduler | No heap (formula); heap variant uses max-heap to pick most-frequent task | Always execute the most-frequent available task to minimize idle slots |

---

> **Last Updated:** 2026-06-26
