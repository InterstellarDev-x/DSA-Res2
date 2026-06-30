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

```java
public int numIslands(char[][] grid) {
    if (grid == null || grid.length == 0) return 0;
    int m = grid.length, n = grid[0].length, count = 0;
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    Queue<int[]> queue = new ArrayDeque<>();
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (grid[i][j] != '1') continue;
            count++;
            grid[i][j] = '0';                 // mark at enqueue
            queue.offer(new int[]{i, j});
            while (!queue.isEmpty()) {
                int[] cell = queue.poll();
                for (int[] d : dirs) {
                    int r = cell[0] + d[0], c = cell[1] + d[1];
                    if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != '1') continue;
                    grid[r][c] = '0';         // mark BEFORE enqueue → no duplicates
                    queue.offer(new int[]{r, c});
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

```java
public int orangesRotting(int[][] grid) {
    int m = grid.length, n = grid[0].length, fresh = 0, minutes = 0;
    Queue<int[]> queue = new ArrayDeque<>();
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (grid[i][j] == 2) queue.offer(new int[]{i, j});
            else if (grid[i][j] == 1) fresh++;
        }
    if (fresh == 0) return 0;                  // edge case: nothing to rot
    int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};
    while (!queue.isEmpty() && fresh > 0) {
        minutes++;
        int size = queue.size();
        for (int s = 0; s < size; s++) {       // process exactly one level
            int[] cell = queue.poll();
            for (int[] d : dirs) {
                int r = cell[0] + d[0], c = cell[1] + d[1];
                if (r < 0 || r >= m || c < 0 || c >= n || grid[r][c] != 1) continue;
                grid[r][c] = 2;
                fresh--;
                queue.offer(new int[]{r, c});
            }
        }
    }
    return fresh == 0 ? minutes : -1;
}
```

### Edge cases
- No fresh oranges at start → `0` (don't return 1).
- A fresh orange with no path to any rotten → `-1` (tracked via `fresh` counter).
- Processing one level at a time with `size = queue.size()` is essential to count minutes.

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

```java
public int[] findOrder(int numCourses, int[][] prerequisites) {
    List<List<Integer>> adj = new ArrayList<>();
    for (int i = 0; i < numCourses; i++) adj.add(new ArrayList<>());
    int[] indegree = new int[numCourses];
    for (int[] p : prerequisites) {        // edge prereq p[1] -> course p[0]
        adj.get(p[1]).add(p[0]);
        indegree[p[0]]++;
    }
    Queue<Integer> queue = new ArrayDeque<>();
    for (int i = 0; i < numCourses; i++)
        if (indegree[i] == 0) queue.offer(i);
    int[] order = new int[numCourses];
    int idx = 0;
    while (!queue.isEmpty()) {
        int node = queue.poll();
        order[idx++] = node;
        for (int next : adj.get(node))
            if (--indegree[next] == 0) queue.offer(next);
    }
    return idx == numCourses ? order : new int[0];   // cycle ⇒ not all processed
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

```java
public int ladderLength(String beginWord, String endWord, List<String> wordList) {
    Set<String> dict = new HashSet<>(wordList);
    if (!dict.contains(endWord)) return 0;
    Map<String, List<String>> buckets = new HashMap<>();
    for (String w : dict)
        for (int i = 0; i < w.length(); i++) {
            String pattern = w.substring(0, i) + "*" + w.substring(i + 1);
            buckets.computeIfAbsent(pattern, k -> new ArrayList<>()).add(w);
        }
    Queue<String> queue = new ArrayDeque<>();
    Set<String> visited = new HashSet<>();
    queue.offer(beginWord);
    visited.add(beginWord);
    int steps = 1;
    while (!queue.isEmpty()) {
        int size = queue.size();
        for (int s = 0; s < size; s++) {
            String word = queue.poll();
            if (word.equals(endWord)) return steps;
            for (int i = 0; i < word.length(); i++) {
                String pattern = word.substring(0, i) + "*" + word.substring(i + 1);
                for (String next : buckets.getOrDefault(pattern, List.of())) {
                    if (visited.add(next)) queue.offer(next);   // visited at enqueue
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
