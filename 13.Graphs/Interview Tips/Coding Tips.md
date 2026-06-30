> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 1 of 4**

# Graphs — Coding Tips

Ten battle-tested idioms that make graph code shorter, faster, and correct. See also [Common Mistakes](./Common%20Mistakes.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Standard adjacency-list build idiom

For a weighted graph use `Map<Integer, List<int[]>>`; for unweighted, `List<List<Integer>>`. Build once, reuse everywhere.

```java
List<List<Integer>> adj = new ArrayList<>();
for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
for (int[] e : edges) {
    adj.get(e[0]).add(e[1]);
    adj.get(e[1]).add(e[0]);          // BOTH directions for undirected
}
// weighted:
Map<Integer, List<int[]>> wadj = new HashMap<>();
for (int[] e : edges)
    wadj.computeIfAbsent(e[0], k -> new ArrayList<>()).add(new int[]{e[1], e[2]});
```

## 2. Use an `int[][] dirs` array for grid neighbors

Replaces four near-identical blocks with one loop. Use 8-dir only when diagonals count.

```java
int[][] dirs = {{1,0},{-1,0},{0,1},{0,-1}};            // 4-directional
int[][] dirs8 = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}}; // 8-dir
for (int[] d : dirs) {
    int r = row + d[0], c = col + d[1];
    if (r < 0 || r >= m || c < 0 || c >= n) continue;  // bounds FIRST
    // ... use grid[r][c]
}
```

## 3. Mark visited BEFORE enqueue, never after dequeue

The single most important BFS rule. Marking at dequeue lets the same node be pushed by multiple neighbors → duplicates, bloated queue, sometimes wrong counts.

```java
visited[start] = true;
queue.offer(start);
while (!queue.isEmpty()) {
    int u = queue.poll();
    for (int v : adj.get(u))
        if (!visited[v]) {
            visited[v] = true;          // mark at the moment you enqueue
            queue.offer(v);
        }
}
```

## 4. Multi-source BFS — seed all sources first

For "spread from many origins simultaneously" (Rotting Oranges, 01 Matrix, Walls and Gates), push every source at time 0, then expand level by level.

```java
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        if (isSource(grid[i][j])) queue.offer(new int[]{i, j});
int level = 0;
while (!queue.isEmpty()) {
    int size = queue.size();            // freeze the layer
    for (int s = 0; s < size; s++) { /* expand one cell */ }
    level++;
}
```

## 5. Dijkstra — stale-entry skip + `Integer.compare`

Lazy-deletion heap: re-push on relax, skip popped entries that are worse than the recorded best. Always compare with `Integer.compare`, never `a[1] - b[1]`.

```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));
pq.offer(new int[]{src, 0});
while (!pq.isEmpty()) {
    int[] top = pq.poll();
    int u = top[0], d = top[1];
    if (d > dist[u]) continue;          // stale — already finalized cheaper
    for (int[] e : adj.get(u)) {
        int nd = d + e[1];
        if (nd < dist[e[0]]) { dist[e[0]] = nd; pq.offer(new int[]{e[0], nd}); }
    }
}
```

## 6. 0-1 BFS with a `Deque`

When edge weights are only 0 or 1, replace Dijkstra's heap with a deque: push 0-weight moves to the **front**, 1-weight to the **back**. O(V+E).

```java
Deque<int[]> dq = new ArrayDeque<>();
dq.offerFirst(new int[]{src, 0});
while (!dq.isEmpty()) {
    int[] cur = dq.pollFirst();
    int u = cur[0], d = cur[1];
    if (d > dist[u]) continue;
    for (int[] e : adj.get(u)) {
        int nd = d + e[1];              // e[1] is 0 or 1
        if (nd < dist[e[0]]) {
            dist[e[0]] = nd;
            if (e[1] == 0) dq.offerFirst(new int[]{e[0], nd});
            else           dq.offerLast(new int[]{e[0], nd});
        }
    }
}
```

## 7. DSU with path compression + union by rank

The canonical near-O(α(n)) disjoint-set. Memorize it; it appears in Union-Find, MST (Kruskal), and cycle detection.

```java
class DSU {
    int[] parent, rank;
    DSU(int n) {
        parent = new int[n]; rank = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
    }
    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; } // compression
        return x;
    }
    boolean union(int a, int b) {
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;     // already connected (cycle for undirected)
        if (rank[ra] < rank[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        if (rank[ra] == rank[rb]) rank[ra]++;
        return true;
    }
}
```

## 8. 3-color DFS for directed cycle detection

WHITE (unvisited) / GRAY (on the current DFS stack) / BLACK (done). A GRAY→GRAY edge is a back edge = cycle. Do **not** use parent-tracking for directed graphs (that's the undirected technique).

```java
int[] color;   // 0 white, 1 gray, 2 black
boolean hasCycle(int u, List<List<Integer>> adj) {
    color[u] = 1;
    for (int v : adj.get(u)) {
        if (color[v] == 1) return true;        // back edge → cycle
        if (color[v] == 0 && hasCycle(v, adj)) return true;
    }
    color[u] = 2;
    return false;
}
```

## 9. Kahn's indegree topological sort

BFS over zero-indegree nodes. If you can't process all nodes, the graph has a cycle.

```java
int[] indegree = new int[n];
for (int[] e : edges) indegree[e[1]]++;        // edge u->v
Queue<Integer> q = new ArrayDeque<>();
for (int i = 0; i < n; i++) if (indegree[i] == 0) q.offer(i);
List<Integer> order = new ArrayList<>();
while (!q.isEmpty()) {
    int u = q.poll();
    order.add(u);
    for (int v : adj.get(u)) if (--indegree[v] == 0) q.offer(v);
}
boolean acyclic = order.size() == n;           // else cycle
```

## 10. Bitmask state for "visit all nodes"

When the answer depends on *which subset* of nodes you've visited (Shortest Path Visiting All Nodes), make the BFS state `(node, mask)`. Goal: `mask == (1 << n) - 1`.

```java
int full = (1 << n) - 1;
Queue<int[]> q = new ArrayDeque<>();           // {node, mask}
boolean[][] seen = new boolean[n][1 << n];
for (int i = 0; i < n; i++) { q.offer(new int[]{i, 1 << i}); seen[i][1 << i] = true; }
int steps = 0;
while (!q.isEmpty()) {
    int size = q.size();
    for (int s = 0; s < size; s++) {
        int[] cur = q.poll();
        if (cur[1] == full) return steps;
        for (int nb : adj.get(cur[0])) {
            int mask = cur[1] | (1 << nb);
            if (!seen[nb][mask]) { seen[nb][mask] = true; q.offer(new int[]{nb, mask}); }
        }
    }
    steps++;
}
```

> **Last Updated:** 2026-06-26
