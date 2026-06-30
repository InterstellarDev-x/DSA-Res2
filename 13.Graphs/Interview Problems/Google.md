> **Topic:** [Graphs](../README.md) · **Section:** Interview Problems · **Company:** Google

# Google — Graphs Interview Deep Dives

Google graph interviews reward building graphs from non-obvious structure and reasoning precisely about edge cases and optimal complexity. See also [Amazon](./Amazon.md) and [Microsoft](./Microsoft.md).

---

## 1. Alien Dictionary (LC 269)

Given words sorted by an unknown alphabet, return a valid character order, or `""` if invalid.

### Building the graph from word pairs

Order information lives only in **adjacent pairs**. For consecutive words `w1`, `w2`, scan to the first differing character: that gives a single edge `w1[i] → w2[i]`. Stop at the first difference — later characters tell you nothing.

### The prefix-conflict edge case

If `w1` is a **prefix-superset** of `w2` — i.e. `w2` is a proper prefix of `w1` like `["abc", "ab"]` — a longer word sorts before its own prefix, which is impossible. This must return `""`. It's the single most-missed case in this problem.

```java
public String alienOrder(String[] words) {
    Map<Character, Set<Character>> adj = new HashMap<>();
    Map<Character, Integer> indegree = new HashMap<>();
    for (String w : words)
        for (char c : w.toCharArray()) {
            adj.putIfAbsent(c, new HashSet<>());
            indegree.putIfAbsent(c, 0);      // every char is a node
        }
    for (int i = 0; i + 1 < words.length; i++) {
        String a = words[i], b = words[i + 1];
        int len = Math.min(a.length(), b.length());
        int j = 0;
        while (j < len && a.charAt(j) == b.charAt(j)) j++;
        if (j == len) {
            if (a.length() > b.length()) return "";   // prefix conflict
        } else {
            char from = a.charAt(j), to = b.charAt(j);
            if (adj.get(from).add(to)) indegree.merge(to, 1, Integer::sum);
        }
    }
    Queue<Character> queue = new ArrayDeque<>();
    for (var e : indegree.entrySet())
        if (e.getValue() == 0) queue.offer(e.getKey());
    StringBuilder sb = new StringBuilder();
    while (!queue.isEmpty()) {
        char c = queue.poll();
        sb.append(c);
        for (char next : adj.get(c))
            if (indegree.merge(next, -1, Integer::sum) == 0) queue.offer(next);
    }
    return sb.length() == indegree.size() ? sb.toString() : "";  // cycle ⇒ ""
}
```

### Edge cases
- Single word → all its distinct chars in any order.
- Duplicate adjacent words → no edge, fine.
- A cycle (`["a","b","a"]`) → not all chars output → `""`.
- Prefix conflict → `""`.

### Follow-ups — return all valid orderings; handle when the alphabet is known/partial.

---

## 2. Critical Connections (LC 1192) — Tarjan bridges

Find every edge whose removal disconnects the graph (a **bridge**).

### disc / low explained

Run a single DFS assigning each node a **discovery time** `disc[u]` (the timer when first visited). `low[u]` is the smallest discovery time reachable from `u`'s subtree using tree edges plus **at most one back edge**. An edge `(u, v)` (v a child of u) is a bridge **iff** `low[v] > disc[u]` — meaning v's subtree has no back edge climbing to u or above, so cutting `(u,v)` isolates it.

```java
int timer = 0;
public List<List<Integer>> criticalConnections(int n, List<List<Integer>> connections) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
    for (List<Integer> e : connections) {
        adj.get(e.get(0)).add(e.get(1));
        adj.get(e.get(1)).add(e.get(0));
    }
    int[] disc = new int[n], low = new int[n];
    Arrays.fill(disc, -1);
    List<List<Integer>> bridges = new ArrayList<>();
    dfs(0, -1, adj, disc, low, bridges);
    return bridges;
}
private void dfs(int u, int parent, List<List<Integer>> adj,
                 int[] disc, int[] low, List<List<Integer>> bridges) {
    disc[u] = low[u] = timer++;
    for (int v : adj.get(u)) {
        if (v == parent) continue;            // skip the edge we came in on
        if (disc[v] == -1) {
            dfs(v, u, adj, disc, low, bridges);
            low[u] = Math.min(low[u], low[v]);
            if (low[v] > disc[u]) bridges.add(List.of(u, v));   // bridge test
        } else {
            low[u] = Math.min(low[u], disc[v]);  // back edge
        }
    }
}
```

**Subtlety:** `parent` must be skipped only once (matters with parallel edges; if duplicates possible, track the edge id instead of the parent node).

### Follow-ups — articulation points (`low[v] >= disc[u]`, root special-cased); Tarjan SCC for directed graphs.

---

## 3. Min Cost to Connect Points (LC 1584) — Prim vs Kruskal

Connect all points with minimum total Manhattan distance.

### The decision

This is an MST on a **complete graph** (every pair is an edge → E = O(V²)).

- **Prim's**: grow a tree from one vertex, repeatedly adding the cheapest edge to an outside vertex. On a dense/complete graph, an array-based Prim is **O(V²)** with no edge list needed — ideal here.
- **Kruskal's**: sort all O(V²) edges, union endpoints if disjoint. Sorting dominates at **O(V² log V)**, and you must materialize every edge.

→ For a complete graph, **Prim wins**. Kruskal shines on **sparse** graphs given as an edge list.

```java
public int minCostConnectPoints(int[][] points) {
    int n = points.length;
    boolean[] inMST = new boolean[n];
    int[] minDist = new int[n];
    Arrays.fill(minDist, Integer.MAX_VALUE);
    minDist[0] = 0;
    int total = 0;
    for (int iter = 0; iter < n; iter++) {
        int u = -1;
        for (int v = 0; v < n; v++)                 // pick cheapest outside vertex
            if (!inMST[v] && (u == -1 || minDist[v] < minDist[u])) u = v;
        inMST[u] = true;
        total += minDist[u];
        for (int v = 0; v < n; v++) {               // relax neighbors
            if (inMST[v]) continue;
            int d = Math.abs(points[u][0] - points[v][0])
                  + Math.abs(points[u][1] - points[v][1]);
            if (d < minDist[v]) minDist[v] = d;
        }
    }
    return total;
}
```

### Follow-ups — *Connecting Cities With Minimum Cost*; *Optimize Water Distribution* (virtual node for wells); *Find Critical & Pseudo-Critical Edges* (Kruskal with forced/excluded edges).

---

## 4. Word Ladder II (LC 126) — BFS layering + parent backtrack

Return **all** shortest transformation sequences.

### The two-phase idea

A single BFS finds the shortest length but not the paths. The clean approach: **BFS to build a parent (predecessor) map level by level**, ensuring we only keep predecessors from the *previous* layer (so all recorded paths are shortest), then **DFS-backtrack** from `endWord` to `beginWord` to reconstruct every path.

```java
public List<List<String>> findLadders(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    List<List<String>> res = new ArrayList<>();
    if (!dict.contains(endWord)) return res;
    Map<String, List<String>> parents = new HashMap<>();
    Set<String> level = new HashSet<>();
    level.add(beginWord);
    boolean found = false;
    while (!level.isEmpty() && !found) {
        dict.removeAll(level);                       // remove this layer from dict
        Set<String> next = new HashSet<>();
        for (String word : level) {
            char[] arr = word.toCharArray();
            for (int i = 0; i < arr.length; i++) {
                char old = arr[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    arr[i] = c;
                    String cand = new String(arr);
                    if (!dict.contains(cand)) continue;
                    if (cand.equals(endWord)) found = true;
                    next.add(cand);
                    parents.computeIfAbsent(cand, k -> new ArrayList<>()).add(word);
                }
                arr[i] = old;
            }
        }
        level = next;
    }
    if (found) backtrack(endWord, beginWord, parents, new LinkedList<>(), res);
    return res;
}
private void backtrack(String word, String begin, Map<String, List<String>> parents,
                       LinkedList<String> path, List<List<String>> res) {
    path.addFirst(word);
    if (word.equals(begin)) {
        res.add(new ArrayList<>(path));
    } else if (parents.containsKey(word)) {
        for (String p : parents.get(word)) backtrack(p, begin, parents, path, res);
    }
    path.removeFirst();
}
```

**Why remove each layer from the dict?** It guarantees a word is only reached via a strictly shorter path, so every parent edge belongs to a shortest path — no longer detours sneak in.

### Follow-ups — count (not list) shortest paths; bidirectional layered BFS for performance.

---

## L4 / L5 Level Context

| Aspect | L4 expectation | L5 expectation |
|--------|----------------|----------------|
| Graph modeling | Build the graph correctly when guided | See the hidden graph immediately (word pairs → edges) |
| Edge cases | Handle prompted ones | Surface prefix-conflict / cycle / disconnected unprompted |
| Algorithm depth | Apply Kahn's / Dijkstra | Derive Tarjan disc/low, justify Prim-vs-Kruskal |
| Communication | Explain after | Narrate trade-offs and complexity while designing |

> **Last Updated:** 2026-06-26
