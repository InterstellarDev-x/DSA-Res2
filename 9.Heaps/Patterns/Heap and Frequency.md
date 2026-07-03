# Heap + Frequency

> **Topic:** [Heaps](../README.md) · **Pattern 5 of 5**
> **Problems:** Reorganize String · Task Scheduler

---

## Core Concept

**Heap + Frequency** problems involve scheduling or arranging elements where frequency determines feasibility. A max-heap (by frequency) always picks the most common element first, ensuring we don't get "stuck" with too many of one type.

**Common structure:**
1. Build a frequency map: `unordered_map<T, int> freq`
2. Load into max-heap by frequency: `std::priority_queue<pair<int,int>> maxHeap` with `(freq, element)`
3. Greedily pick the most frequent; cool down (put back after a delay)

---

## Problem 1: Reorganize String — LC 767

Rearrange so no two adjacent characters are equal. Return `""` if impossible.

```cpp
#include <bits/stdc++.h>
using namespace std;

string reorganizeString(string s) {
    int freq[26] = {};
    for (char c : s) freq[c - 'a']++;

    // Max-heap by frequency
    priority_queue<pair<int,int>> maxHeap;
    for (int i = 0; i < 26; i++) {
        if (freq[i] > 0) maxHeap.push({freq[i], i});
    }

    string sb;
    while (maxHeap.size() >= 2) {
        auto [f1, c1] = maxHeap.top(); maxHeap.pop();   // most frequent
        auto [f2, c2] = maxHeap.top(); maxHeap.pop();   // second most frequent
        sb += (char)('a' + c1);
        sb += (char)('a' + c2);
        if (f1 - 1 > 0) maxHeap.push({f1 - 1, c1});
        if (f2 - 1 > 0) maxHeap.push({f2 - 1, c2});
    }

    if (!maxHeap.empty()) {
        auto [f, c] = maxHeap.top(); maxHeap.pop();
        if (f > 1) return "";   // more than 1 left → impossible
        sb += (char)('a' + c);
    }

    return sb;
}
```

**Impossibility condition:** If any character has frequency > `(n+1)/2`, it's impossible — we can't interleave it with enough other characters.

```cpp
// Early exit check
int maxFreq = 0;
for (int f : freq) maxFreq = max(maxFreq, f);
if (maxFreq > (s.length() + 1) / 2) return "";
```

**Alternative interleaving approach — O(n):**
```cpp
#include <bits/stdc++.h>
using namespace std;

string reorganizeString(string s) {
    int freq[26] = {};
    for (char c : s) freq[c - 'a']++;
    int maxFreq = 0, maxChar = 0;
    for (int i = 0; i < 26; i++) {
        if (freq[i] > maxFreq) { maxFreq = freq[i]; maxChar = i; }
    }
    if (maxFreq > ((int)s.length() + 1) / 2) return "";

    string result(s.length(), ' ');
    int idx = 0;
    // Place most frequent character at all even positions first
    while (freq[maxChar] > 0) { result[idx] = (char)('a' + maxChar); idx += 2; freq[maxChar]--; }
    // Fill remaining positions with all other characters
    for (int i = 0; i < 26; i++) {
        while (freq[i] > 0) {
            if (idx >= (int)s.length()) idx = 1;  // switch to odd positions
            result[idx] = (char)('a' + i); idx += 2; freq[i]--;
        }
    }
    return result;
}
```

---

## Problem 2: Task Scheduler — LC 621

Schedule CPU tasks with a cooldown of `n` intervals between same tasks. Find minimum time.

**Formula approach — O(26 log 26) = O(1):**

```cpp
#include <bits/stdc++.h>
using namespace std;

int leastInterval(vector<char>& tasks, int n) {
    int freq[26] = {};
    for (char t : tasks) freq[t - 'A']++;
    sort(freq, freq + 26);

    int maxFreq = freq[25];
    // Number of tasks with the maximum frequency
    int maxCount = 0;
    for (int f : freq) if (f == maxFreq) maxCount++;

    // Minimum time: either the "frame" calculation or task count (if tasks fill all slots)
    // Frame: (maxFreq - 1) slots × (n + 1) size + maxCount tasks at the end
    int result = (maxFreq - 1) * (n + 1) + maxCount;
    return max(result, (int)tasks.size());
}
```

**Frame visualization for tasks=[A,A,A,B,B,C], n=2:**
```
Frame:  A _ _ | A _ _ | A
With B: A B _ | A B _ | A
With C: A B C | A B _ | A   → total = 7
tasks.length = 6 < 7, so answer = 7

If more tasks: A B C | A B C | A B = 9 tasks.length=9 > frame=7 → answer = 9
```

**Heap approach — simulates actual scheduling (more flexible for follow-ups):**

```cpp
#include <bits/stdc++.h>
using namespace std;

int leastInterval(vector<char>& tasks, int n) {
    int freq[26] = {};
    for (char t : tasks) freq[t - 'A']++;

    // Max-heap by frequency
    priority_queue<int> maxHeap;
    for (int f : freq) if (f > 0) maxHeap.push(f);

    // Queue of (remaining_freq, available_at_time)
    queue<pair<int,int>> cooldown;
    int time = 0;

    while (!maxHeap.empty() || !cooldown.empty()) {
        time++;
        if (!maxHeap.empty()) {
            int f = maxHeap.top() - 1; maxHeap.pop();
            if (f > 0) cooldown.push({f, time + n});
        }
        // Release tasks whose cooldown has expired
        if (!cooldown.empty() && cooldown.front().second == time) {
            maxHeap.push(cooldown.front().first);
            cooldown.pop();
        }
    }
    return time;
}
```

**When to use formula vs heap simulation?**
- Formula: faster, O(1) effectively, but only works for this exact problem
- Heap: generalizes to follow-ups like "what if different tasks have different cooldowns?"

---

## Pattern: Greedy + Max-Heap for Scheduling

The invariant across all frequency-heap problems:
1. **Count frequencies** using a map or array
2. **Max-heap by frequency** always gives the "most urgent" task
3. **Cooldown/delay** forces the used task out temporarily
4. **Return to heap** when cooldown expires

| Problem | "Cooldown" Mechanism | Stopping Condition |
|---------|---------------------|-------------------|
| Reorganize String | Pick two alternately each round | Heap empty or 1 left |
| Task Scheduler | After use, unavailable for n slots | All tasks scheduled |
| Rearrange String K Distance Apart (LC 358) | After use, unavailable for k-1 positions | All placed or failure |

---

## Related Files

- [Heap Basics](./Heap%20Basics.md)
- [Top K Elements](./Top%20K%20Elements.md)
- [Coding Tips](../Interview%20Tips/Coding%20Tips.md)
- [Common Mistakes](../Interview%20Tips/Common%20Mistakes.md)

> **Last Updated:** 2026-06-26
