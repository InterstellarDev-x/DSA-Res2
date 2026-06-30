# DSA Knowledge Repository

> Production-quality DSA interview preparation guide — Java-centric, pattern-driven, company-aware.
> Based on [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2).

---

## Table of Contents

1. [How to Use This Repository](#how-to-use-this-repository)
2. [Repository Structure](#repository-structure)
3. [Topic Index](#topic-index)
4. [Company Index](#company-index)
5. [Pattern Index](#pattern-index)

---

## How to Use This Repository

| Goal | Start Here |
|------|-----------|
| Learn a pattern | `Topic/Patterns/` |
| Prepare for a company OA | `Topic/OA-Qns/Company.md` |
| Prepare for interviews | `Topic/Interview Problems/Company.md` |
| Quick revision | `Topic/Interview Tips/` |
| Track recent questions | `Topic/Most Recent Questions/` |

---

## Repository Structure

Every topic follows this exact structure:

```
Topic/
├── Patterns/                        ← One file per pattern
├── Design Data Structure Problems/  ← Implementation-heavy problems
├── OA-Qns/                          ← Company-wise OA questions
├── Interview Problems/              ← Company-wise interview questions
├── Interview Tips/                  ← Strategy, communication, dry-run
└── Most Recent Questions/           ← Yearly trackers (2024, 2025, 2026)
```

---

## Topic Index

| # | Topic | Problems | Status |
|---|-------|----------|--------|
| 1 | [Learn the Basics](./Learn%20the%20Basics/README.md) | 54 | 🔲 |
| 2 | [Sorting Techniques](./Sorting/README.md) | 7 | 🔲 |
| 3 | [Arrays](./1.Arrays/README.md) | 40 | ✅ |
| 4 | [Binary Search](./2.Binary_Search/README.md) | 32 | ✅ |
| 5 | [Strings](./3.Strings/README.md) | 15 | ✅ |
| 6 | [Linked List](./4.Linked_List/README.md) | 31 | ✅ |
| 7 | [Recursion](./5.Recursion/README.md) | 25 | ✅ |
| 8 | [Bit Manipulation](./6.Bit_Manipulation/README.md) | 18 | ✅ |
| 9 | [Stacks & Queues](./7.Stacks_and_Queues/README.md) | 30 | ✅ |
| 10 | [Sliding Window & Two Pointers](./8.Sliding_Window/README.md) | 12 | ✅ |
| 11 | [Heaps](./9.Heaps/README.md) | 17 | ✅ |
| 12 | [Greedy Algorithms](./10.Greedy/README.md) | 15 | ✅ |
| 13 | [Binary Trees](./11.Binary_Trees/README.md) | 38 | ✅ |
| 14 | [Binary Search Trees](./12.BST/README.md) | 16 | ✅ |
| 15 | [Graphs](./13.Graphs/README.md) | 53 | ✅ |
| 16 | [Dynamic Programming](./14.Dynamic_Programming/README.md) | 55 | ✅ |
| 17 | [Tries](./15.Tries/README.md) | 7 | ✅ |
| 18 | [Advanced Strings](./16.Advanced_Strings/README.md) | 9 | ✅ |

---

## Company Index

| Company | OA Questions | Interview Questions |
|---------|-------------|---------------------|
| Amazon | [Arrays OA](./1.Arrays/OA-Qns/Amazon.md) · [LL OA](./4.Linked_List/OA-Qns/Amazon.md) · [S&Q OA](./7.Stacks_and_Queues/OA-Qns/Amazon.md) · [Greedy OA](./10.Greedy/OA-Qns/Amazon.md) · [Trees OA](./11.Binary_Trees/OA-Qns/Amazon.md) · [BST OA](./12.BST/OA-Qns/Amazon.md) · [Graphs OA](./13.Graphs/OA-Qns/Amazon.md) · [DP OA](./14.Dynamic_Programming/OA-Qns/Amazon.md) · [Tries OA](./15.Tries/OA-Qns/Amazon.md) · [Adv Strings OA](./16.Advanced_Strings/OA-Qns/Amazon.md) | [Arrays](./1.Arrays/Interview%20Problems/Amazon.md) · [LL](./4.Linked_List/Interview%20Problems/Amazon.md) · [S&Q](./7.Stacks_and_Queues/Interview%20Problems/Amazon.md) · [Greedy](./10.Greedy/Interview%20Problems/Amazon.md) · [Trees](./11.Binary_Trees/Interview%20Problems/Amazon.md) · [BST](./12.BST/Interview%20Problems/Amazon.md) · [Graphs](./13.Graphs/Interview%20Problems/Amazon.md) · [DP](./14.Dynamic_Programming/Interview%20Problems/Amazon.md) |
| Google | [Arrays OA](./1.Arrays/OA-Qns/Google.md) · [LL OA](./4.Linked_List/OA-Qns/Google.md) · [S&Q OA](./7.Stacks_and_Queues/OA-Qns/Google.md) · [Greedy OA](./10.Greedy/OA-Qns/Google.md) · [Trees OA](./11.Binary_Trees/OA-Qns/Google.md) · [BST OA](./12.BST/OA-Qns/Google.md) · [Graphs OA](./13.Graphs/OA-Qns/Google.md) · [DP OA](./14.Dynamic_Programming/OA-Qns/Google.md) · [Tries OA](./15.Tries/OA-Qns/Google.md) · [Adv Strings OA](./16.Advanced_Strings/OA-Qns/Google.md) | [Arrays](./1.Arrays/Interview%20Problems/Google.md) · [LL](./4.Linked_List/Interview%20Problems/Google.md) · [S&Q](./7.Stacks_and_Queues/Interview%20Problems/Google.md) · [Greedy](./10.Greedy/Interview%20Problems/Google.md) · [Trees](./11.Binary_Trees/Interview%20Problems/Google.md) · [BST](./12.BST/Interview%20Problems/Google.md) · [Graphs](./13.Graphs/Interview%20Problems/Google.md) · [DP](./14.Dynamic_Programming/Interview%20Problems/Google.md) |
| Microsoft | [Arrays OA](./1.Arrays/OA-Qns/Microsoft.md) · [LL OA](./4.Linked_List/OA-Qns/Microsoft.md) · [S&Q OA](./7.Stacks_and_Queues/OA-Qns/Microsoft.md) · [Greedy OA](./10.Greedy/OA-Qns/Microsoft.md) · [Trees OA](./11.Binary_Trees/OA-Qns/Microsoft.md) · [BST OA](./12.BST/OA-Qns/Microsoft.md) · [Graphs OA](./13.Graphs/OA-Qns/Microsoft.md) · [DP OA](./14.Dynamic_Programming/OA-Qns/Microsoft.md) · [Tries OA](./15.Tries/OA-Qns/Microsoft.md) · [Adv Strings OA](./16.Advanced_Strings/OA-Qns/Microsoft.md) | [Arrays](./1.Arrays/Interview%20Problems/Microsoft.md) · [LL](./4.Linked_List/Interview%20Problems/Microsoft.md) · [S&Q](./7.Stacks_and_Queues/Interview%20Problems/Microsoft.md) · [Greedy](./10.Greedy/Interview%20Problems/Microsoft.md) · [Trees](./11.Binary_Trees/Interview%20Problems/Microsoft.md) · [BST](./12.BST/Interview%20Problems/Microsoft.md) · [Graphs](./13.Graphs/Interview%20Problems/Microsoft.md) · [DP](./14.Dynamic_Programming/Interview%20Problems/Microsoft.md) |
| Adobe | [Arrays OA](./1.Arrays/OA-Qns/Adobe.md) · [LL OA](./4.Linked_List/OA-Qns/Adobe.md) · [S&Q OA](./7.Stacks_and_Queues/OA-Qns/Adobe.md) · [Greedy OA](./10.Greedy/OA-Qns/Adobe.md) · [Trees OA](./11.Binary_Trees/OA-Qns/Adobe.md) · [BST OA](./12.BST/OA-Qns/Adobe.md) · [Graphs OA](./13.Graphs/OA-Qns/Adobe.md) · [DP OA](./14.Dynamic_Programming/OA-Qns/Adobe.md) · [Tries OA](./15.Tries/OA-Qns/Adobe.md) · [Adv Strings OA](./16.Advanced_Strings/OA-Qns/Adobe.md) | — |
| Goldman Sachs | [Arrays OA](./1.Arrays/OA-Qns/Goldman%20Sachs.md) · [LL OA](./4.Linked_List/OA-Qns/Goldman%20Sachs.md) · [S&Q OA](./7.Stacks_and_Queues/OA-Qns/Goldman%20Sachs.md) · [Greedy OA](./10.Greedy/OA-Qns/Goldman%20Sachs.md) · [Trees OA](./11.Binary_Trees/OA-Qns/Goldman%20Sachs.md) · [BST OA](./12.BST/OA-Qns/Goldman%20Sachs.md) · [Graphs OA](./13.Graphs/OA-Qns/Goldman%20Sachs.md) · [DP OA](./14.Dynamic_Programming/OA-Qns/Goldman%20Sachs.md) · [Tries OA](./15.Tries/OA-Qns/Goldman%20Sachs.md) · [Adv Strings OA](./16.Advanced_Strings/OA-Qns/Goldman%20Sachs.md) | — |
| Flipkart | [Arrays OA](./1.Arrays/OA-Qns/Flipkart.md) · [LL OA](./4.Linked_List/OA-Qns/Flipkart.md) | — |

---

## Pattern Index

| Pattern | Topics It Appears In |
|---------|----------------------|
| [Prefix Sum](./1.Arrays/Patterns/Prefix%20Sum.md) | Arrays, Strings, Sliding Window |
| [Sliding Window](./1.Arrays/Patterns/Sliding%20Window.md) | Arrays, Strings |
| [Two Pointers](./1.Arrays/Patterns/Two%20Pointers.md) | Arrays, Linked List |
| [Kadane's Algorithm](./1.Arrays/Patterns/Kadane's%20Algorithm.md) | Arrays, DP |
| [Dutch National Flag](./1.Arrays/Patterns/Dutch%20National%20Flag.md) | Arrays |
| [Merge Intervals](./1.Arrays/Patterns/Merge%20Intervals.md) | Arrays, Graphs |
| [Moore's Voting](./1.Arrays/Patterns/Moore's%20Voting.md) | Arrays |
| [Cyclic Sort](./1.Arrays/Patterns/Cyclic%20Sort.md) | Arrays |
| [Fast & Slow Pointer](./4.Linked_List/Patterns/Fast%20and%20Slow%20Pointer.md) | Linked List |
| [Reverse Linked List](./4.Linked_List/Patterns/Reverse%20Linked%20List.md) | Linked List |
| [Merge Linked Lists](./4.Linked_List/Patterns/Merge%20Linked%20Lists.md) | Linked List, Heaps |
| [Cycle Detection (Floyd's)](./4.Linked_List/Patterns/Cycle%20Detection.md) | Linked List |
| [Two Pointers on LL](./4.Linked_List/Patterns/Two%20Pointers%20on%20LL.md) | Linked List |
| [Backtracking](./5.Recursion/Patterns/Backtracking.md) | Recursion, Graphs |
| [Subsets & Combinations](./5.Recursion/Patterns/Subsets%20and%20Combinations.md) | Recursion, DP |
| [Permutations](./5.Recursion/Patterns/Permutations.md) | Recursion |
| [Divide & Conquer](./5.Recursion/Patterns/Divide%20and%20Conquer.md) | Recursion, Sorting, Binary Search |
| [XOR Tricks](./6.Bit_Manipulation/Patterns/XOR%20Tricks.md) | Bit Manipulation, Arrays |
| [Bitmask DP](./6.Bit_Manipulation/Patterns/Bitmask%20DP.md) | Bit Manipulation, DP, Graphs |
| [Binary Trie (Max XOR)](./6.Bit_Manipulation/Patterns/Bit%20Search%20and%20Trie.md) | Bit Manipulation, Tries |
| [Monotonic Stack](./7.Stacks_and_Queues/Patterns/Monotonic%20Stack.md) | Stacks & Queues, Arrays |
| [Monotonic Deque (Sliding Window)](./7.Stacks_and_Queues/Patterns/Queue%20and%20Deque.md) | Stacks & Queues, Sliding Window |
| [Histogram & Rectangle](./7.Stacks_and_Queues/Patterns/Histogram%20and%20Rectangle.md) | Stacks & Queues |
| [Stack for Expressions](./7.Stacks_and_Queues/Patterns/Stack%20for%20Expressions.md) | Stacks & Queues, Strings |
| [Fixed Size Window](./8.Sliding_Window/Patterns/Fixed%20Size%20Window.md) | Sliding Window |
| [Variable Size Window](./8.Sliding_Window/Patterns/Variable%20Size%20Window.md) | Sliding Window, Strings |
| [At Most K → Exactly K](./8.Sliding_Window/Patterns/At%20Most%20K%20Trick.md) | Sliding Window, Arrays |
| [Top K Elements (Heap)](./9.Heaps/Patterns/Top%20K%20Elements.md) | Heaps, Arrays |
| [K-Way Merge](./9.Heaps/Patterns/K%20Way%20Merge.md) | Heaps, Linked List |
| [Two Heaps (Median)](./9.Heaps/Patterns/Two%20Heaps.md) | Heaps |
| [Heap + Frequency](./9.Heaps/Patterns/Heap%20and%20Frequency.md) | Heaps, Strings |
| [Interval Scheduling (Greedy)](./10.Greedy/Patterns/Interval%20Scheduling.md) | Greedy, Arrays |
| [Jump Game (Greedy)](./10.Greedy/Patterns/Jump%20Game.md) | Greedy |
| [Greedy on Strings](./10.Greedy/Patterns/Greedy%20on%20Strings.md) | Greedy, Strings |
| [Greedy with Heap](./10.Greedy/Patterns/Greedy%20with%20Heap.md) | Greedy, Heaps |
| [Tree Traversals](./11.Binary_Trees/Patterns/Tree%20Traversals.md) | Binary Trees |
| [Tree Construction](./11.Binary_Trees/Patterns/Tree%20Construction.md) | Binary Trees |
| [Tree Path Problems](./11.Binary_Trees/Patterns/Path%20Problems.md) | Binary Trees |
| [Level Order (BFS)](./11.Binary_Trees/Patterns/Level%20Order.md) | Binary Trees, Graphs |
| [Tree DP](./11.Binary_Trees/Patterns/Tree%20DP.md) | Binary Trees, DP |
| [BST Operations](./12.BST/Patterns/BST%20Operations.md) | BST |
| [BST Validation & Inorder](./12.BST/Patterns/BST%20Validation%20and%20Inorder.md) | BST |
| [BST Construction](./12.BST/Patterns/BST%20Construction.md) | BST |
| [BST LCA & Ancestors](./12.BST/Patterns/BST%20LCA%20and%20Ancestors.md) | BST, Binary Trees |
| [Graph Traversal (BFS/DFS)](./13.Graphs/Patterns/Graph%20Traversal%20(BFS%20DFS).md) | Graphs |
| [Grid & Matrix BFS](./13.Graphs/Patterns/Grid%20and%20Matrix%20BFS.md) | Graphs, Arrays |
| [Topological Sort](./13.Graphs/Patterns/Topological%20Sort.md) | Graphs |
| [Shortest Path (Dijkstra/Bellman/Floyd)](./13.Graphs/Patterns/Shortest%20Path.md) | Graphs |
| [Minimum Spanning Tree](./13.Graphs/Patterns/Minimum%20Spanning%20Tree.md) | Graphs |
| [Union Find (DSU)](./13.Graphs/Patterns/Union%20Find%20(DSU).md) | Graphs, Arrays |
| [Cycle Detection](./13.Graphs/Patterns/Cycle%20Detection.md) | Graphs |
| [1D DP (Fibonacci Style)](./14.Dynamic_Programming/Patterns/1D%20DP%20(Fibonacci%20Style).md) | DP |
| [2D DP (Grids)](./14.Dynamic_Programming/Patterns/2D%20DP%20(Grids).md) | DP |
| [Knapsack (Subset Sum)](./14.Dynamic_Programming/Patterns/Knapsack%20(Subset%20Sum).md) | DP |
| [Longest Common Subsequence](./14.Dynamic_Programming/Patterns/Longest%20Common%20Subsequence.md) | DP, Strings |
| [Stock Buy Sell](./14.Dynamic_Programming/Patterns/Stock%20Buy%20Sell.md) | DP |
| [Longest Increasing Subsequence](./14.Dynamic_Programming/Patterns/Longest%20Increasing%20Subsequence.md) | DP, Arrays |
| [Partition DP (MCM)](./14.Dynamic_Programming/Patterns/Partition%20DP%20(MCM).md) | DP |
| [DP on Strings](./14.Dynamic_Programming/Patterns/DP%20on%20Strings.md) | DP, Strings |
| [Trie Fundamentals](./15.Tries/Patterns/Trie%20Fundamentals.md) | Tries |
| [Trie Search Variants](./15.Tries/Patterns/Trie%20with%20Search%20Variants.md) | Tries, Strings |
| [Bitwise Trie (XOR)](./15.Tries/Patterns/Bitwise%20Trie%20(XOR).md) | Tries, Bit Manipulation |
| [String Matching (KMP)](./16.Advanced_Strings/Patterns/String%20Matching%20(KMP).md) | Advanced Strings |
| [Z Algorithm & Hashing](./16.Advanced_Strings/Patterns/Z%20Algorithm%20and%20Hashing.md) | Advanced Strings |
| [Palindrome Algorithms (Manacher)](./16.Advanced_Strings/Patterns/Palindrome%20Algorithms.md) | Advanced Strings, DP |

---

> **Last Updated:** 2026-06-26
> **Maintainer:** DSA Repository
> **Source:** [Striver's A2Z DSA Sheet](https://takeuforward.org/strivers-a2z-dsa-course/strivers-a2z-dsa-course-sheet-2)
