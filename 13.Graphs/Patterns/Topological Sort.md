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

```java
import java.util.*;

public class KahnTemplate {
    // n nodes labelled 0..n-1, edges given as directed [from, to]
    public int[] topoSort(int n, int[][] edges) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        int[] indegree = new int[n];

        for (int[] e : edges) {
            int from = e[0], to = e[1];
            adj.get(from).add(to);
            indegree[to]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int[] order = new int[n];
        int idx = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            order[idx++] = node;
            for (int next : adj.get(node)) {
                if (--indegree[next] == 0) queue.offer(next);
            }
        }

        // If we couldn't place every node, a cycle exists -> no valid order.
        if (idx != n) return new int[0];
        return order;
    }
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

```java
import java.util.*;

public class DfsTopoTemplate {
    private List<List<Integer>> adj;
    private int[] color;   // 0 = white, 1 = gray, 2 = black
    private Deque<Integer> stack;
    private boolean hasCycle;

    public int[] topoSort(int n, int[][] edges) {
        adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] e : edges) adj.get(e[0]).add(e[1]);

        color = new int[n];
        stack = new ArrayDeque<>();
        hasCycle = false;

        for (int i = 0; i < n; i++) {
            if (color[i] == 0) dfs(i);
            if (hasCycle) return new int[0];
        }

        int[] order = new int[n];
        int idx = 0;
        while (!stack.isEmpty()) order[idx++] = stack.pop();
        return order;
    }

    private void dfs(int node) {
        color[node] = 1; // gray: on the recursion stack
        for (int next : adj.get(node)) {
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
        stack.push(node);  // postorder; stack reverses it for us
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

```java
import java.util.*;

class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        int[] indegree = new int[numCourses];

        // prerequisites[i] = [course, prereq]: must take prereq before course
        for (int[] p : prerequisites) {
            int course = p[0], prereq = p[1];
            adj.get(prereq).add(course);
            indegree[course]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int processed = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            processed++;
            for (int next : adj.get(node)) {
                if (--indegree[next] == 0) queue.offer(next);
            }
        }
        return processed == numCourses;
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 20. Course Schedule II — Medium (LC210)

**Idea:** Same setup as LC207, but record the order in which nodes are popped. If a cycle prevents processing all nodes, return an empty array.

```java
import java.util.*;

class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
        int[] indegree = new int[numCourses];

        for (int[] p : prerequisites) {
            int course = p[0], prereq = p[1];
            adj.get(prereq).add(course);
            indegree[course]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int[] order = new int[numCourses];
        int idx = 0;
        while (!queue.isEmpty()) {
            int node = queue.poll();
            order[idx++] = node;
            for (int next : adj.get(node)) {
                if (--indegree[next] == 0) queue.offer(next);
            }
        }

        return idx == numCourses ? order : new int[0];
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

```java
import java.util.*;

class Solution {
    public String alienOrder(String[] words) {
        // Track all characters that appear, and the dependency graph.
        Map<Character, Set<Character>> adj = new HashMap<>();
        Map<Character, Integer> indegree = new HashMap<>();

        for (String w : words) {
            for (char c : w.toCharArray()) {
                adj.putIfAbsent(c, new HashSet<>());
                indegree.putIfAbsent(c, 0);
            }
        }

        // Build edges from each adjacent pair of words.
        for (int i = 0; i + 1 < words.length; i++) {
            String a = words[i], b = words[i + 1];
            int minLen = Math.min(a.length(), b.length());
            int j = 0;
            while (j < minLen && a.charAt(j) == b.charAt(j)) j++;

            if (j < minLen) {
                char from = a.charAt(j), to = b.charAt(j);
                if (!adj.get(from).contains(to)) {
                    adj.get(from).add(to);
                    indegree.put(to, indegree.get(to) + 1);
                }
            } else if (a.length() > b.length()) {
                // b is a prefix of a but comes first -> invalid ordering
                return "";
            }
        }

        Queue<Character> queue = new LinkedList<>();
        for (Map.Entry<Character, Integer> e : indegree.entrySet()) {
            if (e.getValue() == 0) queue.offer(e.getKey());
        }

        StringBuilder sb = new StringBuilder();
        while (!queue.isEmpty()) {
            char c = queue.poll();
            sb.append(c);
            for (char next : adj.get(c)) {
                indegree.put(next, indegree.get(next) - 1);
                if (indegree.get(next) == 0) queue.offer(next);
            }
        }

        // If not all characters placed, there was a cycle.
        return sb.length() == indegree.size() ? sb.toString() : "";
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

```java
import java.util.*;

class Solution {
    public List<Integer> findMinHeightTrees(int n, int[][] edges) {
        if (n == 1) return Collections.singletonList(0);

        List<Set<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new HashSet<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(e[1]);
            adj.get(e[1]).add(e[0]);
        }

        // Initial leaves: degree 1.
        List<Integer> leaves = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (adj.get(i).size() == 1) leaves.add(i);
        }

        int remaining = n;
        while (remaining > 2) {
            remaining -= leaves.size();
            List<Integer> newLeaves = new ArrayList<>();
            for (int leaf : leaves) {
                // Each leaf has exactly one neighbor; detach it.
                int neighbor = adj.get(leaf).iterator().next();
                adj.get(neighbor).remove(leaf);
                if (adj.get(neighbor).size() == 1) newLeaves.add(neighbor);
            }
            leaves = newLeaves;
        }

        return leaves; // the 1 or 2 centroids
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 23. Sequence Reconstruction — Medium (LC444)

**Idea:** Check whether `org` is the **unique** topological order consistent with the given subsequences `seqs`. Build a graph from consecutive elements within each sequence, then run Kahn's. Uniqueness holds iff at every step the queue contains **exactly one** node, the produced order matches `org`, and every element of `org` appears.

```java
import java.util.*;

class Solution {
    public boolean sequenceReconstruction(int[] org, List<List<Integer>> seqs) {
        int n = org.length;
        Map<Integer, List<Integer>> adj = new HashMap<>();
        Map<Integer, Integer> indegree = new HashMap<>();

        // Register every value that appears in seqs.
        for (List<Integer> seq : seqs) {
            for (int v : seq) {
                adj.putIfAbsent(v, new ArrayList<>());
                indegree.putIfAbsent(v, 0);
            }
        }

        for (List<Integer> seq : seqs) {
            for (int i = 0; i + 1 < seq.size(); i++) {
                int from = seq.get(i), to = seq.get(i + 1);
                adj.get(from).add(to);
                indegree.put(to, indegree.get(to) + 1);
            }
        }

        // Every value must lie in [1, n] and match org's universe.
        if (indegree.size() != n) return false;

        Queue<Integer> queue = new LinkedList<>();
        for (Map.Entry<Integer, Integer> e : indegree.entrySet()) {
            if (e.getValue() == 0) queue.offer(e.getKey());
        }

        int idx = 0;
        while (!queue.isEmpty()) {
            if (queue.size() > 1) return false;      // order not unique
            int node = queue.poll();
            if (idx >= n || org[idx] != node) return false; // mismatch
            idx++;
            for (int next : adj.get(node)) {
                indegree.put(next, indegree.get(next) - 1);
                if (indegree.get(next) == 0) queue.offer(next);
            }
        }

        return idx == n;
    }
}
```

**Complexity:** Time `O(V + E)` where `E` is the total length of all sequences, Space `O(V + E)`.

---

## 24. Parallel Courses — Medium (LC1136)

**Idea:** All courses with no remaining prerequisites can be taken **in the same semester (in parallel)**. So run Kahn's **by levels**: each BFS layer is one semester. The number of layers is the minimum number of semesters. If a cycle blocks some courses, return `-1`.

```java
import java.util.*;

class Solution {
    public int minimumSemesters(int n, int[][] relations) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i <= n; i++) adj.add(new ArrayList<>()); // 1-indexed
        int[] indegree = new int[n + 1];

        for (int[] r : relations) {
            int prev = r[0], next = r[1]; // prev -> next
            adj.get(prev).add(next);
            indegree[next]++;
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 1; i <= n; i++) {
            if (indegree[i] == 0) queue.offer(i);
        }

        int semesters = 0, studied = 0;
        while (!queue.isEmpty()) {
            semesters++;
            int size = queue.size();
            for (int s = 0; s < size; s++) {
                int node = queue.poll();
                studied++;
                for (int next : adj.get(node)) {
                    if (--indegree[next] == 0) queue.offer(next);
                }
            }
        }

        return studied == n ? semesters : -1;
    }
}
```

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

## 25. Find Eventual Safe States — Medium (LC802)

**Idea:** A node is **safe** if every path from it leads to a terminal node (no cycle reachable). Reverse the graph and run Kahn's: terminal nodes (outdegree 0) become **indegree 0 in the reversed graph**. Peeling them off marks all nodes whose every outgoing edge leads only to safe nodes. Return safe nodes sorted.

```java
import java.util.*;

class Solution {
    public List<Integer> eventualSafeNodes(int[][] graph) {
        int n = graph.length;
        List<List<Integer>> reverse = new ArrayList<>();
        for (int i = 0; i < n; i++) reverse.add(new ArrayList<>());
        int[] outdegree = new int[n]; // outdegree in original graph

        for (int u = 0; u < n; u++) {
            for (int v : graph[u]) {
                reverse.get(v).add(u);  // reversed edge v -> u
                outdegree[u]++;
            }
        }

        Queue<Integer> queue = new LinkedList<>();
        for (int i = 0; i < n; i++) {
            if (outdegree[i] == 0) queue.offer(i); // terminal nodes
        }

        boolean[] safe = new boolean[n];
        while (!queue.isEmpty()) {
            int node = queue.poll();
            safe[node] = true;
            for (int prev : reverse.get(node)) {
                if (--outdegree[prev] == 0) queue.offer(prev);
            }
        }

        List<Integer> result = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if (safe[i]) result.add(i); // already in ascending order
        }
        return result;
    }
}
```

**Alternative — DFS 3-color:** mark each node WHITE/GRAY/BLACK; a node is safe iff DFS from it never touches a GRAY node (no back edge / cycle). BLACK = safe.

**Complexity:** Time `O(V + E)`, Space `O(V + E)`.

---

> **Last Updated:** 2026-06-26
