> **Topic:** [Greedy Algorithms](../README.md) · **Pattern 1 of 4**

# Interval Scheduling

---

## 1. Core Concept

Interval scheduling problems ask you to select or process a set of intervals (tasks, meetings, ranges) subject to constraints. The central greedy insight is:

> **Always process or select the interval with the earliest end time.**

By picking the interval that finishes soonest, you leave the maximum possible "free time" for future intervals, giving subsequent choices the best chance to succeed. This is the **Activity Selection** greedy strategy.

Variants of this pattern include:
- **Maximize count of non-overlapping intervals** — classic activity selection.
- **Merge or insert intervals** — sweep left-to-right, merging when overlap is detected.
- **Minimize removals** — equivalent to maximizing kept intervals (Activity Selection).
- **Circular / path problems** — Gas Station requires a single-pass variant with a "restart" trick.

---

## 2. Pattern Recognition Signals

| Signal in Problem Statement | Likely Approach |
|-----------------------------|----------------|
| "Maximum number of non-overlapping intervals / activities" | Sort by end time, greedy pick |
| "Minimum number of removals to make non-overlapping" | Sort by end, count kept, subtract |
| "Minimum arrows / shots to cover/burst all intervals" | Sort by end, shoot at end of first interval |
| "Partition string into maximum parts where each letter appears in only one part" | Last-occurrence array + expand window |
| "Insert a new interval and merge overlapping" | Three-phase sweep: before, overlap, after |
| "Can you complete a circuit / loop?" | Circular greedy with restart |
| "Assign resources (cookies, seats) to satisfy requirements" | Sort both arrays, two-pointer match |

---

## 3. Why Greedy Works — Activity Selection Proof (Exchange Argument)

**Claim:** Sorting by end time and greedily selecting compatible intervals yields the maximum number of non-overlapping intervals.

**Proof (Exchange Argument):**

Let `G = {g1, g2, ..., gk}` be the greedy solution (sorted by end time) and let `O = {o1, o2, ..., om}` be any other optimal solution (also sorted by end time). We show `k >= m`.

We prove inductively that for each `i`, `end(gi) <= end(oi)`.

- **Base case (i = 1):** The greedy algorithm picks the interval with the smallest end time among all intervals. Therefore `end(g1) <= end(o1)`.
- **Inductive step:** Assume `end(gi) <= end(oi)`. The next greedy interval `g(i+1)` is the earliest-ending interval that starts after `end(gi)`. Since `end(gi) <= end(oi)`, any interval compatible with `o(i+1)` (starting after `end(oi)`) also starts after `end(gi)`. So the greedy has at least as many choices available as OPT when picking `g(i+1)`. Therefore `end(g(i+1)) <= end(o(i+1))`.

Since greedy never runs out of choices before OPT does, `k >= m`. Because OPT is optimal, `k = m`. QED.

**Intuition:** A solution that picks an earlier-ending interval can only open more future possibilities — you can always "swap" a later-ending choice for an earlier-ending one without losing feasibility.

---

## 4. Problems

---

### 1. Assign Cookies (#455)

**Problem:** Given child greed factors `g[]` and cookie sizes `s[]`, assign at most one cookie per child. A child `i` is content if `s[j] >= g[i]`. Maximize the number of content children.

**Key Insight:** Sort both arrays. Use the smallest cookie that satisfies the least-greedy child. If the current smallest cookie cannot satisfy the current least-greedy child, discard that cookie (it cannot satisfy anyone harder, since greed factors are sorted ascending). This is a two-pointer match on sorted arrays.

**Why it works:** Giving a large cookie to a mildly greedy child wastes a resource that could have satisfied a more demanding child later — wait, actually the opposite: giving a large cookie to an easy child wastes it. We want the smallest cookie that is still sufficient for the least-greedy child, which is exactly what this algorithm does.

```cpp
#include <bits/stdc++.h>
using namespace std;

int findContentChildren(vector<int>& g, vector<int>& s) {
    sort(g.begin(), g.end());
    sort(s.begin(), s.end());
    int child = 0, cookie = 0;
    while (child < (int)g.size() && cookie < (int)s.size()) {
        if (s[cookie] >= g[child]) child++;
        cookie++;
    }
    return child;
}

int main() {
    vector<int> g1 = {1, 2, 3}, s1 = {1, 1};
    cout << findContentChildren(g1, s1) << "\n"; // 1
    vector<int> g2 = {1, 2}, s2 = {1, 2, 3};
    cout << findContentChildren(g2, s2) << "\n"; // 2
}
```

**Complexity:**
- Time: O(n log n + m log m) for sorting; O(n + m) for the scan.
- Space: O(1) extra (in-place sort, two pointers).

**Dry Run (g = [1,2,3], s = [1,1]):**
```
Sorted g = [1, 2, 3]
Sorted s = [1, 1]

child=0, cookie=0: s[0]=1 >= g[0]=1  child becomes 1, cookie becomes 1
child=1, cookie=1: s[1]=1 <  g[1]=2  cookie becomes 2
cookie=2 >= s.length=2 -> exit loop

Result: child = 1
```

---

### 2. Gas Station (#134)

**Problem:** There are `n` gas stations in a circle. At station `i`, you can pick up `gas[i]` liters and it costs `cost[i]` to travel to station `i+1`. Find the starting station index to complete the circuit, or return -1 if impossible.

**Key Insight:** Two observations drive the greedy:
1. **Existence check:** If `sum(gas) >= sum(cost)`, a solution always exists (a circular tour with non-negative total gain must have at least one valid starting point).
2. **Start station selection:** If `currentGain` drops below 0 ending at station `i`, then no station from `startStation` through `i` can be the answer — starting from any of those, you would still hit a deficit at `i`. Reset `startStation = i + 1` and `currentGain = 0`.

This single pass simultaneously checks existence and identifies the unique valid start.

```cpp
#include <bits/stdc++.h>
using namespace std;

int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
    int totalGain = 0, currentGain = 0, startStation = 0;
    for (int i = 0; i < (int)gas.size(); i++) {
        int gain = gas[i] - cost[i];
        totalGain += gain;
        currentGain += gain;
        if (currentGain < 0) {
            startStation = i + 1;
            currentGain = 0;
        }
    }
    return totalGain >= 0 ? startStation : -1;
}

int main() {
    vector<int> gas1 = {1, 2, 3, 4, 5}, cost1 = {3, 4, 5, 1, 2};
    cout << canCompleteCircuit(gas1, cost1) << "\n"; // 3
    vector<int> gas2 = {2, 3, 4}, cost2 = {3, 4, 3};
    cout << canCompleteCircuit(gas2, cost2) << "\n"; // -1
}
```

**Complexity:**
- Time: O(n) — single pass.
- Space: O(1).

**Dry Run (gas = [1,2,3,4,5], cost = [3,4,5,1,2]):**
```
gain[] = [-2, -2, -2, 3, 3]

i=0: totalGain=-2, currentGain=-2 < 0  startStation=1, currentGain=0
i=1: totalGain=-4, currentGain=-2 < 0  startStation=2, currentGain=0
i=2: totalGain=-6, currentGain=-2 < 0  startStation=3, currentGain=0
i=3: totalGain=-3, currentGain=3  >= 0  no reset
i=4: totalGain=0,  currentGain=6  >= 0  no reset

totalGain=0 >= 0  return startStation = 3
```

**Why reset works:** If running gain goes negative ending at station `i`, then for any start `j` in `[startStation, i]`, the gain starting from `j` is a sub-run that also ends negative before reaching `i+1`. So the next viable start must be `i+1`.

---

### 3. Non-overlapping Intervals (#435)

**Problem:** Given an array of intervals, return the minimum number of intervals to remove to make the rest non-overlapping.

**Key Insight:** Minimizing removals is equivalent to maximizing the number of intervals we **keep**. By the Activity Selection theorem, we maximize kept intervals by sorting by end time and greedily selecting each compatible interval (one whose start is >= current end). The answer is `total - kept`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[1] < b[1];
    });
    int count = 0, end = INT_MIN;
    for (auto& interval : intervals) {
        if (interval[0] >= end) {
            end = interval[1];
            count++;
        }
    }
    return (int)intervals.size() - count;
}

int main() {
    vector<vector<int>> iv1 = {{1,2},{2,3},{3,4},{1,3}};
    cout << eraseOverlapIntervals(iv1) << "\n"; // 1
    vector<vector<int>> iv2 = {{1,2},{1,2},{1,2}};
    cout << eraseOverlapIntervals(iv2) << "\n"; // 2
}
```

**Complexity:**
- Time: O(n log n) for sort, O(n) for scan.
- Space: O(1) extra.

**Dry Run ([[1,2],[2,3],[3,4],[1,3]]):**
```
Sorted by end: [[1,2],[2,3],[1,3],[3,4]]

end = INT_MIN, count = 0

[1,2]: 1 >= INT_MIN  keep, end=2, count=1
[2,3]: 2 >= 2        keep, end=3, count=2
[1,3]: 1 < 3         skip (overlaps)
[3,4]: 3 >= 3        keep, end=4, count=3

kept=3, total=4  remove = 4 - 3 = 1
```

---

### 4. Partition Labels (#763)

**Problem:** Given a string `s`, partition it into as many parts as possible so that each letter appears in at most one part. Return the sizes of these parts.

**Key Insight:** For each character, record its **last occurrence** index. As we scan left-to-right, maintain a window `[start, end]`. Extend `end` to `max(end, last[s[i]])` at each character. When `i == end`, the window is "closed" — every character in `[start, end]` has its last occurrence within this window, so it is a valid partition.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> partitionLabels(string s) {
    int last[26] = {};
    for (int i = 0; i < (int)s.size(); i++) last[s[i] - 'a'] = i;
    vector<int> result;
    int start = 0, end = 0;
    for (int i = 0; i < (int)s.size(); i++) {
        end = max(end, last[s[i] - 'a']);
        if (i == end) {
            result.push_back(end - start + 1);
            start = i + 1;
        }
    }
    return result;
}

int main() {
    auto r1 = partitionLabels("ababcbacadefegdehijhklij");
    for (auto x : r1) cout << x << " "; cout << "\n"; // 9 7 8
    auto r2 = partitionLabels("eccbbbbdec");
    for (auto x : r2) cout << x << " "; cout << "\n"; // 10
}
```

**Complexity:**
- Time: O(n) — one pass to build `last[]`, one pass to find partitions.
- Space: O(1) — `last[]` array is fixed size 26.

---

#### Detailed Dry Run — Partition Labels: "ababcbacadefegdehijhklij"

**Step 1: Build last-occurrence array**

```
Index: 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23
Char:  a  b  a  b  c  b  a  c  a  d  e  f  e  g  d  e  h  i  j  h  k  l  i  j

last['a'] = 8   (last 'a' at index 8)
last['b'] = 5   (last 'b' at index 5)
last['c'] = 7   (last 'c' at index 7)
last['d'] = 14  (last 'd' at index 14)
last['e'] = 15  (last 'e' at index 15)
last['f'] = 11  (last 'f' at index 11)
last['g'] = 13  (last 'g' at index 13)
last['h'] = 19  (last 'h' at index 19)
last['i'] = 22  (last 'i' at index 22)
last['j'] = 23  (last 'j' at index 23)
last['k'] = 20  (last 'k' at index 20)
last['l'] = 21  (last 'l' at index 21)
```

**Step 2: Scan and build windows**

```
start=0, end=0

i=0  char='a' last=8   end=max(0,8)=8
i=1  char='b' last=5   end=max(8,5)=8
i=2  char='a' last=8   end=max(8,8)=8
i=3  char='b' last=5   end=max(8,5)=8
i=4  char='c' last=7   end=max(8,7)=8
i=5  char='b' last=5   end=max(8,5)=8
i=6  char='a' last=8   end=max(8,8)=8
i=7  char='c' last=7   end=max(8,7)=8
i=8  char='a' last=8   end=max(8,8)=8
     i==end(8==8)  SIZE = 8-0+1 = 9, result=[9], start=9

i=9  char='d' last=14  end=max(8,14)=14
i=10 char='e' last=15  end=max(14,15)=15
i=11 char='f' last=11  end=max(15,11)=15
i=12 char='e' last=15  end=max(15,15)=15
i=13 char='g' last=13  end=max(15,13)=15
i=14 char='d' last=14  end=max(15,14)=15
i=15 char='e' last=15  end=max(15,15)=15
     i==end(15==15)  SIZE = 15-9+1 = 7, result=[9,7], start=16

i=16 char='h' last=19  end=max(15,19)=19
i=17 char='i' last=22  end=max(19,22)=22
i=18 char='j' last=23  end=max(22,23)=23
i=19 char='h' last=19  end=max(23,19)=23
i=20 char='k' last=20  end=max(23,20)=23
i=21 char='l' last=21  end=max(23,21)=23
i=22 char='i' last=22  end=max(23,22)=23
i=23 char='j' last=23  end=max(23,23)=23
     i==end(23==23)  SIZE = 23-16+1 = 8, result=[9,7,8], start=24

Final result: [9, 7, 8]
```

**Verification:**
- Partition 1: indices 0-8  "ababcbaca"  letters {a,b,c} all last occurrences <= 8 ✓
- Partition 2: indices 9-15 "defegde"   letters {d,e,f,g} all last occurrences <= 15 ✓
- Partition 3: indices 16-23 "hijhklij" letters {h,i,j,k,l} all last occurrences <= 23 ✓

---

### 5. Insert Interval (#57)

**Problem:** Given a sorted list of non-overlapping intervals and a `newInterval`, insert it at the correct position, merging any overlapping intervals. Return the updated list.

**Key Insight:** Three phases in a single pass:
1. **Before overlap:** Add all intervals that end strictly before `newInterval` starts.
2. **Overlap zone:** Merge all intervals that overlap with `newInterval` by expanding its bounds.
3. **After overlap:** Add all remaining intervals as-is.

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int> newInterval) {
    vector<vector<int>> result;
    int i = 0, n = (int)intervals.size();

    // Phase 1: intervals ending before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0]) result.push_back(intervals[i++]);

    // Phase 2: overlapping intervals — merge into newInterval
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    result.push_back(newInterval);

    // Phase 3: intervals starting after newInterval ends
    while (i < n) result.push_back(intervals[i++]);

    return result;
}

int main() {
    vector<vector<int>> iv1 = {{1,3},{6,9}};
    auto res1 = insert(iv1, {2,5});
    for (auto& r : res1) cout << "[" << r[0] << "," << r[1] << "]\n";
    // [1,5]  [6,9]

    cout << "---\n";

    vector<vector<int>> iv2 = {{1,2},{3,5},{6,7},{8,10},{12,16}};
    auto res2 = insert(iv2, {4,8});
    for (auto& r : res2) cout << "[" << r[0] << "," << r[1] << "]\n";
    // [1,2]  [3,10]  [12,16]
}
```

**Complexity:**
- Time: O(n) — each interval is visited exactly once.
- Space: O(n) for the result list.

**Dry Run ([[1,2],[3,5],[6,7],[8,10],[12,16]], newInterval=[4,8]):**
```
Phase 1:
  [1,2]: end=2 < 4  add [1,2], i=1
  [3,5]: end=5 >= 4  stop

Phase 2:
  [3,5]:  start=3 <= 8  newInterval=[min(4,3),max(8,5)]=[3,8], i=2
  [6,7]:  start=6 <= 8  newInterval=[min(3,6),max(8,7)]=[3,8], i=3
  [8,10]: start=8 <= 8  newInterval=[min(3,8),max(8,10)]=[3,10], i=4
  [12,16]:start=12 > 8  stop
  add [3,10]

Phase 3: add [12,16]

Result: [[1,2],[3,10],[12,16]]
```

---

### 6. Minimum Number of Arrows to Burst Balloons (#452)

**Problem:** Balloons are represented as intervals `[xstart, xend]`. An arrow at position `x` bursts all balloons with `xstart <= x <= xend`. Find the minimum arrows to burst all balloons.

**Key Insight:** Sort by end coordinate. Shoot at the end of the first balloon. This arrow bursts all subsequent balloons that start before or at that position. When a balloon starts strictly after the current arrow position, fire a new arrow at the new balloon's end.

Note: Unlike Non-overlapping Intervals, touching endpoints ARE hit by the same arrow. Hence the condition for a new arrow is `balloon[0] > arrowPos` (strictly greater), not `>=`.

```cpp
#include <bits/stdc++.h>
using namespace std;

int findMinArrowShots(vector<vector<int>>& points) {
    sort(points.begin(), points.end(), [](const vector<int>& a, const vector<int>& b) {
        return a[1] < b[1];
    });
    int arrows = 1;
    int arrowPos = points[0][1];
    for (auto& balloon : points) {
        if (balloon[0] > arrowPos) {
            arrows++;
            arrowPos = balloon[1];
        }
    }
    return arrows;
}

int main() {
    vector<vector<int>> p1 = {{10,16},{2,8},{1,6},{7,12}};
    cout << findMinArrowShots(p1) << "\n"; // 2
    vector<vector<int>> p2 = {{1,2},{3,4},{5,6},{7,8}};
    cout << findMinArrowShots(p2) << "\n"; // 4
}
```

**Complexity:**
- Time: O(n log n) for sort, O(n) for scan.
- Space: O(1) extra.

**Dry Run ([[10,16],[2,8],[1,6],[7,12]]):**
```
Sorted by end: [[1,6],[2,8],[7,12],[10,16]]

arrows=1, arrowPos=6

[1,6]:   start=1 <= 6   already burst by arrow at 6
[2,8]:   start=2 <= 6   burst by arrow at 6
[7,12]:  start=7 > 6    new arrow, arrows=2, arrowPos=12
[10,16]: start=10 <= 12 burst by arrow at 12

Result: 2 arrows (at positions 6 and 12)
```

**Comparison — Non-overlapping vs Min Arrows:**

| Aspect | Non-overlapping (#435) | Min Arrows (#452) |
|--------|------------------------|-------------------|
| Goal | Maximize kept (min remove) | Min points to pierce all |
| New selection condition | `interval[0] >= end` | `balloon[0] > arrowPos` |
| Touching endpoints | Non-overlapping: [1,2] and [2,3] are compatible | Same arrow hits both |
| Answer | `total - kept` | Direct count |

---

## 5. Summary Table

| Problem | Sort By | Greedy Decision | Answer |
|---------|---------|-----------------|--------|
| Assign Cookies | Both arrays ascending | Give smallest sufficient cookie | Count matched children |
| Gas Station | No sort needed | Restart when gain < 0 | `startStation` if `totalGain >= 0` |
| Non-overlapping Intervals | End time | Keep if `start >= currentEnd` | `total - kept` |
| Partition Labels | No sort (last-occurrence scan) | Close partition when `i == windowEnd` | Partition sizes list |
| Insert Interval | Already sorted input | Merge all overlapping in one pass | Rebuilt result list |
| Min Arrows | End coordinate | Fire new arrow when `start > arrowPos` | Arrow count |

---

> **Last Updated:** 2026-06-26
