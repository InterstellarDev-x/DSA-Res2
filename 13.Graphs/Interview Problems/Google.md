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

```rust
use std::collections::{HashMap, HashSet, VecDeque};

fn alien_order(words: &[String]) -> String {
    let mut adj: HashMap<char, HashSet<char>> = HashMap::new();
    let mut indegree: HashMap<char, i32> = HashMap::new();
    for w in words {
        for c in w.chars() {
            adj.entry(c).or_insert_with(HashSet::new);
            indegree.entry(c).or_insert(0); // every char is a node
        }
    }
    for i in 0..words.len().saturating_sub(1) {
        let a = &words[i];
        let b = &words[i + 1];
        let len = a.len().min(b.len());
        let a_bytes = a.as_bytes();
        let b_bytes = b.as_bytes();
        let mut j = 0;
        while j < len && a_bytes[j] == b_bytes[j] {
            j += 1;
        }
        if j == len {
            if a.len() > b.len() {
                return String::new(); // prefix conflict
            }
        } else {
            let from = a_bytes[j] as char;
            let to = b_bytes[j] as char;
            if adj.entry(from).or_insert_with(HashSet::new).insert(to) {
                *indegree.entry(to).or_insert(0) += 1;
            }
        }
    }
    let mut q: VecDeque<char> = VecDeque::new();
    for (&c, &deg) in &indegree {
        if deg == 0 {
            q.push_back(c);
        }
    }
    let mut sb = String::new();
    while let Some(c) = q.pop_front() {
        sb.push(c);
        if let Some(neighbors) = adj.get(&c) {
            let neighbors: Vec<char> = neighbors.iter().cloned().collect();
            for next in neighbors {
                let deg = indegree.get_mut(&next).unwrap();
                *deg -= 1;
                if *deg == 0 {
                    q.push_back(next);
                }
            }
        }
    }
    if sb.len() == indegree.len() { sb } else { String::new() } // cycle ⇒ ""
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

```rust
fn dfs(
    u: usize,
    parent: i32,
    adj: &Vec<Vec<usize>>,
    disc: &mut Vec<i32>,
    low: &mut Vec<i32>,
    timer: &mut i32,
    bridges: &mut Vec<Vec<i32>>,
) {
    disc[u] = *timer;
    low[u] = *timer;
    *timer += 1;
    for &v in &adj[u] {
        if v as i32 == parent {
            continue; // skip the edge we came in on
        }
        if disc[v] == -1 {
            dfs(v, u as i32, adj, disc, low, timer, bridges);
            low[u] = low[u].min(low[v]);
            if low[v] > disc[u] {
                bridges.push(vec![u as i32, v as i32]); // bridge test
            }
        } else {
            low[u] = low[u].min(disc[v]); // back edge
        }
    }
}

fn critical_connections(n: i32, connections: Vec<Vec<i32>>) -> Vec<Vec<i32>> {
    let n = n as usize;
    let mut adj: Vec<Vec<usize>> = vec![vec![]; n];
    for e in &connections {
        adj[e[0] as usize].push(e[1] as usize);
        adj[e[1] as usize].push(e[0] as usize);
    }
    let mut disc = vec![-1i32; n];
    let mut low = vec![0i32; n];
    let mut bridges: Vec<Vec<i32>> = Vec::new();
    let mut timer = 0i32;
    dfs(0, -1, &adj, &mut disc, &mut low, &mut timer, &mut bridges);
    bridges
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

```rust
fn min_cost_connect_points(points: Vec<Vec<i32>>) -> i32 {
    let n = points.len();
    let mut in_mst = vec![false; n];
    let mut min_dist = vec![i32::MAX; n];
    min_dist[0] = 0;
    let mut total = 0;
    for _ in 0..n {
        // pick cheapest outside vertex
        let u = (0..n)
            .filter(|&v| !in_mst[v])
            .min_by_key(|&v| min_dist[v])
            .unwrap();
        in_mst[u] = true;
        total += min_dist[u];
        for v in 0..n {
            // relax neighbors
            if in_mst[v] {
                continue;
            }
            let d = (points[u][0] - points[v][0]).abs()
                  + (points[u][1] - points[v][1]).abs();
            if d < min_dist[v] {
                min_dist[v] = d;
            }
        }
    }
    total
}
```

### Follow-ups — *Connecting Cities With Minimum Cost*; *Optimize Water Distribution* (virtual node for wells); *Find Critical & Pseudo-Critical Edges* (Kruskal with forced/excluded edges).

---

## 4. Word Ladder II (LC 126) — BFS layering + parent backtrack

Return **all** shortest transformation sequences.

### The two-phase idea

A single BFS finds the shortest length but not the paths. The clean approach: **BFS to build a parent (predecessor) map level by level**, ensuring we only keep predecessors from the *previous* layer (so all recorded paths are shortest), then **DFS-backtrack** from `endWord` to `beginWord` to reconstruct every path.

```rust
use std::collections::{HashMap, HashSet, VecDeque};

fn backtrack(
    word: &str,
    begin: &str,
    parents: &HashMap<String, Vec<String>>,
    path: &mut VecDeque<String>,
    res: &mut Vec<Vec<String>>,
) {
    path.push_front(word.to_string());
    if word == begin {
        res.push(path.iter().cloned().collect());
    } else if let Some(preds) = parents.get(word) {
        let preds = preds.clone();
        for p in &preds {
            backtrack(p, begin, parents, path, res);
        }
    }
    path.pop_front();
}

fn find_ladders(begin_word: String, end_word: String, word_list: Vec<String>) -> Vec<Vec<String>> {
    let mut dict: HashSet<String> = word_list.into_iter().collect();
    let mut res: Vec<Vec<String>> = Vec::new();
    if !dict.contains(&end_word) {
        return res;
    }
    let mut parents: HashMap<String, Vec<String>> = HashMap::new();
    let mut level: HashSet<String> = HashSet::new();
    level.insert(begin_word.clone());
    let mut found = false;
    while !level.is_empty() && !found {
        for w in &level {
            dict.remove(w); // remove this layer from dict
        }
        let mut next: HashSet<String> = HashSet::new();
        for word in &level {
            let mut arr: Vec<u8> = word.as_bytes().to_vec();
            for i in 0..arr.len() {
                let old = arr[i];
                for c in b'a'..=b'z' {
                    arr[i] = c;
                    let candidate = String::from_utf8(arr.clone()).unwrap();
                    if dict.contains(&candidate) {
                        if candidate == end_word {
                            found = true;
                        }
                        next.insert(candidate.clone());
                        parents
                            .entry(candidate)
                            .or_insert_with(Vec::new)
                            .push(word.clone());
                    }
                }
                arr[i] = old; // restore after all substitutions at position i
            }
        }
        level = next;
    }
    if found {
        let mut path: VecDeque<String> = VecDeque::new();
        backtrack(&end_word, &begin_word, &parents, &mut path, &mut res);
    }
    res
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
