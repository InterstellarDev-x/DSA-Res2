> **Topic:** [Graphs](../README.md) · **Section:** Interview Problems · **Company:** Amazon

# Amazon — Graphs Interview Deep Dives

Full-reasoning walkthroughs of the graph problems Amazon asks most. Each includes the decision behind the approach, edge cases, and follow-ups. See also [Google](./Google.md) and [Microsoft](./Microsoft.md).

---

## 1. Number of Islands (LC 200)

Count 4-connected components of `'1'` in an `m x n` grid.

### Approach trade-offs

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| **DFS (recursive)** | O(m·n) | O(m·n) stack worst case | Cleanest; risk of stack overflow on huge single islands |
| **BFS (iterative)** | O(m·n) | O(min(m,n)) queue | Safe for large grids; preferred when depth unbounded |
| **Union-Find** | O(m·n·α) | O(m·n) | Best when land is **added incrementally** (LC 305) |

The key shared idea: **mark visited in place** by setting `grid[r][c] = '0'`, which removes the need for a separate visited matrix.

### BFS solution

```cpp
#include <bits/stdc++.h>
using namespace std;

int numIslands(vector<vector<char>>& grid) {
    if (grid.empty() || grid[0].empty()) return 0;
    int m = grid.size(), n = grid[0].size(), count = 0;
    vector<vector<int>> dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    queue<pair<int,int>> q;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] != '1') continue;
            count++;
            grid[i][j] = '0';                 // mark at enqueue
            q.push({i, j});
            while (!q.empty()) {
                auto [r0, c0] = q.front(); q.pop();
                for (auto& d : dirs) {
                    int r = r0 + d[0], c = c0 + d[1];
                    if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != '1') continue;
                    grid[r][c] = '0';         // mark BEFORE enqueue → no duplicates
                    q.push({r, c});
                }
            }
        }
    }
    return count;
}
```

**Why mark at enqueue, not dequeue?** If you only mark when popping, the same cell can be enqueued by multiple neighbors before it is processed — duplicate work, inflated queue, and potential double counting. Marking the instant you push guarantees each cell enters the queue once.

### Edge cases
- Empty / null grid → `0`.
- All water → `0`; all land → `1`.
- Single row/column (1-D grid).

### Follow-ups
- **Number of Islands II** (LC 305): land cells added one at a time, return island count after each → [Union Find (DSU)](../Patterns/Union%20Find%20(DSU).md).
- **Making A Large Island** (LC 827): flip one `0`; label each island with a size and sum distinct neighboring labels.
- "Grid doesn't fit in memory" → process row-by-row with DSU, or distributed connected components.

---

## 2. Rotting Oranges (LC 994)

Each minute every rotten orange (`2`) rots 4-adjacent fresh (`1`). Minutes until none fresh, else `-1`.

### Why seed all sources first

This is **multi-source BFS**. All rotten oranges rot their neighbors *simultaneously* each minute. If you BFS from one source at a time, you'd compute the wrong (sequential) timing. Seeding the queue with **every** rotten cell at time 0 makes each BFS level == one minute, so they spread in lockstep.

```cpp
#include <bits/stdc++.h>
using namespace std;

int orangesRotting(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size(), fresh = 0, minutes = 0;
    queue<pair<int,int>> q;
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) q.push({i, j});
            else if (grid[i][j] == 1) fresh++;
        }
    if (fresh == 0) return 0;                  // edge case: nothing to rot
    vector<vector<int>> dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!q.empty() && fresh > 0) {
        minutes++;
        int size = q.size();
        for (int s = 0; s < size; s++) {       // process exactly one level
            auto [r0, c0] = q.front(); q.pop();
            for (auto& d : dirs) {
                int r = r0 + d[0], c = c0 + d[1];
                if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != 1) continue;
                grid[r][c] = 2;
                fresh--;
                q.push({r, c});
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```

### Edge cases
- No fresh oranges at start → `0` (don't return 1).
- A fresh orange with no path to any rotten → `-1` (tracked via `fresh` counter).
- Processing one level at a time with `size = q.size()` is essential to count minutes.

### Follow-ups
- **01 Matrix / Walls and Gates** — identical multi-source seeding.
- Different rot speeds per cell → weighted → [Shortest Path](../Patterns/Shortest%20Path.md) (Dijkstra / 0-1 BFS).

---

## 3. Course Schedule II (LC 210)

Return a valid course order, or `[]` if a cycle exists.

### Kahn's vs DFS

| | Kahn's (BFS indegree) | DFS postorder |
|--|----------------------|---------------|
| Cycle detection | `processed < numCourses` | back-edge (gray node) in 3-color |
| Output order | natural forward order | reverse of postorder |
| Intuition | "remove zero-dependency nodes" | "finish dependents first" |

Kahn's is usually cleaner for returning the order:

```cpp
#include <bits/stdc++.h>
using namespace std;

vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(numCourses);
    vector<int> indegree(numCourses, 0);
    for (auto& p : prerequisites) {        // edge prereq p[1] -> course p[0]
        adj[p[1]].push_back(p[0]);
        indegree[p[0]]++;
    }
    queue<int> q;
    for (int i = 0; i < numCourses; i++)
        if (indegree[i] == 0) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);
        for (int next : adj[node])
            if (--indegree[next] == 0) q.push(next);
    }
    return (int)order.size() == numCourses ? order : vector<int>{};   // cycle ⇒ not all processed
}
```

### Edge cases
- No prerequisites → any order (here: `0..n-1`).
- Self-loop `[i,i]` → cycle → `[]`.
- Disconnected DAG components → all still get ordered.

### Follow-ups
- **Course Schedule I** — feasibility only.
- **Alien Dictionary** — build the graph first, then this exact topo-sort.
- **Parallel Courses** — minimum semesters = number of BFS levels.

---

## 4. Word Ladder (LC 127)

Shortest transformation length from `beginWord` to `endWord`.

### Wildcard bucket optimization

Naively, generating neighbors of a word means trying all 26 letters at each of L positions and checking membership — fine, but pairwise comparison across the list is O(N²·L). Instead **bucket by wildcard pattern**: `hot → *ot, h*t, ho*`. Words sharing a bucket differ by exactly one letter, giving O(N·L) adjacency.

```cpp
#include <bits/stdc++.h>
using namespace std;

int ladderLength(string beginWord, string endWord, vector<string>& wordList) {
    unordered_set<string> dict(wordList.begin(), wordList.end());
    if (!dict.count(endWord)) return 0;
    unordered_map<string, vector<string>> buckets;
    for (auto& w : dict)
        for (int i = 0; i < (int)w.length(); i++) {
            string pattern = w.substr(0, i) + "*" + w.substr(i + 1);
            buckets[pattern].push_back(w);
        }
    queue<string> q;
    unordered_set<string> visited;
    q.push(beginWord);
    visited.insert(beginWord);
    int steps = 1;
    while (!q.empty()) {
        int size = q.size();
        for (int s = 0; s < size; s++) {
            string word = q.front(); q.pop();
            if (word == endWord) return steps;
            for (int i = 0; i < (int)word.length(); i++) {
                string pattern = word.substr(0, i) + "*" + word.substr(i + 1);
                auto it = buckets.find(pattern);
                if (it == buckets.end()) continue;
                for (auto& next : it->second) {
                    if (!visited.count(next)) {
                        visited.insert(next);
                        q.push(next);   // visited at enqueue
                    }
                }
            }
        }
        steps++;
    }
    return 0;
}
```

### Edge cases
- `endWord` not in dictionary → `0`.
- `beginWord` need not be in the dictionary.
- Words all distinct length assumed (LC guarantees equal length).

### Follow-ups
- **Word Ladder II** — return all shortest paths (BFS layers + parent backtrack); see [Google deep dive](./Google.md).
- **Bidirectional BFS** — search from both ends, swap to the smaller frontier; large dictionaries see big speedups.

---

## Amazon Leadership Principle Alignment

| LP | Demonstrated by |
|----|-----------------|
| **Customer Obsession** | Clarifying input format and failure semantics (`-1`, `[]`) up front |
| **Dive Deep** | Explaining enqueue-time marking, multi-source seeding, bucket adjacency |
| **Bias for Action** | Reaching a correct standard template quickly |
| **Insist on Highest Standards** | Handling degenerate grids, self-loops, unreachable cells |
| **Are Right, A Lot** | Choosing BFS vs DFS vs DSU with justification |
| **Invent and Simplify** | Wildcard buckets, in-place marking |

> **Last Updated:** 2026-06-26
