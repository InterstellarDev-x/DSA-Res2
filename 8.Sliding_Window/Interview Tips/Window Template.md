# Window Template Guide

> **Topic:** [Sliding Window](../README.md) · **Section:** Interview Tips

---

## The Universal Sliding Window Template

```rust
// Step 1: Choose state representation
//   - i32 counter for simple counts (zeros, odds)
//   - [i32; N] freq array for character frequencies
//   - HashMap<K,V> for arbitrary key frequencies/counts

// Step 2: Choose goal (maximize or minimize window)
//   - Maximize: use 'if' for shrink, update answer after shrink
//   - Minimize: use 'while' for shrink, update answer before shrink

// Step 3: Choose what to track
//   - Window size (right - left + 1)
//   - Count of valid windows (result += right - left + 1)

let mut left = 0usize;
// state variables here

for right in 0..n {
    // [EXPAND] Add nums[right] to window state

    // [SHRINK] While/if window is invalid/should shrink
    while is_invalid() {  // or 'if' for max-length problems
        // Remove nums[left] from window state
        left += 1;
    }

    // [RECORD] Update answer
    // For max length: max_len = max_len.max(right - left + 1)
    // For min length: min_len = min_len.min(right - left + 1) [inside while]
    // For count: result += right - left + 1
}
```

---

## Template Instantiations

### Template 1: Max Length — At Most k of Property

**Problems:** Max Consecutive Ones III, Fruits Into Baskets, LSWORC

```rust
let mut left = 0usize;
let mut state = 0i32;
let mut max_len = 0usize;
for right in 0..n {
    state += increase(nums[right]);            // expand
    if state > k {                             // violated → if, not while
        state -= decrease(nums[left]);         // shrink by 1
        left += 1;
    }
    max_len = max_len.max(right - left + 1);  // window is valid
}
max_len
```

### Template 2: Min Length — At Least Target Sum/Count

**Problems:** Minimum Subarray Sum

```rust
let mut left = 0usize;
let mut state = 0i32;
let mut min_len = usize::MAX;
for right in 0..n {
    state += nums[right];                      // expand
    while state >= target {                    // valid → while, minimize
        min_len = min_len.min(right - left + 1);
        state -= nums[left];
        left += 1;
    }
}
if min_len == usize::MAX { 0 } else { min_len }
```

### Template 3: Min Length — All Chars Present

**Problems:** Minimum Window Substring

```rust
let mut need = [0i32; 128];
// populate need[] from t
let mut left = 0usize;
let mut have = 0i32;
let required = t.len() as i32;
let mut min_len = usize::MAX;
let mut min_left = 0usize;

let s_bytes = s.as_bytes();
for right in 0..n {
    let rc = s_bytes[right] as usize;
    if need[rc] > 0 { have += 1; }
    need[rc] -= 1;                              // expand with condition
    while have == required {                    // valid → while, minimize
        if right - left + 1 < min_len {
            min_len = right - left + 1;
            min_left = left;
        }
        let lc = s_bytes[left] as usize;
        need[lc] += 1;
        if need[lc] > 0 { have -= 1; }         // shrink with condition
        left += 1;
    }
}
if min_len == usize::MAX { "".to_string() } else { s[min_left..min_left + min_len].to_string() }
```

### Template 4: Count Subarrays — At Most k

**Problems:** Subarray Product < K, Binary Subarrays With Sum (combined with subtraction)

```rust
let mut left = 0usize;
let mut state = initial_state;
let mut result = 0usize;
for right in 0..n {
    state = apply_right(state, nums[right]);   // expand
    while state > k || is_invalid(state) {    // shrink until valid
        state = remove_left(state, nums[left]);
        left += 1;
    }
    result += right - left + 1;               // count all starting positions
}
result
```

---

## Decision Flowchart

```
Problem says "subarray/substring"?
    YES →
        Fixed k elements?
            YES → Fixed Window Template
            NO  →
                Count valid subarrays?
                    YES → atMost template + subtraction if "exactly k"
                    NO  →
                        Minimize window?
                            YES → Variable Window, while-shrink, update inside while
                            NO  → Variable Window, if-shrink, update after if
        NO → Two Pointers (if sorted) or different approach
```

---

## State Variable Selection

| Window Property | State Variable | Init | Expand | Shrink | Violated When |
|----------------|---------------|------|--------|--------|---------------|
| Running sum | `i32 sum` | 0 | `sum += x` | `sum -= x` | `sum >= target` (for minimize) |
| Zero count | `i32 zeros` | 0 | `if x==0: zeros+=1` | `if x==0: zeros-=1` | `zeros > k` |
| Odd count | `i32 odds` | 0 | `if x%2==1: odds+=1` | `if x%2==1: odds-=1` | `odds > k` |
| Running product | `i32 product` | 1 | `product *= x` | `product /= x` | `product >= k` |
| Distinct count | `HashMap + len` | empty | `put/merge x` | `remove if 0` | `map.len() > k` |
| Char frequency | `[i32; N] freq` | zeros | `freq[x]+=1` | `freq[x]-=1` | `freq[x] > 1` or `freq[x] > allowed` |
| Have/need | `i32 have` | 0 | conditional `have+=1` | conditional `have-=1` | `have < required` |

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)
- [Variable Size Window](../Patterns/Variable%20Size%20Window.md)

> **Last Updated:** 2026-06-26
