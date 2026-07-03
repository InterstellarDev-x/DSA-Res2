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

```cpp
#include <bits/stdc++.h>
using namespace std;

string alienOrder(vector<string>& words) {
    unordered_map<char, unordered_set<char>> adj;
    unordered_map<char, int> indegree;
    for (auto& w : words)
        for (char c : w) {
            if (!adj.count(c)) adj[c] = {};
            if (!indegree.count(c)) indegree[c] = 0;  // every char is a node
        }
    for (int i = 0; i + 1 < (int)words.size(); i++) {
        auto& a = words[i]; auto& b = words[i + 1];
        int len = min(a.length(), b.length());
        int j = 0;
        while (j < (int)len && a[j] == b[j]) j++;
        if (j == (int)len) {
            if (a.length() > b.length()) return "";   // prefix conflict
        } else {
            char from = a[j], to = b[j];
            if (adj[from].insert(to).second) indegree[to]++;
        }
    }
    queue<char> q;
    for (auto& [c, deg] : indegree)
        if (deg == 0) q.push(c);
    string sb;
    while (!q.empty()) {
        char c = q.front(); q.pop();
        sb += c;
        for (char next : adj[c])
            if (--indegree[next] == 0) q.push(next);
    }
    return sb.size() == indegree.size() ? sb : "";  // cycle ⇒ ""
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

```cpp
#include <bits/stdc++.h>
using namespace std;

int timer_ = 0;

void dfs(int u, int parent, vector<vector<int>>& adj,
         vector<int>& disc, vector<int>& low, vector<vector<int>>& bridges) {
    disc[u] = low[u] = timer_++;
    for (int v : adj[u]) {
        if (v == parent) continue;            // skip the edge we came in on
        if (disc[v] == -1) {
            dfs(v, u, adj, disc, low, bridges);
            low[u] = min(low[u], low[v]);
            if (low[v] > disc[u]) bridges.push_back({u, v});   // bridge test
        } else {
            low[u] = min(low[u], disc[v]);  // back edge
        }
    }
}

vector<vector<int>> criticalConnections(int n, vector<vector<int>>& connections) {
    vector<vector<int>> adj(n);
    for (auto& e : connections) {
        adj[e[0]].push_back(e[1]);
        adj[e[1]].push_back(e[0]);
    }
    vector<int> disc(n, -1), low(n, 0);
    vector<vector<int>> bridges;
    dfs(0, -1, adj, disc, low, bridges);
    return bridges;
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

```cpp
#include <bits/stdc++.h>
using namespace std;

int minCostConnectPoints(vector<vector<int>>& points) {
    int n = points.size();
    vector<bool> inMST(n, false);
    vector<int> minDist(n, INT_MAX);
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
            int d = abs(points[u][0] - points[v][0])
                  + abs(points[u][1] - points[v][1]);
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

```cpp
#include <bits/stdc++.h>
using namespace std;

void backtrack(const string& word, const string& begin,
               unordered_map<string, vector<string>>& parents,
               deque<string>& path, vector<vector<string>>& res) {
    path.push_front(word);
    if (word == begin) {
        res.push_back(vector<string>(path.begin(), path.end()));
    } else if (parents.count(word)) {
        for (auto& p : parents[word]) backtrack(p, begin, parents, path, res);
    }
    path.pop_front();
}

vector<vector<string>> findLadders(string beginWord, string endWord, vector<string>& wordList) {
    unordered_set<string> dict(wordList.begin(), wordList.end());
    vector<vector<string>> res;
    if (!dict.count(endWord)) return res;
    unordered_map<string, vector<string>> parents;
    unordered_set<string> level;
    level.insert(beginWord);
    bool found = false;
    while (!level.empty() && !found) {
        for (auto& w : level) dict.erase(w);         // remove this layer from dict
        unordered_set<string> next;
        for (auto& word : level) {
            string arr = word;
            for (int i = 0; i < (int)arr.size(); i++) {
                char old = arr[i];
                for (char c = 'a'; c <= 'z'; c++) {
                    arr[i] = c;
                    if (!dict.count(arr)) continue;
                    if (arr == endWord) found = true;
                    next.insert(arr);
                    parents[arr].push_back(word);
                }
                arr[i] = old;
            }
        }
        level = next;
    }
    if (found) {
        deque<string> path;
        backtrack(endWord, beginWord, parents, path, res);
    }
    return res;
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
