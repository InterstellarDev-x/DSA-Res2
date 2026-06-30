# Communication Tips — Arrays Interviews

> **Topic:** [Arrays](../README.md) · **Section:** Interview Tips
> **Tags:** `communication` `interview-tips` `soft-skills` `FAANG`

---

## Table of Contents

1. [The Interview Framework](#the-interview-framework)
2. [What to Say at Each Stage](#what-to-say-at-each-stage)
3. [Think-Aloud Templates](#think-aloud-templates)
4. [When You're Stuck](#when-youre-stuck)
5. [Red Flags for Interviewers](#red-flags-for-interviewers)
6. [Green Flags for Interviewers](#green-flags-for-interviewers)
7. [Related Files](#related-files)

---

## The Interview Framework

```
1. UNDERSTAND  (2–3 min)  → Clarify, restate, examples
2. PLAN        (3–5 min)  → Brute force → optimal, state complexity
3. CODE        (10–15 min)→ Talk while writing, modular
4. TEST        (3–5 min)  → Walk through examples, edge cases
5. OPTIMIZE    (2–3 min)  → Discuss improvements if any
```

---

## What to Say at Each Stage

### Stage 1: Understanding

> "Let me make sure I understand the problem. We have an array of integers and we want to find... Is the array sorted? Can values be negative? Can the array be empty? What should I return if no answer exists?"

> "Let me trace through this example: with input `[1,2,3]` and `k=3`, the output should be `2` because... Does that match?"

### Stage 2: Planning

> "My first instinct is a brute force: check every pair in O(n²). But we can do better. If I use a prefix sum with a hashmap, I can reduce this to O(n) time and O(n) space. Here's the idea..."

> "I'm going to sort the array first — that costs O(n log n). Then I'll use two pointers to scan in O(n). Overall O(n log n)."

### Stage 3: Coding

> "I'm declaring `l` and `r` as my two pointers at the start and end of the array..."

> "At each step, if the sum is too small I move `l` right, if too large I move `r` left..."

> "I'm initializing `maxSoFar` to `nums[0]`, not 0, because if all values are negative, the answer could be negative..."

### Stage 4: Testing

> "Let me test with the given example first: `[2,3,-2,4]`..."
> "Now let me check edge cases: empty array — I should return... single element — the loop doesn't execute, so we return `nums[0]`..."
> "All negative: `[-3,-1,-2]` — we'd return `-1`, which is correct."

### Stage 5: Optimizing

> "The current solution is O(n) time and O(n) space. If the interviewer requires O(1) space, we could use the array itself as the prefix sum storage by... though that mutates the input."

---

## Think-Aloud Templates

### Pattern Recognition

> "This looks like a sliding window problem because we need the longest contiguous subarray with a constraint, and all values are positive."

> "I see 'subarray sum equals K' — that's a classic prefix sum + hashmap pattern."

> "The array has three distinct values — Dutch National Flag gives us O(n) single-pass."

### Trade-off Discussion

> "I have two approaches: HashMap gives O(n) time but O(n) space. Sorting gives O(n log n) time but O(1) space. Given the constraints, I'll go with... unless space is more constrained."

### Complexity Explanation

> "This is O(n) time. Even though there's a while loop inside the for loop, `l` only ever moves forward — it advances at most n times total, so the combined inner+outer loops do at most 2n iterations."

---

## When You're Stuck

**Don't freeze. Say one of:**

1. > "Let me think about this from a different angle. What if I... "
2. > "I'm trying to figure out the invariant here. What's always true at each step?"
3. > "Can I ask a clarifying question? Is the input guaranteed to be sorted?"
4. > "I know a brute force that works — let me code that and then optimize."
5. > "I've seen a similar structure in [Kadane's / Prefix Sum / Sliding Window] — let me think if that applies here."

**Never:**
- Go silent for 3+ minutes
- Say "I don't know" and stop
- Start randomly writing code without a plan

---

## Red Flags for Interviewers

| Red Flag | What It Signals |
|----------|----------------|
| Codes without speaking | Can't communicate — problem for team collaboration |
| Jumps to code before understanding | Doesn't handle ambiguity |
| Only knows the "trick" answer | Can't derive solutions organically |
| Can't explain their own code | Memorized, not understood |
| Dismisses brute force | Misses partial credit; may jump to wrong optimal |
| Never asks about edge cases | Shallow problem modeling |
| Gets defensive when corrected | Won't take feedback in team settings |

---

## Green Flags for Interviewers

| Green Flag | What It Signals |
|------------|----------------|
| Asks clarifying questions | Thorough problem modeling |
| States brute force before optimal | Strong fundamentals |
| Explains WHY, not just WHAT | Deep understanding |
| Finds bugs in own code | Self-correcting, mature engineer |
| Proactively states complexity | Aware of performance |
| Discusses trade-offs | Systems thinking |
| Handles interviewer hints gracefully | Coachable |

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Follow-up Questions](./Follow-up%20Questions.md)
- [Dry Run Technique](./Dry%20Run.md)
- [Arrays README](../README.md)

> **Last Updated:** 2026-06-26
