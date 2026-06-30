# Step 12 — Greedy Algorithms

← [Heaps](../9.Heaps/README.md) | → [Binary Trees](../BinaryTrees/README.md)

---

## Introduction

A **greedy algorithm** makes the locally optimal choice at each step with the hope that these local decisions lead to a globally optimal solution. Unlike dynamic programming (which explores all subproblems), greedy algorithms commit to a choice and never reconsider it. The key challenge is proving that the greedy choice is safe — that no future choice can benefit from a different decision now.

Greedy algorithms shine when the problem has the **greedy-choice property** (a globally optimal solution can be reached by locally optimal choices) and **optimal substructure** (an optimal solution to the problem contains optimal solutions to subproblems). The most common proof techniques are the **exchange argument** (swapping a non-greedy choice for a greedy one never worsens the solution) and the **greedy stays ahead** argument (the greedy solution is at least as good at every step).

In interview settings, greedy problems fall into recognizable patterns: interval scheduling (sort by end time), jump-game reachability (track a window), greedy invariant tracking on strings/arrays, and greedy combined with a heap for resource allocation.

---

## Problem List

| # | Problem | Difficulty | Pattern | Companies | LC# |
|---|---------|------------|---------|-----------|-----|
| 1 | [Assign Cookies](Patterns/Interval%20Scheduling.md#1-assign-cookies-455) | Easy | Interval Scheduling | Amazon, Adobe | [#455](https://leetcode.com/problems/assign-cookies/) |
| 2 | [Lemonade Change](Patterns/Greedy%20on%20Strings.md#1-lemonade-change-860) | Easy | Greedy on Strings | Amazon | [#860](https://leetcode.com/problems/lemonade-change/) |
| 3 | [Jump Game](Patterns/Jump%20Game.md#1-jump-game-55) | Medium | Jump Game | Microsoft, Amazon | [#55](https://leetcode.com/problems/jump-game/) |
| 4 | [Jump Game II](Patterns/Jump%20Game.md#2-jump-game-ii-45) | Medium | Jump Game | Amazon, Google | [#45](https://leetcode.com/problems/jump-game-ii/) |
| 5 | [Gas Station](Patterns/Interval%20Scheduling.md#2-gas-station-134) | Medium | Interval Scheduling | Amazon, Goldman Sachs | [#134](https://leetcode.com/problems/gas-station/) |
| 6 | [Hand of Straights](Patterns/Greedy%20with%20Heap.md#1-hand-of-straights-846) | Medium | Greedy with Heap | Google | [#846](https://leetcode.com/problems/hand-of-straights/) |
| 7 | [Merge Triplets to Form Target](Patterns/Greedy%20on%20Strings.md#3-merge-triplets-to-form-target-1899) | Medium | Greedy on Strings | Google | [#1899](https://leetcode.com/problems/merge-triplets-to-form-target-triplet/) |
| 8 | [Partition Labels](Patterns/Interval%20Scheduling.md#4-partition-labels-763) | Medium | Interval Scheduling | Google, Adobe | [#763](https://leetcode.com/problems/partition-labels/) |
| 9 | [Valid Parenthesis String](Patterns/Greedy%20on%20Strings.md#2-valid-parenthesis-string-678) | Medium | Greedy on Strings | Amazon, Google | [#678](https://leetcode.com/problems/valid-parenthesis-string/) |
| 10 | [Non-overlapping Intervals](Patterns/Interval%20Scheduling.md#3-non-overlapping-intervals-435) | Medium | Interval Scheduling | Microsoft, Amazon | [#435](https://leetcode.com/problems/non-overlapping-intervals/) |
| 11 | [Meeting Rooms II](Patterns/Greedy%20with%20Heap.md#2-meeting-rooms-ii-253) | Medium | Greedy with Heap | Google, Microsoft | [#253](https://leetcode.com/problems/meeting-rooms-ii/) |
| 12 | [Candy](Patterns/Greedy%20on%20Strings.md#4-candy-135) | Hard | Greedy on Strings | Goldman Sachs | [#135](https://leetcode.com/problems/candy/) |
| 13 | [Insert Interval](Patterns/Interval%20Scheduling.md#5-insert-interval-57) | Medium | Interval Scheduling | Microsoft, Google | [#57](https://leetcode.com/problems/insert-interval/) |
| 14 | [Minimum Number of Arrows to Burst Balloons](Patterns/Interval%20Scheduling.md#6-minimum-number-of-arrows-to-burst-balloons-452) | Medium | Interval Scheduling | Google, Amazon | [#452](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) |
| 15 | [Task Scheduler](Patterns/Greedy%20with%20Heap.md#3-task-scheduler-621) | Medium | Greedy with Heap | Amazon, Google | [#621](https://leetcode.com/problems/task-scheduler/) |

---

## Patterns

| Pattern | Core Idea | Problems | File |
|---------|-----------|----------|------|
| Interval Scheduling | Sort intervals by end time to greedily maximize the number of non-overlapping selections; always pick the interval that ends earliest to leave maximum room for future intervals | #1, #5, #8, #10, #13, #14 | [Interval Scheduling.md](Patterns/Interval%20Scheduling.md) |
| Jump Game | Track a reachability window (`maxReach` or `curEnd`/`farthest`); at each index check if you can extend the window, jump count increments only when you exhaust the current window | #3, #4 | [Jump Game.md](Patterns/Jump%20Game.md) |
| Greedy on Strings | Greedy left-to-right scan maintaining an invariant (counters, balance ranges, or per-character tracking); no backtracking, commit to the optimal local decision immediately | #2, #7, #9, #12 | [Greedy on Strings.md](Patterns/Greedy%20on%20Strings.md) |
| Greedy with Heap | Use a priority queue to always assign the "best available" resource greedily; min-heap tracks earliest-ending active tasks/rooms, TreeMap tracks sorted frequencies | #6, #11, #15 | [Greedy with Heap.md](Patterns/Greedy%20with%20Heap.md) |

---

## Design Problems

| # | Problem | Key Concept | Difficulty |
|---|---------|-------------|------------|
| 1 | Design a task scheduler with cooldown | Frequency array + formula vs. max-heap simulation | Medium |
| 2 | Design a meeting room allocator | Sorted set / min-heap on end times | Medium |
| 3 | Design a jump game validator for variable grids | Generalized reachability window | Medium |

---

## Company Coverage

| Company | Problems |
|---------|----------|
| Amazon | #1, #2, #3, #4, #5, #9, #10, #14, #15 |
| Google | #4, #6, #7, #8, #9, #11, #13, #14, #15 |
| Microsoft | #3, #10, #11, #13 |
| Goldman Sachs | #5, #12 |
| Adobe | #1, #8 |

---

## Subdirectories

- [Patterns/](Patterns/) — Deep dives into each of the 4 greedy patterns with full solutions and proofs
  - [Interval Scheduling.md](Patterns/Interval%20Scheduling.md)
  - [Jump Game.md](Patterns/Jump%20Game.md)
  - [Greedy on Strings.md](Patterns/Greedy%20on%20Strings.md)
  - [Greedy with Heap.md](Patterns/Greedy%20with%20Heap.md)
- [Design Data Structure Problems/](Design%20Data%20Structure%20Problems/) — Greedy-driven design questions
- [OA-Qns/](OA-Qns/) — Online assessment greedy questions from top companies
- [Interview Problems/](Interview%20Problems/) — High-frequency greedy interview problems
- [Interview Tips/](Interview%20Tips/) — How to recognize and argue greedy correctness in interviews
- [Most Recent Questions/](Most%20Recent%20Questions/) — Greedy problems from recent interview rounds

---

> **Last Updated:** 2026-06-26
