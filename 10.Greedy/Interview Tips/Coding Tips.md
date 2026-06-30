> **Topic:** [Greedy Algorithms](../README.md) · **Interview Tips — Coding**

# Greedy Algorithms — Coding Tips

## Tip 1: Sort-First Discipline

The most important habit in greedy: **sort before you greedily scan**.

| Problem Type | Sort Key | Why |
|-------------|----------|-----|
| Maximize non-overlapping intervals | End time ascending | Earliest-ending interval leaves maximum future room |
| Merge / insert intervals | Start time ascending | Process intervals left to right |
| Assign resources (cookies, tasks) | Both arrays ascending | Match smallest sufficient resource to smallest need |
| Meeting Rooms II | Start time ascending | Process meetings in chronological order |
| Minimum Arrows | End time ascending | Arrow at end of first balloon pops maximum overlapping |

**Java idiom:**
```java
// Sort by end time (interval scheduling)
Arrays.sort(intervals, Comparator.comparingInt(a -> a[1]));

// Sort by start time (merge intervals)
Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));

// Sort 2D array by first column ascending, break ties by second column descending
Arrays.sort(intervals, (a, b) -> a[0] != b[0] 
    ? Integer.compare(a[0], b[0]) 
    : Integer.compare(b[1], a[1]));

// Sort two arrays together (Assign Cookies)
Arrays.sort(g);
Arrays.sort(s);
```

**Never use `a - b` as a comparator** — it overflows for large negative values. Always use `Integer.compare(a, b)`.

---

## Tip 2: Exchange Argument Sketch Template

When asked "why does greedy work?", use this template:

```
1. Assume optimal solution O does NOT use our greedy choice G.
2. Identify where O first diverges from greedy.
3. Show we can swap O's choice with G (or swap two adjacent elements in O's ordering).
4. Show the swapped solution is at least as good as O (or strictly better).
5. By induction / repeated swapping, conclude greedy = optimal.
```

**Example for Non-overlapping Intervals:**
> "Assume optimal keeps interval X (ends at time t₂) over Y (ends at time t₁ < t₂). Both start after the previous interval. Swap X with Y: Y ends earlier, creating more room for future intervals. The swap cannot make things worse. By induction, always picking the earliest-ending interval is optimal."

---

## Tip 3: Two-Pass Pattern (Candy / Ratings Problems)

When a constraint depends on **both neighbors**, a single pass is insufficient. Use two passes:

```java
// Generic two-pass template
int[] result = new int[n];
Arrays.fill(result, 1); // Initialize minimum

// Pass 1: Left → Right (satisfy left-neighbor constraint)
for (int i = 1; i < n; i++) {
    if (condition(i, i - 1)) {
        result[i] = result[i - 1] + 1;
    }
}

// Pass 2: Right → Left (satisfy right-neighbor constraint)
for (int i = n - 2; i >= 0; i--) {
    if (condition(i, i + 1)) {
        result[i] = Math.max(result[i], result[i + 1] + 1);
    }
}
```

**Key:** Use `Math.max` in Pass 2 — do NOT overwrite Pass 1 results. Each position must satisfy BOTH passes.

**Why single-pass fails:** Consider [3, 2, 1]. Left-to-right pass gives [1, 1, 1] (each rating is lower → stays at 1). But we need [3, 2, 1] because each person must have fewer candies than their left neighbor (who has a higher rating). The single pass in either direction cannot enforce both constraints simultaneously.

---

## Tip 4: lo/hi Bounds for Range-Tracking Problems (Valid Parenthesis String)

When a character can represent multiple states (like `*` = `(`, `)`, or empty), track the **range of possible valid states**:

```java
int lo = 0; // Minimum open-paren count (treating '*' as ')' or '')
int hi = 0; // Maximum open-paren count (treating '*' as '(')

for (char c : s.toCharArray()) {
    if (c == '(')      { lo++; hi++; }
    else if (c == ')') { lo--; hi--; }
    else               { lo--; hi++; } // '*' decrements min, increments max

    if (hi < 0) return false;  // Even optimistically, too many ')'
    lo = Math.max(lo, 0);      // lo can't be negative (can't have negative open parens)
}
return lo == 0; // Minimum open count must reach 0 (all closed)
```

**Invariant:** `lo ≤ actual_open_count ≤ hi` at every step.
**Valid condition:** `hi >= 0` (not impossible) AND `lo == 0` at end (can be balanced).

---

## Tip 5: TreeMap for Greedy with Ordering

When greedy requires processing in sorted order AND dynamic insertion/deletion, use `TreeMap`:

```java
TreeMap<Integer, Integer> count = new TreeMap<>();

// Insert / increment
for (int card : hand) count.merge(card, 1, Integer::sum);

// Process smallest key first (greedy order)
while (!count.isEmpty()) {
    int first = count.firstKey(); // O(log n)
    for (int i = first; i < first + groupSize; i++) {
        if (!count.containsKey(i)) return false;
        count.merge(i, -1, Integer::sum);
        if (count.get(i) == 0) count.remove(i); // Clean up zeros
    }
}
```

**When to use TreeMap vs PriorityQueue:**
| Feature | TreeMap | PriorityQueue |
|---------|---------|---------------|
| Access by value | ✓ `get(key)` | ✗ |
| Access min/max | ✓ `firstKey()`/`lastKey()` | ✓ `peek()` |
| Range queries | ✓ `subMap()` | ✗ |
| Duplicates | Via value field | ✓ naturally |
| Use when | Need both lookup + ordering | Need min/max only |

---

## Tip 6: Greedy Template Checklist

Before writing greedy code, answer these questions:

- [ ] What is the greedy choice at each step?
- [ ] What order should elements be processed? (What to sort by?)
- [ ] What state do I need to track between steps?
- [ ] What is the termination / success condition?
- [ ] Can I prove the greedy choice never invalidates future optimal choices?

---

## Tip 7: Common Greedy Code Patterns

```java
// Pattern A: Two-pointer greedy (Assign Cookies)
int i = 0, j = 0;
while (i < a.length && j < b.length) {
    if (b[j] >= a[i]) i++; // match found
    j++;                    // always advance the "resource" pointer
}

// Pattern B: Interval keep/remove (Non-overlapping, Min Arrows)
int prevEnd = Integer.MIN_VALUE; // or first interval's end
int kept = 0;
for (int[] interval : sortedIntervals) {
    if (interval[0] >= prevEnd) { // no overlap
        kept++;
        prevEnd = interval[1];
    }
}

// Pattern C: Window stretching (Partition Labels)
int start = 0, end = 0;
for (int i = 0; i < n; i++) {
    end = Math.max(end, last[element(i)]);
    if (i == end) { recordPartition(); start = i + 1; }
}

// Pattern D: BFS-level greedy (Jump Game II)
int jumps = 0, curEnd = 0, farthest = 0;
for (int i = 0; i < n - 1; i++) {
    farthest = Math.max(farthest, reach(i));
    if (i == curEnd) { jumps++; curEnd = farthest; }
}
```

> **Last Updated:** 2026-06-26
