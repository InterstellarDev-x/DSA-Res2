# Heap + Frequency

> **Topic:** [Heaps](../README.md) · **Pattern 5 of 5**
> **Problems:** Reorganize String · Task Scheduler

---

## Core Concept

**Heap + Frequency** problems involve scheduling or arranging elements where frequency determines feasibility. A max-heap (by frequency) always picks the most common element first, ensuring we don't get "stuck" with too many of one type.

**Common structure:**
1. Build a frequency map: `Map<T, Integer> freq`
2. Load into max-heap by frequency: `PriorityQueue<int[]> maxHeap` with `(freq, element)`
3. Greedily pick the most frequent; cool down (put back after a delay)

---

## Problem 1: Reorganize String — LC 767

Rearrange so no two adjacent characters are equal. Return `""` if impossible.

```java
public String reorganizeString(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;

    // Max-heap by frequency
    PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a, b) -> b[0] - a[0]);
    for (int i = 0; i < 26; i++) {
        if (freq[i] > 0) maxHeap.offer(new int[]{freq[i], i});
    }

    StringBuilder sb = new StringBuilder();
    while (maxHeap.size() >= 2) {
        int[] first  = maxHeap.poll();   // most frequent
        int[] second = maxHeap.poll();   // second most frequent
        sb.append((char)('a' + first[1]));
        sb.append((char)('a' + second[1]));
        if (--first[0] > 0)  maxHeap.offer(first);
        if (--second[0] > 0) maxHeap.offer(second);
    }

    if (!maxHeap.isEmpty()) {
        int[] last = maxHeap.poll();
        if (last[0] > 1) return "";   // more than 1 left → impossible
        sb.append((char)('a' + last[1]));
    }

    return sb.toString();
}
```

**Impossibility condition:** If any character has frequency > `(n+1)/2`, it's impossible — we can't interleave it with enough other characters.

```java
// Early exit check
int maxFreq = 0;
for (int f : freq) maxFreq = Math.max(maxFreq, f);
if (maxFreq > (s.length() + 1) / 2) return "";
```

**Alternative interleaving approach — O(n):**
```java
public String reorganizeString(String s) {
    int[] freq = new int[26];
    for (char c : s.toCharArray()) freq[c - 'a']++;
    int maxFreq = 0, maxChar = 0;
    for (int i = 0; i < 26; i++) {
        if (freq[i] > maxFreq) { maxFreq = freq[i]; maxChar = i; }
    }
    if (maxFreq > (s.length() + 1) / 2) return "";

    char[] result = new char[s.length()];
    int idx = 0;
    // Place most frequent character at all even positions first
    while (freq[maxChar] > 0) { result[idx] = (char)('a' + maxChar); idx += 2; freq[maxChar]--; }
    // Fill remaining positions with all other characters
    for (int i = 0; i < 26; i++) {
        while (freq[i] > 0) {
            if (idx >= s.length()) idx = 1;  // switch to odd positions
            result[idx] = (char)('a' + i); idx += 2; freq[i]--;
        }
    }
    return new String(result);
}
```

---

## Problem 2: Task Scheduler — LC 621

Schedule CPU tasks with a cooldown of `n` intervals between same tasks. Find minimum time.

**Formula approach — O(26 log 26) = O(1):**

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;
    Arrays.sort(freq);

    int maxFreq = freq[25];
    // Number of tasks with the maximum frequency
    int maxCount = 0;
    for (int f : freq) if (f == maxFreq) maxCount++;

    // Minimum time: either the "frame" calculation or task count (if tasks fill all slots)
    // Frame: (maxFreq - 1) slots × (n + 1) size + maxCount tasks at the end
    int result = (maxFreq - 1) * (n + 1) + maxCount;
    return Math.max(result, tasks.length);
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

```java
public int leastInterval(char[] tasks, int n) {
    int[] freq = new int[26];
    for (char t : tasks) freq[t - 'A']++;

    // Max-heap by frequency
    PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
    for (int f : freq) if (f > 0) maxHeap.offer(f);

    // Queue of (remaining_freq, available_at_time)
    Queue<int[]> cooldown = new ArrayDeque<>();
    int time = 0;

    while (!maxHeap.isEmpty() || !cooldown.isEmpty()) {
        time++;
        if (!maxHeap.isEmpty()) {
            int f = maxHeap.poll() - 1;
            if (f > 0) cooldown.offer(new int[]{f, time + n});
        }
        // Release tasks whose cooldown has expired
        if (!cooldown.isEmpty() && cooldown.peek()[1] == time) {
            maxHeap.offer(cooldown.poll()[0]);
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
