> **Topic:** [Graphs](../README.md) · **Section:** Interview Tips · **Tips 2 of 4**

# Graphs — Common Mistakes

Twelve recurring bugs as wrong/correct pairs. Each costs correctness or a TLE in real interviews. See also [Coding Tips](./Coding%20Tips.md), [Algorithm Selection](./Algorithm%20Selection.md), [Complexity Analysis](./Complexity%20Analysis.md).

---

## 1. Marking visited at dequeue instead of enqueue → TLE / duplicates

```java
// WRONG — same node enqueued by many neighbors before it's marked
while (!q.isEmpty()) {
    int u = q.poll();
    if (visited[u]) continue;       // marked too late
    visited[u] = true;
    for (int v : adj.get(u)) q.offer(v);
}
// CORRECT — mark the instant you enqueue
visited[start] = true; q.offer(start);
while (!q.isEmpty()) {
    int u = q.poll();
    for (int v : adj.get(u))
        if (!visited[v]) { visited[v] = true; q.offer(v); }
}
```

## 2. Subtraction in a PQ comparator → integer overflow

```java
// WRONG — a[1] - b[1] overflows when distances are large
new PriorityQueue<int[]>((a, b) -> a[1] - b[1]);
// CORRECT
new PriorityQueue<int[]>((a, b) -> Integer.compare(a[1], b[1]));
```

## 3. Forgetting the stale-entry skip in Dijkstra

```java
// WRONG — reprocesses finalized nodes, may relax with outdated d
int[] top = pq.poll();
for (int[] e : adj.get(top[0])) { /* relax using top[1] */ }
// CORRECT
int[] top = pq.poll();
int u = top[0], d = top[1];
if (d > dist[u]) continue;          // skip stale entry
for (int[] e : adj.get(u)) { /* relax using d */ }
```

## 4. Bellman-Ford for K-stops without cloning `dist`

```java
// WRONG — one round chains multiple edges → exceeds stop limit
for (int round = 0; round <= k; round++)
    for (int[] f : flights)
        dist[f[1]] = Math.min(dist[f[1]], dist[f[0]] + f[2]);
// CORRECT — freeze previous round's distances
for (int round = 0; round <= k; round++) {
    int[] prev = dist.clone();
    for (int[] f : flights)
        if (prev[f[0]] != Integer.MAX_VALUE)
            dist[f[1]] = Math.min(dist[f[1]], prev[f[0]] + f[2]);
}
```

## 5. Adding an undirected edge in only one direction

```java
// WRONG — half the graph is missing
adj.get(u).add(v);
// CORRECT
adj.get(u).add(v);
adj.get(v).add(u);
```

## 6. DSU without path compression → near-O(n) find, TLE

```java
// WRONG — degenerate chains
int find(int x) { while (parent[x] != x) x = parent[x]; return x; }
// CORRECT — path compression (+ union by rank)
int find(int x) { while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; } return x; }
```

## 7. Directed cycle detection via parent-tracking → wrong

Parent-skipping detects cycles in **undirected** graphs only. For directed graphs use 3-color.

```java
// WRONG for directed — misses cycles like a->b->a's true direction semantics
boolean dfs(int u, int parent) {
    for (int v : adj.get(u)) {
        if (v == parent) continue;          // undirected idea, wrong here
        if (visited[v] || dfs(v, u)) return true;
    }
    return true;
}
// CORRECT for directed — gray = on current path
boolean dfs(int u) {
    color[u] = 1;
    for (int v : adj.get(u)) {
        if (color[v] == 1) return true;     // back edge
        if (color[v] == 0 && dfs(v)) return true;
    }
    color[u] = 2;
    return false;
}
```

## 8. Kahn's without checking all nodes processed → misses cycle

```java
// WRONG — returns a partial "order" even with a cycle
while (!q.isEmpty()) { order.add(q.poll()); /* ... */ }
return order;
// CORRECT
return order.size() == n ? order : new ArrayList<>();   // cycle ⇒ empty
```

## 9. Grid bounds check ordered after the array read → out-of-bounds

```java
// WRONG — reads grid[r][c] before validating r,c
if (grid[r][c] == '1' && r >= 0 && r < m && c >= 0 && c < n) ...
// CORRECT — bounds FIRST, short-circuit protects the read
if (r >= 0 && r < m && c >= 0 && c < n && grid[r][c] == '1') ...
```

## 10. Modifying the grid during BFS without marking → revisits / infinite loop

```java
// WRONG — neighbor added repeatedly; never marked
for (int[] d : dirs) { int r=..., c=...; if (grid[r][c]==1) q.offer(new int[]{r,c}); }
// CORRECT — mark as you enqueue
for (int[] d : dirs) {
    int r=..., c=...;
    if (inBounds(r,c) && grid[r][c]==1) { grid[r][c] = 2; q.offer(new int[]{r,c}); }
}
```

## 11. Alien Dictionary missing the prefix-conflict check

```java
// WRONG — ["abc","ab"] should be invalid but slips through
int j = 0; while (j < len && a.charAt(j)==b.charAt(j)) j++;
if (j < len) addEdge(a.charAt(j), b.charAt(j));
// CORRECT
if (j == len) { if (a.length() > b.length()) return ""; }   // longer word before its prefix
else addEdge(a.charAt(j), b.charAt(j));
```

## 12. 8-direction vs 4-direction mix-up

Number of Islands / Rotting Oranges are **4-directional**. Using `dirs8` over-connects and inflates/merges islands. Conversely, problems like *Shortest Path in Binary Matrix* are **8-directional** — using 4 dirs gives wrong (too-long or no) paths. Always confirm connectivity from the prompt before choosing `dirs` vs `dirs8`.

```java
int[][] dirs  = {{1,0},{-1,0},{0,1},{0,-1}};                         // islands, rotting
int[][] dirs8 = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}}; // binary matrix path
```

> **Last Updated:** 2026-06-26
