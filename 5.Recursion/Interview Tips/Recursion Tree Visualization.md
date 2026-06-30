# Recursion Tree Visualization

> **Topic:** [Recursion](../README.md) · **Section:** Interview Tips
> **Last Updated:** 2026-06-26

---

## Why Draw the Recursion Tree?

Recursion trees serve three purposes in interviews:
1. **Prove correctness** — show all cases are covered
2. **Derive complexity** — count nodes and work per node
3. **Communicate** — show the interviewer you can think structurally

Draw the tree before coding. The code is just the tree, mechanically written.

---

## Subsets of [1, 2, 3]

```
                      []
            /          |          \        (omit)
          [1]         [2]         [3]
        /    \       /    \         \
     [1,2]  [1,3]  [2,3]  []      []
      |
   [1,2,3]
```

Leaves (including internal nodes): 2^3 = 8 subsets. Code adds at every node.

---

## Combination Sum [2,3,6,7], target=7

```
start=0, rem=7
├── pick 2 → rem=5
│   ├── pick 2 → rem=3
│   │   ├── pick 2 → rem=1
│   │   │   └── pick 2 → rem=-1 (prune)
│   │   └── pick 3 → rem=0 ✅ [2,2,3]
│   └── pick 3 → rem=2
│       └── pick 3 → rem=-1 (prune)
├── pick 3 → rem=4
│   └── pick 3 → rem=1
│       └── pick 3 → rem=-2 (prune)
├── pick 6 → rem=1
│   └── prune (all remaining ≥ 6)
└── pick 7 → rem=0 ✅ [7]
```

---

## Generate Parentheses n=2

```
open=0, close=0, sb=""
├── add '(' → open=1
│   ├── add '(' → open=2
│   │   └── add ')' → close=1
│   │       └── add ')' → close=2 → "(())" ✅
│   └── add ')' → close=1
│       └── add '(' → open=2
│           └── add ')' → close=2 → "()()" ✅
```

Tree has 2 leaves for n=2 (Catalan(2) = 2). Each internal node: ≤2 children.

---

## N-Queens n=4

```
Row 0: try col 0, 1, 2, 3
  Col 0 placed:
    Row 1: try col 0(×), 1(×), 2(✓), 3(✓)
      Col 2 placed:
        Row 2: no valid col → backtrack
      Col 3 placed:
        Row 2: col 1 valid
          Row 3: col 3(×), 0(×), ... → backtrack
  Col 1 placed:
    Row 1: col 3 valid
      Row 2: col 0 valid
        Row 3: col 2 valid → [1,3,0,2] ✅
```

---

## Tree Depth vs Stack Depth

```
Problem                | Tree Height | Stack Depth
-----------------------|-------------|-------------
Subsets of n elements  | n           | O(n)
Combination Sum        | target/min  | O(target/min)
Permutations           | n           | O(n)
N-Queens               | n (rows)    | O(n)
Merge Sort             | log n       | O(log n)
Fibonacci (naive)      | n           | O(n)
Pow(x, n)              | log n       | O(log n)
```

**Key:** Stack depth = tree height (longest path from root to leaf).

---

## Counting Leaves = Counting Solutions

| Problem | Number of Leaves |
|---------|-----------------|
| Subsets of n | 2^n |
| Permutations of n | n! |
| Permutations of n with k duplicates | n! / k! |
| Generate Parentheses n | Catalan(n) = C(2n,n)/(n+1) |
| N-Queens n | varies (92 for n=8) |

---

## How to Draw in an Interview

1. Root = initial call with full input
2. Each edge = one choice made
3. Each node = current state (remaining input, current path)
4. Cross out pruned branches immediately
5. Box valid solutions (leaf nodes that meet goal)

Keep it high-level — don't draw every node for large n. Draw the first 2 levels fully, then say "this pattern continues recursively."

---

## Related Files

- [Coding Tips](./Coding%20Tips.md)
- [Common Mistakes](./Common%20Mistakes.md)
- [Complexity Analysis](./Complexity%20Analysis.md)

> **Last Updated:** 2026-06-26
