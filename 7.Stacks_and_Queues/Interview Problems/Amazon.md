# Amazon — Stacks & Queues Interview Problems

> **Topic:** [Stacks & Queues](../README.md) · **Company:** Amazon
> [← Amazon OA](../OA-Qns/Amazon.md)

---

## Problem 1: Min Stack — Deep Dive

**LC 155** · Medium · O(1) all operations

### The Interview Flow

Amazon interviewers typically start here then escalate:
1. Min Stack → Max Stack → Min-Max Stack
2. Followed by: "What if concurrent threads are pushing/popping?"

### Solution with Key Insight Explanation

```rust
struct MinStack {
    st: Vec<(i32, i32)>, // (val, minAtThisLevel)
}

impl MinStack {
    fn new() -> Self {
        MinStack { st: Vec::new() }
    }

    fn push(&mut self, val: i32) {
        let mn = if self.st.is_empty() { val } else { val.min(self.st.last().unwrap().1) };
        self.st.push((val, mn));
    }

    fn pop(&mut self) { self.st.pop(); }

    fn top(&self) -> i32 { self.st.last().unwrap().0 }

    fn get_min(&self) -> i32 { self.st.last().unwrap().1 }
}
```

**Q: Why not use a separate variable to track global minimum?**
A: A single variable breaks on `pop()`. If we pop the current minimum, we have no way to recover the previous minimum. The pair approach stores the running minimum at every stack level so any `pop()` naturally restores the previous minimum.

**Q: Can you achieve O(1) min with O(1) space?**
A: No — with a stack of N elements each of which could be the minimum, we fundamentally need O(N) information to track all possible minimums. The encoding trick (`val - min` difference) doesn't reduce asymptotic space, just the constant factor.

---

## Problem 2: Largest Rectangle in Histogram — Deep Dive

**LC 84** · Hard · O(n) time, O(n) space

```rust
fn largest_rectangle_area(heights: Vec<i32>) -> i32 {
    let n = heights.len();
    let mut st: Vec<usize> = Vec::new();
    let mut max_area = 0;
    for i in 0..=n {
        let h = if i == n { 0 } else { heights[i] };
        while !st.is_empty() && heights[*st.last().unwrap()] > h {
            let height = heights[st.pop().unwrap()];
            let width = if st.is_empty() { i as i32 } else { (i - st.last().unwrap() - 1) as i32 };
            max_area = max_area.max(height * width);
        }
        st.push(i);
    }
    max_area
}
```

**Q: Walk me through the width calculation.**
A: When we pop index `j` with height `h`, the rectangle of height `h` extends:
- **Right boundary:** `i - 1` (the current index minus 1, since `heights[i] < h`)
- **Left boundary:** `stack.top() + 1` (the new top is the last bar shorter than `h`, so our rectangle starts after it)
- **Width:** `i - stack.top() - 1` = `(i-1) - (stack.top()+1) + 1`
- **If stack is empty:** The rectangle extends all the way to index 0, so width = `i`.

**Q: Why the sentinel 0 at index n?**
A: After processing all bars, some may remain in the stack (never found a shorter bar to their right). The sentinel `height[n]=0` forces all remaining bars to be popped and compute their areas. Without it, we'd need a post-loop flush — sentinel is cleaner.

**Q: Extend this to Maximal Rectangle in binary matrix.**
A: Build a histogram per row: for each cell `(r, c)`, `heights[c]` = consecutive `1`s in column `c` ending at row `r` (reset to 0 if `'0'`). Apply LRH to each row's histogram. Total: O(rows × cols).

---

## Problem 3: Trapping Rain Water — Deep Dive

**LC 42** · Hard

### Two Approaches Compared

```rust
// Approach 1: Two Pointers — O(n) time, O(1) space
fn trap_two_pointers(height: &[i32]) -> i32 {
    let (mut left, mut right) = (0usize, height.len() - 1);
    let (mut max_l, mut max_r, mut water) = (0, 0, 0);
    while left < right {
        if height[left] <= height[right] {
            max_l = max_l.max(height[left]);
            water += max_l - height[left];
            left += 1;
        } else {
            max_r = max_r.max(height[right]);
            water += max_r - height[right];
            right -= 1;
        }
    }
    water
}

// Approach 2: Monotonic Stack — O(n) time, O(n) space
fn trap_stack(height: &[i32]) -> i32 {
    let mut st: Vec<usize> = Vec::new();
    let mut water = 0;
    for i in 0..height.len() {
        while !st.is_empty() && height[*st.last().unwrap()] < height[i] {
            let bottom = st.pop().unwrap();
            if st.is_empty() { break; }
            let width = (i - st.last().unwrap() - 1) as i32;
            let h = height[*st.last().unwrap()].min(height[i]) - height[bottom];
            water += width * h;
        }
        st.push(i);
    }
    water
}
```

**Q: How does the two-pointer approach know which side to process?**
A: Water trapped at position `x` = `min(maxLeft[x], maxRight[x]) - height[x]`. If `height[left] ≤ height[right]`, then `maxRight` from the left's perspective is at least `height[right]`, so `min(maxLeft, maxRight) = maxLeft`. We can safely compute water for `left` without knowing the exact `maxRight`.

**Q: When would you prefer the monotonic stack approach?**
A: When computing in a streaming/online fashion where the full array isn't available up front. The stack naturally processes left-to-right. Also useful when the interviewer asks for variations like "which indices trap water" — easier to extract from stack approach.

---

## Problem 4: 132 Pattern — Amazon Phone Screen

**LC 456** · Medium · O(n) time

```rust
fn find132pattern(nums: &[i32]) -> bool {
    let mut third = i32::MIN; // candidate for nums[k] (132's "2")
    let mut st: Vec<i32> = Vec::new(); // candidates for nums[j] (132's "3")
    for i in (0..nums.len()).rev() {
        if nums[i] < third { return true; }
        while !st.is_empty() && *st.last().unwrap() < nums[i] {
            third = st.pop().unwrap();
        }
        st.push(nums[i]);
    }
    false
}
```

**Q: Explain why we scan right-to-left.**
A: We're building a "j, k" pair as we go. When we pop from the stack (because current element > stack top), the popped value becomes the best `k` candidate (it was pushed as a potential `j` but is now below a new, taller `j`). Then we check if any future `i` (to the left) is smaller than this `k`. Scanning left-to-right would require us to look ahead for `j` and `k` simultaneously.

---

## Amazon LP Alignment

| LP Principle | Connection |
|-------------|-----------|
| Dive Deep | Width formula derivation for histogram; sentinel element explanation |
| Invent and Simplify | Min Stack pair encoding; Score of Parentheses O(1) space |
| Deliver Results | "My O(n) solution is guaranteed single pass — no edge case can make it slower" |
| Are Right, A Lot | Justify two-pointer approach for Trapping Rain Water; when to prefer stack |

---

## Related Files

- [Amazon OA](../OA-Qns/Amazon.md)
- [Google Interview Problems](./Google.md)
- [Histogram & Rectangle Pattern](../Patterns/Histogram%20and%20Rectangle.md)
- [Monotonic Stack Pattern](../Patterns/Monotonic%20Stack.md)

> **Last Updated:** 2026-06-26
