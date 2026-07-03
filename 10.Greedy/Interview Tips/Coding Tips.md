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

**Rust idiom:**
```rust
// Sort by end time (interval scheduling)
intervals.sort_by(|a, b| a[1].cmp(&b[1]));

// Sort by start time (merge intervals)
intervals.sort_by(|a, b| a[0].cmp(&b[0]));

// Sort 2D array by first column ascending, break ties by second column descending
intervals.sort_by(|a, b| {
    if a[0] != b[0] { a[0].cmp(&b[0]) } else { b[1].cmp(&a[1]) }
});

// Sort two arrays together (Assign Cookies)
g.sort();
s.sort();
```

**Never use `a - b` as a comparator** — it overflows for large negative values. Always use explicit comparison lambdas.

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

```rust
// Generic two-pass template
let mut result = vec![1i32; n]; // Initialize minimum

// Pass 1: Left → Right (satisfy left-neighbor constraint)
for i in 1..n {
    if condition(i, i - 1) {
        result[i] = result[i - 1] + 1;
    }
}

// Pass 2: Right → Left (satisfy right-neighbor constraint)
for i in (0..n - 1).rev() {
    if condition(i, i + 1) {
        result[i] = result[i].max(result[i + 1] + 1);
    }
}
```

**Key:** Use `max` in Pass 2 — do NOT overwrite Pass 1 results. Each position must satisfy BOTH passes.

**Why single-pass fails:** Consider [3, 2, 1]. Left-to-right pass gives [1, 1, 1] (each rating is lower → stays at 1). But we need [3, 2, 1] because each person must have fewer candies than their left neighbor (who has a higher rating). The single pass in either direction cannot enforce both constraints simultaneously.

---

## Tip 4: lo/hi Bounds for Range-Tracking Problems (Valid Parenthesis String)

When a character can represent multiple states (like `*` = `(`, `)`, or empty), track the **range of possible valid states**:

```rust
let mut lo: i32 = 0; // Minimum open-paren count (treating '*' as ')' or '')
let mut hi: i32 = 0; // Maximum open-paren count (treating '*' as '(')

for c in s.chars() {
    if c == '('      { lo += 1; hi += 1; }
    else if c == ')' { lo -= 1; hi -= 1; }
    else             { lo -= 1; hi += 1; } // '*' decrements min, increments max

    if hi < 0 { return false; }  // Even optimistically, too many ')'
    lo = lo.max(0);              // lo can't be negative (can't have negative open parens)
}
lo == 0 // Minimum open count must reach 0 (all closed)
```

**Invariant:** `lo ≤ actual_open_count ≤ hi` at every step.
**Valid condition:** `hi >= 0` (not impossible) AND `lo == 0` at end (can be balanced).

---

## Tip 5: BTreeMap for Greedy with Ordering

When greedy requires processing in sorted order AND dynamic insertion/deletion, use `BTreeMap`:

```rust
use std::collections::BTreeMap;
let mut count: BTreeMap<i32, i32> = BTreeMap::new();

// Insert / increment
for &card in &hand {
    *count.entry(card).or_insert(0) += 1;
}

// Process smallest key first (greedy order)
while !count.is_empty() {
    let first = *count.keys().next().unwrap(); // O(log n)
    for i in first..first + group_size {
        if !count.contains_key(&i) { return false; }
        let cnt = count.get_mut(&i).unwrap();
        *cnt -= 1;
        if *cnt == 0 { count.remove(&i); } // Clean up zeros
    }
}
```

**When to use BTreeMap vs BinaryHeap:**
| Feature | BTreeMap | BinaryHeap |
|---------|----------|---------------------|
| Access by value | ✓ `[key]` | ✗ |
| Access min/max | ✓ `.keys().next()`/`.keys().next_back()` | ✓ `peek()` |
| Range queries | ✓ `.range()` | ✗ |
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

```rust
// Pattern A: Two-pointer greedy (Assign Cookies)
let mut i = 0usize;
let mut j = 0usize;
while i < a.len() && j < b.len() {
    if b[j] >= a[i] { i += 1; } // match found
    j += 1;                      // always advance the "resource" pointer
}

// Pattern B: Interval keep/remove (Non-overlapping, Min Arrows)
let mut prev_end = i32::MIN; // or first interval's end
let mut kept = 0;
for interval in &sorted_intervals {
    if interval[0] >= prev_end { // no overlap
        kept += 1;
        prev_end = interval[1];
    }
}

// Pattern C: Window stretching (Partition Labels)
let mut start = 0usize;
let mut end = 0usize;
for i in 0..n {
    end = end.max(last[element(i)]);
    if i == end { record_partition(); start = i + 1; }
}

// Pattern D: BFS-level greedy (Jump Game II)
let mut jumps = 0;
let mut cur_end = 0usize;
let mut farthest = 0usize;
for i in 0..n - 1 {
    farthest = farthest.max(reach(i));
    if i == cur_end { jumps += 1; cur_end = farthest; }
}
```

> **Last Updated:** 2026-06-26
