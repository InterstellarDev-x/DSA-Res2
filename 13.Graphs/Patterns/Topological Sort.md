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

```cpp
#include <bits/stdc++.h>
using namespace std;

// n nodes labelled 0..n-1, edges given as directed [from, to]
vector<int> topoSort(int n, vector<vector<int>>& edges) {
    vector<vector<int>> adj(n);
    vector<int> indegree(n, 0);

    for (auto& e : edges) {
        int from = e[0], to = e[1];
        adj[from].push_back(to);
        indegree[to]++;
    }

    queue<int> q;
    for (int i = 0; i < n; i++) {
        if (indegree[i] == 0) q.push(i);
    }

    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int next : adj[node]) {
            if (--indegree[next] == 0) q.push(next);
        }
    }

    // If we couldn't place every node, a cycle exists -> no valid order.
    if ((int)order.size() != n) return {};
    return order;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class DfsTopoTemplate {
    vector<vector<int>> adj;
    vector<int> color;   // 0 = white, 1 = gray, 2 = black
    stack<int> stk;
    bool hasCycle;

    void dfs(int node) {
        color[node] = 1; // gray: on the recursion stack
        for (int next : adj[node]) {
            if (color[next] == 1) {      // back edge -> cycle
                hasCycle = true;
                return;
            }
            if (color[next] == 0) {
                dfs(next);
                if (hasCycle) return;
            }
        }
        color[node] = 2;   // black: done
        stk.push(node);  // postorder; stack reverses it for us
    }

public:
    vector<int> topoSort(int n, vector<vector<int>>& edges) {
        adj.assign(n, {});
        for (auto& e : edges) adj[e[0]].push_back(e[1]);

        color.assign(n, 0);
        hasCycle = false;

        for (int i = 0; i < n; i++) {
            if (color[i] == 0) dfs(i);
            if (hasCycle) return {};
        }

        vector<int> order;
        while (!stk.empty()) {
            order.push_back(stk.top());
            stk.pop();
        }
        return order;
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adj(numCourses);
        vector<int> indegree(numCourses, 0);

        // prerequisites[i] = [course, prereq]: must take prereq before course
        for (auto& p : prerequisites) {
            int course = p[0], prereq = p[1];
            adj[prereq].push_back(course);
            indegree[course]++;
        }

        queue<int> q;
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) q.push(i);
        }

        int processed = 0;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            processed++;
            for (int next : adj[node]) {
                if (--indegree[next] == 0) q.push(next);
            }
        }
        return processed == numCourses;
    }
};
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 20. Course Schedule II — Medium (LC210)

**Idea:** Same setup as LC207, but record the order in which nodes are popped. If a cycle prevents processing all nodes, return an empty array.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adj(numCourses);
        vector<int> indegree(numCourses, 0);

        for (auto& p : prerequisites) {
            int course = p[0], prereq = p[1];
            adj[prereq].push_back(course);
            indegree[course]++;
        }

        queue<int> q;
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) q.push(i);
        }

        vector<int> order;
        while (!q.empty()) {
            int node = q.front(); q.pop();
            order.push_back(node);
            for (int next : adj[node]) {
                if (--indegree[next] == 0) q.push(next);
            }
        }

        return (int)order.size() == numCourses ? order : vector<int>{};
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    string alienOrder(vector<string>& words) {
        // Track all characters that appear, and the dependency graph.
        unordered_map<char, unordered_set<char>> adj;
        unordered_map<char, int> indegree;

        for (auto& w : words) {
            for (char c : w) {
                if (!adj.count(c)) adj[c] = {};
                if (!indegree.count(c)) indegree[c] = 0;
            }
        }

        // Build edges from each adjacent pair of words.
        for (int i = 0; i + 1 < (int)words.size(); i++) {
            auto& a = words[i]; auto& b = words[i + 1];
            int minLen = min(a.length(), b.length());
            int j = 0;
            while (j < (int)minLen && a[j] == b[j]) j++;

            if (j < (int)minLen) {
                char from = a[j], to = b[j];
                if (!adj[from].count(to)) {
                    adj[from].insert(to);
                    indegree[to]++;
                }
            } else if (a.length() > b.length()) {
                // b is a prefix of a but comes first -> invalid ordering
                return "";
            }
        }

        queue<char> q;
        for (auto& [c, deg] : indegree) {
            if (deg == 0) q.push(c);
        }

        string result;
        while (!q.empty()) {
            char c = q.front(); q.pop();
            result += c;
            for (char next : adj[c]) {
                indegree[next]--;
                if (indegree[next] == 0) q.push(next);
            }
        }

        // If not all characters placed, there was a cycle.
        return result.length() == indegree.size() ? result : "";
    }
};
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

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> findMinHeightTrees(int n, vector<vector<int>>& edges) {
        if (n == 1) return {0};

        vector<unordered_set<int>> adj(n);
        for (auto& e : edges) {
            adj[e[0]].insert(e[1]);
            adj[e[1]].insert(e[0]);
        }

        // Initial leaves: degree 1.
        vector<int> leaves;
        for (int i = 0; i < n; i++) {
            if (adj[i].size() == 1) leaves.push_back(i);
        }

        int remaining = n;
        while (remaining > 2) {
            remaining -= leaves.size();
            vector<int> newLeaves;
            for (int leaf : leaves) {
                // Each leaf has exactly one neighbor; detach it.
                int neighbor = *adj[leaf].begin();
                adj[neighbor].erase(leaf);
                if (adj[neighbor].size() == 1) newLeaves.push_back(neighbor);
            }
            leaves = newLeaves;
        }

        return leaves; // the 1 or 2 centroids
    }
};
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 23. Sequence Reconstruction — Medium (LC444)

**Idea:** Check whether `org` is the **unique** topological order consistent with the given subsequences `seqs`. Build a graph from consecutive elements within each sequence, then run Kahn's. Uniqueness holds iff at every step the queue contains **exactly one** node, the produced order matches `org`, and every element of `org` appears.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    bool sequenceReconstruction(vector<int>& org, vector<vector<int>>& seqs) {
        int n = org.size();
        unordered_map<int, vector<int>> adj;
        unordered_map<int, int> indegree;

        // Register every value that appears in seqs.
        for (auto& seq : seqs) {
            for (int v : seq) {
                if (!adj.count(v)) adj[v] = {};
                if (!indegree.count(v)) indegree[v] = 0;
            }
        }

        for (auto& seq : seqs) {
            for (int i = 0; i + 1 < (int)seq.size(); i++) {
                int from = seq[i], to = seq[i + 1];
                adj[from].push_back(to);
                indegree[to]++;
            }
        }

        // Every value must lie in [1, n] and match org's universe.
        if ((int)indegree.size() != n) return false;

        queue<int> q;
        for (auto& [v, deg] : indegree) {
            if (deg == 0) q.push(v);
        }

        int idx = 0;
        while (!q.empty()) {
            if (q.size() > 1) return false;      // order not unique
            int node = q.front(); q.pop();
            if (idx >= n || org[idx] != node) return false; // mismatch
            idx++;
            for (int next : adj[node]) {
                indegree[next]--;
                if (indegree[next] == 0) q.push(next);
            }
        }

        return idx == n;
    }
};
```

**Complexity:** Time `O(V + E)` where `E` is the total length of all sequences, Space `O(V + E)`.

---

## 24. Parallel Courses — Medium (LC1136)

**Idea:** All courses with no remaining prerequisites can be taken **in the same semester (in parallel)**. So run Kahn's **by levels**: each BFS layer is one semester. The number of layers is the minimum number of semesters. If a cycle blocks some courses, return `-1`.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    int minimumSemesters(int n, vector<vector<int>>& relations) {
        vector<vector<int>> adj(n + 1); // 1-indexed
        vector<int> indegree(n + 1, 0);

        for (auto& r : relations) {
            int prev = r[0], next = r[1]; // prev -> next
            adj[prev].push_back(next);
            indegree[next]++;
        }

        queue<int> q;
        for (int i = 1; i <= n; i++) {
            if (indegree[i] == 0) q.push(i);
        }

        int semesters = 0, studied = 0;
        while (!q.empty()) {
            semesters++;
            int size = q.size();
            for (int s = 0; s < size; s++) {
                int node = q.front(); q.pop();
                studied++;
                for (int next : adj[node]) {
                    if (--indegree[next] == 0) q.push(next);
                }
            }
        }

        return studied == n ? semesters : -1;
    }
};
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 25. Find Eventual Safe States — Medium (LC802)

**Idea:** A node is **safe** if every path from it leads to a terminal node (no cycle reachable). Reverse the graph and run Kahn's: terminal nodes (outdegree 0) become **indegree 0 in the reversed graph**. Peeling them off marks all nodes whose every outgoing edge leads only to safe nodes. Return safe nodes sorted.

```cpp
#include <bits/stdc++.h>
using namespace std;

class Solution {
public:
    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<vector<int>> rev(n);
        vector<int> outdegree(n, 0); // outdegree in original graph

        for (int u = 0; u < n; u++) {
            for (int v : graph[u]) {
                rev[v].push_back(u);  // reversed edge v -> u
                outdegree[u]++;
            }
        }

        queue<int> q;
        for (int i = 0; i < n; i++) {
            if (outdegree[i] == 0) q.push(i); // terminal nodes
        }

        vector<bool> safe(n, false);
        while (!q.empty()) {
            int node = q.front(); q.pop();
            safe[node] = true;
            for (int prev : rev[node]) {
                if (--outdegree[prev] == 0) q.push(prev);
            }
        }

        vector<int> result;
        for (int i = 0; i < n; i++) {
            if (safe[i]) result.push_back(i); // already in ascending order
        }
        return result;
    }
};
```

**Alternative — DFS 3-color:** mark each node WHITE/GRAY/BLACK; a node is safe iff DFS from it never touches a GRAY node (no back edge / cycle). BLACK = safe.

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

> **Last Updated:** 2026-06-26
