> **Topic:** [Graphs](../README.md) · **Pattern 3 of 7**

# Topological Sort

A **topological sort** (or topological ordering) of a directed graph is a linear ordering of its vertices such that for every directed edge `u -> v`, vertex `u` comes **before** `v` in the ordering. Intuitively, if `u` must happen before `v`, then `u` appears earlier.

## Core Concept

- A topological order is **only defined for a Directed Acyclic Graph (DAG)**.
- If the graph contains a **cycle**, no topological ordering exists — you can never linearize the dependency `a -> b -> ... -> a`.
- A DAG may have **multiple** valid topological orders. Some problems ask for *any* valid order; others ask whether a *unique* order exists.
- Two standard algorithms produce it:
  - **Kahn's algorithm (BFS)** — repeatedly remove nodes with indegree 0.
  - **DFS-based** — emit nodes in reverse postorder.
- Both run in **O(V + E)** time and **O(V + E)** space.

The single most common use of topo sort is **dependency resolution**: course prerequisites, build systems, task scheduling, package installation order.

> **Cross-link:** Detecting whether a directed graph has a cycle (a prerequisite for "is a topo order even possible") is covered in depth in [Cycle Detection](./Cycle%20Detection.md). Kahn's algorithm gives you cycle detection for free: if you cannot process all `V` nodes, a cycle exists.

---

## Kahn's Algorithm (BFS)

Idea: a node can only appear in the ordering once all of its prerequisites are placed. A node with **indegree 0** has no remaining prerequisites, so it is safe to output. Removing it decrements the indegree of its neighbors, potentially exposing new zero-indegree nodes.

**Cycle check:** count how many nodes you actually pop from the queue. If `processed < V`, some nodes were stuck with indegree > 0 forever — they form (or depend on) a cycle, so no topo order exists.

```rust
use std::collections::VecDeque;

// n nodes labelled 0..n-1, edges given as directed [from, to]
fn topo_sort(n: usize, edges: &Vec<Vec<i32>>) -> Vec<usize> {
    let mut adj: Vec<Vec<usize>> = vec![vec![]; n];
    let mut indegree: Vec<usize> = vec![0; n];

    for e in edges {
        let from = e[0] as usize;
        let to = e[1] as usize;
        adj[from].push(to);
        indegree[to] += 1;
    }

    let mut q: VecDeque<usize> = VecDeque::new();
    for i in 0..n {
        if indegree[i] == 0 {
            q.push_back(i);
        }
    }

    let mut order: Vec<usize> = Vec::new();
    while let Some(node) = q.pop_front() {
        order.push(node);
        for &next in &adj[node] {
            indegree[next] -= 1;
            if indegree[next] == 0 {
                q.push_back(next);
            }
        }
    }

    // If we couldn't place every node, a cycle exists -> no valid order.
    if order.len() != n {
        return vec![];
    }
    order
}
```

---

## DFS-Based Topological Sort

Idea: run DFS. After fully exploring all descendants of a node (postorder), push it onto a stack. The stack, read top-to-bottom, is the topological order. Equivalently, collect nodes in postorder and **reverse** the list.

**Cycle detection via 3-color marking:**
- `0 = WHITE` (unvisited)
- `1 = GRAY` (in the current recursion stack / being processed)
- `2 = BLACK` (fully processed)

If DFS reaches a **GRAY** node, there is a back edge → a cycle → no topo order.

```rust
struct DfsTopoTemplate {
    adj: Vec<Vec<usize>>,
    color: Vec<i32>,   // 0 = white, 1 = gray, 2 = black
    stk: Vec<usize>,
    has_cycle: bool,
}

impl DfsTopoTemplate {
    fn new() -> Self {
        DfsTopoTemplate {
            adj: vec![],
            color: vec![],
            stk: vec![],
            has_cycle: false,
        }
    }

    fn dfs(&mut self, node: usize) {
        self.color[node] = 1; // gray: on the recursion stack
        let neighbors: Vec<usize> = self.adj[node].clone(); // clone to avoid borrow conflict
        for next in neighbors {
            if self.color[next] == 1 {      // back edge -> cycle
                self.has_cycle = true;
                return;
            }
            if self.color[next] == 0 {
                self.dfs(next);
                if self.has_cycle {
                    return;
                }
            }
        }
        self.color[node] = 2;   // black: done
        self.stk.push(node);  // postorder; stack reverses it for us
    }

    fn topo_sort(&mut self, n: usize, edges: &Vec<Vec<i32>>) -> Vec<usize> {
        self.adj = vec![vec![]; n];
        for e in edges {
            self.adj[e[0] as usize].push(e[1] as usize);
        }

        self.color = vec![0; n];
        self.has_cycle = false;

        for i in 0..n {
            if self.color[i] == 0 {
                self.dfs(i);
            }
            if self.has_cycle {
                return vec![];
            }
        }

        let mut order: Vec<usize> = Vec::new();
        while let Some(node) = self.stk.pop() {
            order.push(node);
        }
        order
    }
}
```

---

## Recognition Signals

| Signal in the problem statement | What it points to |
| --- | --- |
| "prerequisite", "must come before", "depends on" | Directed edges; topological sort |
| "find an ordering" / "valid sequence" / "build order" | Kahn's or DFS topo, return the order |
| "is it possible to finish all / schedule all" | Cycle detection via Kahn's (`processed == V`?) |
| "minimum number of semesters / levels / rounds" | Kahn's by **levels** (BFS layers) |
| "unique ordering" / "can be reconstructed" | Kahn's where the queue must hold exactly **one** node at every step |
| "derive order of letters/items from sorted input" | Build graph from adjacent pairs, then topo sort |
| "eventually safe" / "terminal / no infinite loop" | Reverse-graph Kahn's, or DFS 3-color |
| Given a tree/undirected graph, "trim leaves" / "centroid" | Topo-style peeling of degree-1 nodes (Minimum Height Trees) |

---

## 19. Course Schedule — Medium (LC207)

**Idea:** Model courses as nodes and prerequisites as directed edges `prereq -> course`. You can finish all courses iff the graph is a DAG (no cycle). Use Kahn's: if every node gets processed, no cycle.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn can_finish(num_courses: i32, prerequisites: Vec<Vec<i32>>) -> bool {
        let n = num_courses as usize;
        let mut adj: Vec<Vec<usize>> = vec![vec![]; n];
        let mut indegree: Vec<usize> = vec![0; n];

        // prerequisites[i] = [course, prereq]: must take prereq before course
        for p in &prerequisites {
            let course = p[0] as usize;
            let prereq = p[1] as usize;
            adj[prereq].push(course);
            indegree[course] += 1;
        }

        let mut q: VecDeque<usize> = VecDeque::new();
        for i in 0..n {
            if indegree[i] == 0 {
                q.push_back(i);
            }
        }

        let mut processed = 0;
        while let Some(node) = q.pop_front() {
            processed += 1;
            for &next in &adj[node] {
                indegree[next] -= 1;
                if indegree[next] == 0 {
                    q.push_back(next);
                }
            }
        }
        processed == n
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 20. Course Schedule II — Medium (LC210)

**Idea:** Same setup as LC207, but record the order in which nodes are popped. If a cycle prevents processing all nodes, return an empty array.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn find_order(num_courses: i32, prerequisites: Vec<Vec<i32>>) -> Vec<i32> {
        let n = num_courses as usize;
        let mut adj: Vec<Vec<usize>> = vec![vec![]; n];
        let mut indegree: Vec<usize> = vec![0; n];

        for p in &prerequisites {
            let course = p[0] as usize;
            let prereq = p[1] as usize;
            adj[prereq].push(course);
            indegree[course] += 1;
        }

        let mut q: VecDeque<usize> = VecDeque::new();
        for i in 0..n {
            if indegree[i] == 0 {
                q.push_back(i);
            }
        }

        let mut order: Vec<i32> = Vec::new();
        while let Some(node) = q.pop_front() {
            order.push(node as i32);
            for &next in &adj[node] {
                indegree[next] -= 1;
                if indegree[next] == 0 {
                    q.push_back(next);
                }
            }
        }

        if order.len() == n { order } else { vec![] }
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

### Dry Run (Course Schedule II)

Input: `numCourses = 4`, `prerequisites = [[1,0],[2,0],[3,1],[3,2]]`
Meaning: `0 -> 1`, `0 -> 2`, `1 -> 3`, `2 -> 3`.

Initial indegrees: `[0, 1, 1, 2]` (node 0 has none; node 3 depends on 1 and 2).

| Step | Queue (front→back) | Popped | order so far | Indegree updates |
| --- | --- | --- | --- | --- |
| start | `[0]` | — | `[]` | only node 0 has indegree 0 |
| 1 | `[]` | `0` | `[0]` | dec 1→0, dec 2→0; enqueue 1, 2 → `[1,2]` |
| 2 | `[2]` | `1` | `[0,1]` | dec 3 → 1 (not 0 yet) |
| 3 | `[]` | `2` | `[0,1,2]` | dec 3 → 0; enqueue 3 → `[3]` |
| 4 | `[]` | `3` | `[0,1,2,3]` | node 3 has no outgoing edges |

`idx = 4 = numCourses` → no cycle. Result: `[0, 1, 2, 3]` (a valid order; `[0,2,1,3]` would also be valid).

---

## 21. Alien Dictionary — Hard (LC269)

**Idea:** The words are sorted by an unknown alphabet. For each adjacent pair of words, the **first differing character** gives an ordering edge `c1 -> c2`. Topologically sort the characters. Edge case: if word A is longer than word B, A comes after B, yet B is a prefix of A (no differing char), that is invalid → return `""`.

```rust
use std::collections::{HashMap, HashSet, VecDeque};

struct Solution;

impl Solution {
    pub fn alien_order(words: Vec<String>) -> String {
        // Track all characters that appear, and the dependency graph.
        let mut adj: HashMap<char, HashSet<char>> = HashMap::new();
        let mut indegree: HashMap<char, usize> = HashMap::new();

        for w in &words {
            for c in w.chars() {
                adj.entry(c).or_default();
                indegree.entry(c).or_insert(0);
            }
        }

        // Build edges from each adjacent pair of words.
        for i in 0..words.len().saturating_sub(1) {
            let a: Vec<char> = words[i].chars().collect();
            let b: Vec<char> = words[i + 1].chars().collect();
            let min_len = a.len().min(b.len());
            let mut j = 0;
            while j < min_len && a[j] == b[j] {
                j += 1;
            }

            if j < min_len {
                let from = a[j];
                let to = b[j];
                if !adj[&from].contains(&to) {
                    adj.get_mut(&from).unwrap().insert(to);
                    *indegree.entry(to).or_insert(0) += 1;
                }
            } else if a.len() > b.len() {
                // b is a prefix of a but comes first -> invalid ordering
                return String::new();
            }
        }

        let mut q: VecDeque<char> = VecDeque::new();
        for (&c, &deg) in &indegree {
            if deg == 0 {
                q.push_back(c);
            }
        }

        let mut result = String::new();
        while let Some(c) = q.pop_front() {
            result.push(c);
            let neighbors: Vec<char> = adj[&c].iter().cloned().collect();
            for next in neighbors {
                let deg = indegree.get_mut(&next).unwrap();
                *deg -= 1;
                if *deg == 0 {
                    q.push_back(next);
                }
            }
        }

        // If not all characters placed, there was a cycle.
        if result.len() == indegree.len() { result } else { String::new() }
    }
}
```

**Complexity:** Time `O(C)` where `C` is total content length across words (graph has at most 26 nodes and 26*26 edges), Space `O(1)` for the fixed alphabet (or `O(U + edges)` for the distinct chars).

### Dry Run (Alien Dictionary)

Input: `words = ["wrt", "wrf", "er", "ett", "rftt"]`

Compare adjacent pairs to extract the first differing char:
- `"wrt"` vs `"wrf"` → first diff at index 2: `t -> f`
- `"wrf"` vs `"er"` → first diff at index 0: `w -> e`
- `"er"` vs `"ett"` → first diff at index 1: `r -> t`
- `"ett"` vs `"rftt"` → first diff at index 0: `e -> r`

Characters present: `w, r, t, f, e`. Edges: `t->f`, `w->e`, `r->t`, `e->r`.

Indegrees: `w:0, e:1, r:1, t:1, f:1`.

| Step | Queue | Popped | result | Updates |
| --- | --- | --- | --- | --- |
| start | `[w]` | — | `""` | only `w` has indegree 0 |
| 1 | `[]` | `w` | `w` | `e`: 1→0, enqueue → `[e]` |
| 2 | `[]` | `e` | `we` | `r`: 1→0, enqueue → `[r]` |
| 3 | `[]` | `r` | `wer` | `t`: 1→0, enqueue → `[t]` |
| 4 | `[]` | `t` | `wert` | `f`: 1→0, enqueue → `[f]` |
| 5 | `[]` | `f` | `wertf` | no outgoing edges |

All 5 chars placed → result `"wertf"`.

---

## 22. Minimum Height Trees — Medium (LC310)

**Idea:** The roots that minimize tree height are the **centroids** of the tree, and there are at most 2 of them. Find them by **trimming leaves layer by layer** (a topo-style peel on an undirected graph): repeatedly remove all degree-1 nodes. The last 1 or 2 nodes remaining are the answers.

```rust
use std::collections::HashSet;

struct Solution;

impl Solution {
    pub fn find_min_height_trees(n: i32, edges: Vec<Vec<i32>>) -> Vec<i32> {
        let n = n as usize;
        if n == 1 {
            return vec![0];
        }

        let mut adj: Vec<HashSet<usize>> = vec![HashSet::new(); n];
        for e in &edges {
            let u = e[0] as usize;
            let v = e[1] as usize;
            adj[u].insert(v);
            adj[v].insert(u);
        }

        // Initial leaves: degree 1.
        let mut leaves: Vec<usize> = (0..n).filter(|&i| adj[i].len() == 1).collect();

        let mut remaining = n;
        while remaining > 2 {
            remaining -= leaves.len();
            let mut new_leaves: Vec<usize> = Vec::new();
            for &leaf in &leaves {
                // Each leaf has exactly one neighbor; detach it.
                let neighbor = *adj[leaf].iter().next().unwrap();
                adj[neighbor].remove(&leaf);
                if adj[neighbor].len() == 1 {
                    new_leaves.push(neighbor);
                }
            }
            leaves = new_leaves;
        }

        leaves.iter().map(|&x| x as i32).collect() // the 1 or 2 centroids
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 23. Sequence Reconstruction — Medium (LC444)

**Idea:** Check whether `org` is the **unique** topological order consistent with the given subsequences `seqs`. Build a graph from consecutive elements within each sequence, then run Kahn's. Uniqueness holds iff at every step the queue contains **exactly one** node, the produced order matches `org`, and every element of `org` appears.

```rust
use std::collections::{HashMap, VecDeque};

struct Solution;

impl Solution {
    pub fn sequence_reconstruction(org: Vec<i32>, seqs: Vec<Vec<i32>>) -> bool {
        let n = org.len();
        let mut adj: HashMap<i32, Vec<i32>> = HashMap::new();
        let mut indegree: HashMap<i32, i32> = HashMap::new();

        // Register every value that appears in seqs.
        for seq in &seqs {
            for &v in seq {
                adj.entry(v).or_default();
                indegree.entry(v).or_insert(0);
            }
        }

        for seq in &seqs {
            for i in 0..seq.len().saturating_sub(1) {
                let from = seq[i];
                let to = seq[i + 1];
                adj.get_mut(&from).unwrap().push(to);
                *indegree.entry(to).or_insert(0) += 1;
            }
        }

        // Every value must lie in [1, n] and match org's universe.
        if indegree.len() != n {
            return false;
        }

        let mut q: VecDeque<i32> = VecDeque::new();
        for (&v, &deg) in &indegree {
            if deg == 0 {
                q.push_back(v);
            }
        }

        let mut idx = 0;
        while let Some(node) = q.pop_front() {
            if q.len() > 0 {
                return false; // order not unique
            }
            if idx >= n || org[idx] != node {
                return false; // mismatch
            }
            idx += 1;
            let neighbors: Vec<i32> = adj[&node].clone();
            for next in neighbors {
                let deg = indegree.get_mut(&next).unwrap();
                *deg -= 1;
                if *deg == 0 {
                    q.push_back(next);
                }
            }
        }

        idx == n
    }
}
```

**Complexity:** Time `O(V + E)` where `E` is the total length of all sequences, Space `O(V + E)`.

---

## 24. Parallel Courses — Medium (LC1136)

**Idea:** All courses with no remaining prerequisites can be taken **in the same semester (in parallel)**. So run Kahn's **by levels**: each BFS layer is one semester. The number of layers is the minimum number of semesters. If a cycle blocks some courses, return `-1`.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn minimum_semesters(n: i32, relations: Vec<Vec<i32>>) -> i32 {
        let n = n as usize;
        let mut adj: Vec<Vec<usize>> = vec![vec![]; n + 1]; // 1-indexed
        let mut indegree: Vec<usize> = vec![0; n + 1];

        for r in &relations {
            let prev = r[0] as usize;
            let next = r[1] as usize; // prev -> next
            adj[prev].push(next);
            indegree[next] += 1;
        }

        let mut q: VecDeque<usize> = VecDeque::new();
        for i in 1..=n {
            if indegree[i] == 0 {
                q.push_back(i);
            }
        }

        let mut semesters = 0i32;
        let mut studied = 0;
        while !q.is_empty() {
            semesters += 1;
            let size = q.len();
            for _ in 0..size {
                let node = q.pop_front().unwrap();
                studied += 1;
                for &next in &adj[node] {
                    indegree[next] -= 1;
                    if indegree[next] == 0 {
                        q.push_back(next);
                    }
                }
            }
        }

        if studied == n { semesters } else { -1 }
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 25. Find Eventual Safe States — Medium (LC802)

**Idea:** A node is **safe** if every path from it leads to a terminal node (no cycle reachable). Reverse the graph and run Kahn's: terminal nodes (outdegree 0) become **indegree 0 in the reversed graph**. Peeling them off marks all nodes whose every outgoing edge leads only to safe nodes. Return safe nodes sorted.

```rust
use std::collections::VecDeque;

struct Solution;

impl Solution {
    pub fn eventual_safe_nodes(graph: Vec<Vec<i32>>) -> Vec<i32> {
        let n = graph.len();
        let mut rev: Vec<Vec<usize>> = vec![vec![]; n];
        let mut outdegree: Vec<usize> = vec![0; n]; // outdegree in original graph

        for u in 0..n {
            for &v in &graph[u] {
                let v = v as usize;
                rev[v].push(u);  // reversed edge v -> u
                outdegree[u] += 1;
            }
        }

        let mut q: VecDeque<usize> = VecDeque::new();
        for i in 0..n {
            if outdegree[i] == 0 {
                q.push_back(i); // terminal nodes
            }
        }

        let mut safe: Vec<bool> = vec![false; n];
        while let Some(node) = q.pop_front() {
            safe[node] = true;
            for &prev in &rev[node] {
                outdegree[prev] -= 1;
                if outdegree[prev] == 0 {
                    q.push_back(prev);
                }
            }
        }

        (0..n)
            .filter(|&i| safe[i])
            .map(|i| i as i32)
            .collect() // already in ascending order
    }
}
```

**Alternative — DFS 3-color:** mark each node WHITE/GRAY/BLACK; a node is safe iff DFS from it never touches a GRAY node (no back edge / cycle). BLACK = safe.

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

> **Last Updated:** 2026-06-26
