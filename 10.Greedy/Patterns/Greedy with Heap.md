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
| TreeMap (sorted map) for frequency/order needed | No | Yes |

---

## 2. Pattern Recognition Signals

| Signal | Approach |
|--------|----------|
| "Minimum rooms / workers / machines needed" | Sort by start; min-heap of end times |
| "Group consecutive elements into fixed-size groups" | TreeMap to process smallest key first |
| "Schedule tasks with cooldown" | Frequency array + formula, or max-heap simulation |
| "Assign tasks to available workers" | Min-heap of (availability_time) |
| "Process in sorted order while dynamically updating counts" | TreeMap (navigable sorted map) |

---

## 3. Problems

---

### 1. Hand of Straights (#846)

**Problem:** Given an array `hand` of cards and a `groupSize`, determine if you can rearrange all cards into groups of `groupSize` consecutive cards.

**Key Insight:** Use a `TreeMap<Integer, Integer>` to count card frequencies, keeping keys in sorted order. At each step, take the smallest card (`firstKey()`) and try to form a consecutive group of length `groupSize` starting from it. If any required card is missing, return false. Decrement counts (removing key when count reaches 0) and repeat until the map is empty.

**Why TreeMap (not HashMap):** We must always start a new group with the current smallest card. A `HashMap` has no `firstKey()` operation. A `TreeMap` maintains sorted order and provides O(log n) `firstKey()`, `containsKey()`, and `remove()`.

**Why `merge(key, -1, Integer::sum)` instead of `put`:** The `merge` method atomically updates a key — if the result of the function is null, the key is removed. However here we manage removal explicitly by checking `if (count.get(i) == 0) count.remove(i)` for clarity.

```java
import java.util.TreeMap;

public class HandOfStraights {

    public boolean isNStraightHand(int[] hand, int groupSize) {
        if (hand.length % groupSize != 0) return false;
        TreeMap<Integer, Integer> count = new TreeMap<>();
        for (int card : hand) count.merge(card, 1, Integer::sum);
        while (!count.isEmpty()) {
            int first = count.firstKey();
            for (int i = first; i < first + groupSize; i++) {
                if (!count.containsKey(i)) return false;
                count.merge(i, -1, Integer::sum);
                if (count.get(i) == 0) count.remove(i);
            }
        }
        return true;
    }

    public static void main(String[] args) {
        HandOfStraights sol = new HandOfStraights();
        System.out.println(sol.isNStraightHand(new int[]{1,2,3,6,2,3,4,7,8}, 3)); // true
        System.out.println(sol.isNStraightHand(new int[]{1,2,3,4,5}, 4));          // false
    }
}
```

**Complexity:**
- Time: O(n log n) — each card is inserted into the TreeMap once (O(log n)) and processed once (O(log n)). Total O(n log n).
- Space: O(k) where k = number of distinct card values.

**Dry Run (hand=[1,2,3,6,2,3,4,7,8], groupSize=3):**
```
count = {1:1, 2:2, 3:2, 4:1, 6:1, 7:1, 8:1}

Iteration 1: first=1
  i=1: count={1:1}  exists, count.merge(1,-1)  {1:0}  remove  count={2:2,3:2,4:1,6:1,7:1,8:1}
  i=2: count={2:2}  exists, count.merge(2,-1)  {2:1}          count={2:1,3:2,4:1,6:1,7:1,8:1}
  i=3: count={3:2}  exists, count.merge(3,-1)  {3:1}          count={2:1,3:1,4:1,6:1,7:1,8:1}

Iteration 2: first=2
  i=2: remove 2  count={3:1,4:1,6:1,7:1,8:1}
  i=3: remove 3  count={4:1,6:1,7:1,8:1}
  i=4: remove 4  count={6:1,7:1,8:1}

Iteration 3: first=6
  i=6: remove 6  count={7:1,8:1}
  i=7: remove 7  count={8:1}
  i=8: remove 8  count={}

count.isEmpty()  return true
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

```java
import java.util.Arrays;
import java.util.PriorityQueue;

public class MeetingRoomsII {

    public int minMeetingRooms(int[][] intervals) {
        Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
        PriorityQueue<Integer> minHeap = new PriorityQueue<>();
        for (int[] interval : intervals) {
            if (!minHeap.isEmpty() && minHeap.peek() <= interval[0]) minHeap.poll();
            minHeap.offer(interval[1]);
        }
        return minHeap.size();
    }

    public static void main(String[] args) {
        MeetingRoomsII sol = new MeetingRoomsII();
        System.out.println(sol.minMeetingRooms(
            new int[][]{{0,30},{5,10},{15,20}})); // 2
        System.out.println(sol.minMeetingRooms(
            new int[][]{{7,10},{2,4}})); // 1
    }
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
  peek=30, interval[0]=5  30 <= 5? No  no pop (Room A still busy)
  push 10  minHeap=[10, 30]
  rooms in use: {Room A ends at 30, Room B ends at 10}

interval=[15,20]:
  peek=10, interval[0]=15  10 <= 15? Yes  pop 10 (Room B is free)
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

```java
import java.util.Arrays;

public class TaskScheduler {

    public int leastInterval(char[] tasks, int n) {
        int[] freq = new int[26];
        for (char t : tasks) freq[t - 'A']++;
        Arrays.sort(freq);
        int maxFreq = freq[25];
        int maxCount = 0;
        for (int f : freq) if (f == maxFreq) maxCount++;
        return Math.max(tasks.length, (maxFreq - 1) * (n + 1) + maxCount);
    }

    public static void main(String[] args) {
        TaskScheduler sol = new TaskScheduler();
        System.out.println(sol.leastInterval(
            new char[]{'A','A','A','B','B','B'}, 2)); // 8
        System.out.println(sol.leastInterval(
            new char[]{'A','A','A','B','B','B'}, 0)); // 6
        System.out.println(sol.leastInterval(
            new char[]{'A','A','A','A','B','B','B','C','C','C','D','D','D','E','E','E','F','F','F'}, 2)); // 19
    }
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
| Hand of Straights (#846) | O(n log n) | O(k) distinct values | `TreeMap<Integer,Integer>` |
| Meeting Rooms II (#253) | O(n log n) | O(n) worst case | `PriorityQueue<Integer>` (min-heap) |
| Task Scheduler (#621) | O(n) | O(1) | `int[26]` freq array |

---

## 5. Pattern Comparison

| Problem | Heap Role | Key Greedy Decision |
|---------|-----------|---------------------|
| Hand of Straights | TreeMap processes smallest-first; ensures consecutive groups formed left-to-right | Always start a new group from the minimum available card |
| Meeting Rooms II | Min-heap of end times; O(log n) check for earliest-freed room | Reuse room with earliest end if it finishes before new meeting starts |
| Task Scheduler | No heap (formula); heap variant uses max-heap to pick most-frequent task | Always execute the most-frequent available task to minimize idle slots |

---

> **Last Updated:** 2026-06-26
