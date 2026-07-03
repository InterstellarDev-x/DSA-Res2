# Heap + Frequency

> **Topic:** [Heaps](../README.md) · **Pattern 5 of 5**
> **Problems:** Reorganize String · Task Scheduler

---

## Core Concept

**Heap + Frequency** problems involve scheduling or arranging elements where frequency determines feasibility. A max-heap (by frequency) always picks the most common element first, ensuring we don't get "stuck" with too many of one type.

**Common structure:**
1. Build a frequency map: `HashMap<T, i32>`
2. Load into max-heap by frequency: `BinaryHeap<(i32, i32)>` with `(freq, element)`
3. Greedily pick the most frequent; cool down (put back after a delay)

---

## Problem 1: Reorganize String — LC 767

Rearrange so no two adjacent characters are equal. Return `""` if impossible.

```rust
use std::collections::BinaryHeap;

fn reorganize_string(s: String) -> String {
    let mut freq = [0i32; 26];
    for c in s.chars() {
        freq[(c as u8 - b'a') as usize] += 1;
    }

    // Max-heap by frequency: (freq, char_index)
    let mut max_heap = BinaryHeap::new();
    for i in 0..26 {
        if freq[i] > 0 {
            max_heap.push((freq[i], i as i32));
        }
    }

    let mut sb = String::new();
    while max_heap.len() >= 2 {
        let (f1, c1) = max_heap.pop().unwrap(); // most frequent
        let (f2, c2) = max_heap.pop().unwrap(); // second most frequent
        sb.push((b'a' + c1 as u8) as char);
        sb.push((b'a' + c2 as u8) as char);
        if f1 - 1 > 0 { max_heap.push((f1 - 1, c1)); }
        if f2 - 1 > 0 { max_heap.push((f2 - 1, c2)); }
    }

    if let Some((f, c)) = max_heap.pop() {
        if f > 1 { return String::new(); } // more than 1 left → impossible
        sb.push((b'a' + c as u8) as char);
    }

    sb
}
```

**Impossibility condition:** If any character has frequency > `(n+1)/2`, it's impossible — we can't interleave it with enough other characters.

```rust
// Early exit check
let max_freq = freq.iter().copied().max().unwrap_or(0);
if max_freq > (s.len() as i32 + 1) / 2 { return String::new(); }
```

**Alternative interleaving approach — O(n):**
```rust
fn reorganize_string(s: String) -> String {
    let mut freq = [0i32; 26];
    for c in s.chars() {
        freq[(c as u8 - b'a') as usize] += 1;
    }
    let mut max_freq = 0i32;
    let mut max_char = 0usize;
    for i in 0..26 {
        if freq[i] > max_freq {
            max_freq = freq[i];
            max_char = i;
        }
    }
    if max_freq > (s.len() as i32 + 1) / 2 { return String::new(); }

    let n = s.len();
    let mut result = vec![b' '; n];
    let mut idx = 0usize;
    // Place most frequent character at all even positions first
    while freq[max_char] > 0 {
        result[idx] = b'a' + max_char as u8;
        idx += 2;
        freq[max_char] -= 1;
    }
    // Fill remaining positions with all other characters
    for i in 0..26 {
        while freq[i] > 0 {
            if idx >= n { idx = 1; } // switch to odd positions
            result[idx] = b'a' + i as u8;
            idx += 2;
            freq[i] -= 1;
        }
    }
    String::from_utf8(result).unwrap()
}
```

---

## Problem 2: Task Scheduler — LC 621

Schedule CPU tasks with a cooldown of `n` intervals between same tasks. Find minimum time.

**Formula approach — O(26 log 26) = O(1):**

```rust
fn least_interval(tasks: &[char], n: i32) -> i32 {
    let mut freq = [0i32; 26];
    for &t in tasks {
        freq[(t as u8 - b'A') as usize] += 1;
    }
    freq.sort();

    let max_freq = freq[25];
    // Number of tasks with the maximum frequency
    let max_count = freq.iter().filter(|&&f| f == max_freq).count() as i32;

    // Minimum time: either the "frame" calculation or task count (if tasks fill all slots)
    // Frame: (maxFreq - 1) slots × (n + 1) size + maxCount tasks at the end
    let result = (max_freq - 1) * (n + 1) + max_count;
    result.max(tasks.len() as i32)
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

```rust
use std::collections::{BinaryHeap, VecDeque};

fn least_interval(tasks: &[char], n: i32) -> i32 {
    let mut freq = [0i32; 26];
    for &t in tasks {
        freq[(t as u8 - b'A') as usize] += 1;
    }

    // Max-heap by frequency
    let mut max_heap = BinaryHeap::new();
    for &f in &freq {
        if f > 0 { max_heap.push(f); }
    }

    // Queue of (remaining_freq, available_at_time)
    let mut cooldown: VecDeque<(i32, i32)> = VecDeque::new();
    let mut time = 0i32;

    while !max_heap.is_empty() || !cooldown.is_empty() {
        time += 1;
        if let Some(f) = max_heap.pop() {
            let f = f - 1;
            if f > 0 { cooldown.push_back((f, time + n)); }
        }
        // Release tasks whose cooldown has expired
        if let Some(&(freq_val, avail)) = cooldown.front() {
            if avail == time {
                max_heap.push(freq_val);
                cooldown.pop_front();
            }
        }
    }
    time
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
