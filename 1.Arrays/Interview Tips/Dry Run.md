# Dry Run Technique — Arrays

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `dry-run` `tracing` `debugging` `interview`

---

## Table of Contents

1. [Why Dry Run Matters](#why-dry-run-matters)
2. [How to Dry Run](#how-to-dry-run)
3. [Dry Run Examples](#dry-run-examples)
4. [What to Check During Dry Run](#what-to-check-during-dry-run)
5. [Related Files](#related-files)

---

## Why Dry Run Matters

Dry running your code:
1. **Catches off-by-one errors** before the interviewer does
2. **Demonstrates understanding** — you know what each variable represents
3. **Shows methodical thinking** — not just "it looks right"
4. **Builds interviewer confidence** — they see you verify your work

**Rule:** Always dry run on at least one non-trivial example AND one edge case before declaring solution complete.

---

## How to Dry Run

**Format:**

```
State a table of variables. Update each column per iteration.

Iteration | i | l | r | sum | result
----------|---|---|---|-----|-------
start     | - | 0 | 4 |  0  |  0
i=0       | 0 | 0 | 4 |  1  |  0
...
```

**Steps:**
1. Write down all key variables and their initial values
2. Execute the loop body manually, step by step
3. Check the condition at each step
4. Note when variables change
5. Verify the final answer matches expected output

---

## Dry Run Examples

### Example 1: Two Sum (HashMap)

Input: `nums = [2,7,11,15], target = 9`

```
map = {}

i=0: num=2, complement=7, map={} → no hit → map={2:0}
i=1: num=7, complement=2, map={2:0} → HIT! → return [0,1]

Result: [0, 1] ✓
```

### Example 2: Sliding Window (Longest Substring No Repeat)

Input: `s = "abcba"`

```
lastSeen={}, l=0, maxLen=0

r=0: c='a', not in map (or < l) → lastSeen={a:0}, maxLen=max(0,1)=1
r=1: c='b', not in map → lastSeen={a:0,b:1}, maxLen=max(1,2)=2
r=2: c='c', not in map → lastSeen={a:0,b:1,c:2}, maxLen=max(2,3)=3
r=3: c='b', lastSeen[b]=1 >= l=0 → l=2 → lastSeen={a:0,b:3,c:2}, maxLen=max(3,3-2+1)=max(3,2)=3
r=4: c='a', lastSeen[a]=0 < l=2 → no move → lastSeen={a:4,b:3,c:2}, maxLen=max(3,4-2+1)=max(3,3)=3

Result: 3 (substring "abc" or "cba") ✓
```

### Example 3: Dutch National Flag

Input: `nums = [2,0,2,1,1,0]`

```
lo=0, mid=0, hi=5

mid=0: nums[0]=2 → swap(0,5) → [0,0,2,1,1,2], hi=4, mid stays
mid=0: nums[0]=0 → swap(0,0) → [0,0,2,1,1,2], lo=1, mid=1
mid=1: nums[1]=0 → swap(1,1) → [0,0,2,1,1,2], lo=2, mid=2
mid=2: nums[2]=2 → swap(2,4) → [0,0,1,1,2,2], hi=3, mid stays
mid=2: nums[2]=1 → mid=3
mid=3: nums[3]=1 → mid=4
mid=4 > hi=3 → STOP

Result: [0,0,1,1,2,2] ✓
```

### Example 4: Kadane's Algorithm

Input: `nums = [-2,1,-3,4,-1,2,1,-5,4]`

```
maxEndingHere = -2, maxSoFar = -2

i=1: max(1, -2+1=-1) → maxEndingHere=1,  maxSoFar=max(-2,1)=1
i=2: max(-3,1-3=-2)  → maxEndingHere=-2, maxSoFar=max(1,-2)=1
i=3: max(4,-2+4=2)   → maxEndingHere=4,  maxSoFar=max(1,4)=4
i=4: max(-1,4-1=3)   → maxEndingHere=3,  maxSoFar=max(4,3)=4
i=5: max(2,3+2=5)    → maxEndingHere=5,  maxSoFar=max(4,5)=5
i=6: max(1,5+1=6)    → maxEndingHere=6,  maxSoFar=max(5,6)=6
i=7: max(-5,6-5=1)   → maxEndingHere=1,  maxSoFar=max(6,1)=6
i=8: max(4,1+4=5)    → maxEndingHere=5,  maxSoFar=max(6,5)=6

Result: 6 (subarray [4,-1,2,1]) ✓
```

---

## What to Check During Dry Run

```
□ Does the first iteration work?
□ Does the last iteration work (boundary)?
□ Are the loop exit conditions correct?
□ Are pointers advancing correctly?
□ Is the result updated in the right place?
□ Edge case: single element — does the loop run 0 times correctly?
□ Edge case: all same — does de-duplication work?
```

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Communication Tips](./Communication.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
