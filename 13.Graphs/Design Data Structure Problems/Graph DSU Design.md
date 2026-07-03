> **Topic:** [Graphs](../README.md) · **Design Data Structure**

# Graph DSU Design

## Overview

A **Disjoint Set Union (DSU)**, also known as **Union-Find**, is one of the
most reusable data structures in graph problem solving. It maintains a
partition of a universe of elements into disjoint sets and answers two
questions extremely fast:

- **Which set does element `x` belong to?** (`find`)
- **Are `a` and `b` in the same set?** (`connected`)

while also supporting the mutation:

- **Merge the sets containing `a` and `b`.** (`unite`)

### Why it matters for graphs

1. **Dynamic connectivity.** As edges are added one at a time, DSU tells you
   in near constant time whether two vertices are already connected. This is
   the backbone of **Kruskal's MST** (skip an edge if its endpoints are
   already connected, otherwise unite them) and of detecting cycles in an
   undirected graph.
2. **Grouping / clustering.** Problems that ask for the *number of connected
   components*, *number of provinces*, *number of islands*, *accounts merge*,
   *friend circles*, or *redundant connection* are all DSU in disguise. You
   unite related items and read off `count()` or group by representative.
3. **Online vs. offline.** Plain DSU is an *incremental* (online) structure:
   it handles unions and queries interleaved. With a **rollback** variant you
   can also support *offline dynamic connectivity* where edges are added and
   removed (segment-tree-on-time + rollback DSU).
4. **Relational constraints.** A **weighted DSU** stores the relationship of
   each node to its representative (an offset or a ratio), which lets you
   answer "what is the relation between `a` and `b`?" — used for *Evaluate
   Division*, *Satisfiability of Equality Equations*, and parity/2-coloring
   problems.

Two optimizations make DSU practically constant time:

- **Path compression** in `find`: flatten the tree so future lookups are
  direct.
- **Union by rank** (or by size): always attach the shorter tree under the
  taller one to keep trees flat.

Together they give an amortized cost of **O(α(n))** per operation, where
`α` is the inverse Ackermann function — effectively a small constant (≤ 4)
for any input that fits in the universe.

---

## 1. Production `UnionFind` for integer nodes `[0, n)`

```rust
// Disjoint Set Union (Union-Find) over the integer universe [0, n).
//
// Supports near-constant-time unite and find via
// path compression and union by rank. Also tracks the current number of
// disjoint components.
pub struct UnionFind {
    // parent[x] is the parent of x; a root r satisfies parent[r] == r.
    parent: Vec<usize>,

    // rnk[x] is an upper bound on the height of the tree rooted at x.
    rnk: Vec<usize>,

    // Current number of disjoint sets (components).
    cnt: usize,
}

impl UnionFind {
    // Creates a structure with n singleton sets {0}, {1}, ..., {n-1}.
    // n: the number of elements
    pub fn new(n: usize) -> Self {
        UnionFind {
            parent: (0..n).collect(),
            rnk: vec![0; n],
            cnt: n,
        }
    }

    // Returns the representative (root) of the set containing x,
    // applying iterative path compression so that every node on the path
    // points directly at the root afterwards.
    pub fn find(&mut self, x: usize) -> usize {
        // First pass: locate the root.
        let mut root = x;
        while root != self.parent[root] {
            root = self.parent[root];
        }
        // Second pass: compress the path by repointing each node to root.
        let mut cur = x;
        while cur != root {
            let next = self.parent[cur];
            self.parent[cur] = root;
            cur = next;
        }
        root
    }

    // Merges the sets containing a and b using union by rank.
    // Returns true if a and b were in different sets and a merge
    // occurred; false if they were already connected.
    pub fn unite(&mut self, a: usize, b: usize) -> bool {
        let root_a = self.find(a);
        let root_b = self.find(b);
        if root_a == root_b {
            return false; // already in the same set, nothing to do
        }
        // Attach the lower-rank tree under the higher-rank tree.
        if self.rnk[root_a] < self.rnk[root_b] {
            self.parent[root_a] = root_b;
        } else if self.rnk[root_a] > self.rnk[root_b] {
            self.parent[root_b] = root_a;
        } else {
            self.parent[root_b] = root_a;
            self.rnk[root_a] += 1; // equal ranks: pick one as root and bump its rank
        }
        self.cnt -= 1;
        true
    }

    // Reports whether a and b belong to the same set.
    pub fn connected(&mut self, a: usize, b: usize) -> bool {
        self.find(a) == self.find(b)
    }

    // Returns the current number of disjoint components.
    pub fn count(&self) -> usize {
        self.cnt
    }
}
```

### Example: counting connected components

```rust
// Graph with 5 vertices and edges (0-1), (1-2), (3-4).
let mut uf = UnionFind::new(5);
uf.unite(0, 1);
uf.unite(1, 2);
uf.unite(3, 4);

println!("{}", uf.count());          // 2  -> {0,1,2} and {3,4}
println!("{}", uf.connected(0, 2));  // true
println!("{}", uf.connected(0, 3));  // false
```

---

## 2. Generic `UnionFind<T>` for arbitrary nodes

When nodes are strings, longs, or custom objects (e.g. account emails,
city names, coordinate pairs), back the structure with hash maps instead of
arrays. Elements are created lazily on first reference.

```rust
use std::collections::HashMap;
use std::hash::Hash;

// Generic Disjoint Set Union over an arbitrary element type T.
//
// Elements are introduced lazily: any element passed to make_set,
// find, or unite that is not yet known becomes a new
// singleton set. T must implement Eq, Hash, and Clone.
pub struct UnionFind<T: Eq + Hash + Clone> {
    // Maps each element to its parent; a root maps to itself.
    parent: HashMap<T, T>,

    // Maps each root candidate to its rank (tree height upper bound).
    rnk: HashMap<T, usize>,

    // Current number of disjoint sets.
    cnt: usize,
}

impl<T: Eq + Hash + Clone> UnionFind<T> {
    pub fn new() -> Self {
        UnionFind {
            parent: HashMap::new(),
            rnk: HashMap::new(),
            cnt: 0,
        }
    }

    // Ensures x exists as a singleton set if it is not already known.
    pub fn make_set(&mut self, x: &T) {
        if !self.parent.contains_key(x) {
            self.parent.insert(x.clone(), x.clone());
            self.rnk.insert(x.clone(), 0);
            self.cnt += 1;
        }
    }

    // Returns the representative of x's set, registering x
    // first if needed, with full path compression.
    pub fn find(&mut self, x: &T) -> T {
        self.make_set(x);
        // Collect path from x up to (but not including) the root.
        let mut path: Vec<T> = Vec::new();
        let mut cur = x.clone();
        loop {
            let p = self.parent[&cur].clone();
            if p == cur {
                break; // cur is the root
            }
            path.push(cur);
            cur = p;
        }
        let root = cur;
        // Compress the path: point every node directly at the root.
        for node in &path {
            self.parent.insert(node.clone(), root.clone());
        }
        root
    }

    // Merges the sets containing a and b by rank.
    // Returns true if a merge occurred, false if already joined.
    pub fn unite(&mut self, a: &T, b: &T) -> bool {
        let root_a = self.find(a);
        let root_b = self.find(b);
        if root_a == root_b {
            return false;
        }
        let rank_a = self.rnk[&root_a];
        let rank_b = self.rnk[&root_b];
        if rank_a < rank_b {
            self.parent.insert(root_a, root_b);
        } else if rank_a > rank_b {
            self.parent.insert(root_b, root_a);
        } else {
            self.parent.insert(root_b.clone(), root_a.clone());
            self.rnk.insert(root_a, rank_a + 1);
        }
        self.cnt -= 1;
        true
    }

    // Reports whether a and b are in the same set.
    pub fn connected(&mut self, a: &T, b: &T) -> bool {
        self.find(a) == self.find(b)
    }

    // Returns the current number of disjoint sets.
    pub fn count(&self) -> usize {
        self.cnt
    }
}
```

### Example: grouping strings

```rust
let mut uf: UnionFind<String> = UnionFind::new();
uf.unite(&"alice@a.com".to_string(), &"alice@b.com".to_string());
uf.unite(&"bob@x.com".to_string(),   &"bob@y.com".to_string());

println!("{}", uf.connected(&"alice@a.com".to_string(), &"alice@b.com".to_string())); // true
println!("{}", uf.connected(&"alice@a.com".to_string(), &"bob@x.com".to_string()));   // false
println!("{}", uf.count());                                                            // 2
```

---

## 3. Complexity analysis

With both **path compression** and **union by rank**, any sequence of `m`
operations on `n` elements runs in **O(m · α(n))** total, i.e. essentially
**O(1) amortized** per operation. Here `α(n)` is the **inverse Ackermann
function**, which grows so slowly that `α(n) ≤ 4` for every `n` that could
ever be stored in practice (well beyond the number of atoms in the universe).

| Operation                | Array `UnionFind` | Generic `UnionFind<T>` | Notes |
|--------------------------|-------------------|------------------------|-------|
| `find(x)`                | O(α(n)) amortized | O(α(n)) amortized*     | `*` plus hash-map constant per node touched |
| `unite(a, b)`            | O(α(n)) amortized | O(α(n)) amortized*     | two finds + a constant merge |
| `connected(a, b)`        | O(α(n)) amortized | O(α(n)) amortized*     | two finds |
| `count()`                | O(1)              | O(1)                   | maintained incrementally |
| Construction / `make_set` | O(n) / O(1)      | O(1) per element       | array init vs. lazy insert |
| **Space**                | **O(n)**          | **O(n)**               | arrays vs. two hash maps |

> Without path compression *or* union by rank, a single `find` can degrade to
> **O(n)** in the worst case (a degenerate linked-list-shaped tree). Using only
> union by rank gives **O(log n)** per operation; adding path compression drops
> it to the inverse-Ackermann bound.

---

## 4. Follow-ups

### 4a. Weighted / relational DSU

Sometimes you do not just want to know *whether* `a` and `b` are related, but
*how*. Store a **weight (offset) from each node to its parent**. The classic
applications are:

- **Evaluate Division** (LeetCode 399): `weight[x]` = ratio `value(x) / value(parent(x))`, combined multiplicatively. `a / b` is the product of weights along the path.
- **Equation / parity problems**: `weight[x]` = additive offset (e.g. `value(x) - value(parent(x))`).

Below is a multiplicative-ratio version (Evaluate Division). `find` accumulates
the product of edge weights while compressing, and `unite(a, b, ratio)` records
`a / b == ratio`.

```rust
use std::collections::HashMap;

// Weighted (relational) Disjoint Set Union for ratio constraints of the form
// value(a) / value(b) == ratio.
//
// weight[x] stores value(x) / value(parent[x]).
// Path compression multiplies weights along the path so each node ends up
// storing its ratio directly to the root.
pub struct WeightedUnionFind {
    parent: HashMap<String, String>,
    weight: HashMap<String, f64>,
}

impl WeightedUnionFind {
    pub fn new() -> Self {
        WeightedUnionFind {
            parent: HashMap::new(),
            weight: HashMap::new(),
        }
    }

    // Registers x as a singleton with ratio 1.0 to itself.
    pub fn make_set(&mut self, x: &str) {
        if !self.parent.contains_key(x) {
            self.parent.insert(x.to_string(), x.to_string());
            self.weight.insert(x.to_string(), 1.0);
        }
    }

    // Finds the root of x while accumulating the multiplicative weight
    // from x up to the root, and compresses the path so that
    // weight[x] becomes value(x) / value(root).
    pub fn find(&mut self, x: &str) -> String {
        self.make_set(x);
        // Collect path from x up to (but not including) the root.
        let mut path: Vec<String> = Vec::new();
        let mut cur = x.to_string();
        loop {
            let p = self.parent[&cur].clone();
            if p == cur {
                break; // cur is the root
            }
            path.push(cur);
            cur = p;
        }
        let root = cur;
        if path.is_empty() {
            return root;
        }
        // Compute each node's weight relative to the root using suffix products.
        // path[i] / root = weight[path[i]] * weight[path[i+1]] * ... * weight[path[last]]
        let n = path.len();
        let mut weights_to_root = vec![0.0f64; n];
        weights_to_root[n - 1] = self.weight[&path[n - 1]]; // path[n-1] / root
        for i in (0..n - 1).rev() {
            weights_to_root[i] = self.weight[&path[i]] * weights_to_root[i + 1]; // x/root = x/p * p/root
        }
        // Apply path compression: point every node directly at root with updated weight.
        for i in 0..n {
            self.parent.insert(path[i].clone(), root.clone());
            self.weight.insert(path[i].clone(), weights_to_root[i]);
        }
        root
    }

    // Records the relation value(a) / value(b) == ratio by merging the
    // two sets. If they are already connected this is treated as consistent
    // input and ignored.
    // a: numerator element, b: denominator element, ratio: value(a) / value(b)
    pub fn unite(&mut self, a: &str, b: &str, ratio: f64) {
        self.make_set(a);
        self.make_set(b);
        let root_a = self.find(a);
        let root_b = self.find(b);
        if root_a == root_b {
            return; // already related (assumes consistent input)
        }
        // We need: value(a)/value(b) = ratio.
        // weight[a] = a/rootA, weight[b] = b/rootB.
        // Attach rootA under rootB. Set weight[rootA] = rootA/rootB so that
        //   a/b = (a/rootA) * (rootA/rootB) * (rootB/b) = ratio
        // => rootA/rootB = ratio * (b/rootB) / (a/rootA)
        //               = ratio * weight[b] / weight[a]
        let wa = self.weight[a];
        let wb = self.weight[b];
        self.weight.insert(root_a.clone(), ratio * wb / wa);
        self.parent.insert(root_a, root_b);
    }

    // Returns value(a) / value(b) if both are known and connected,
    // otherwise -1.0 (the LeetCode "Evaluate Division" sentinel).
    pub fn query(&mut self, a: &str, b: &str) -> f64 {
        if !self.parent.contains_key(a) || !self.parent.contains_key(b) {
            return -1.0;
        }
        let root_a = self.find(a);
        let root_b = self.find(b);
        if root_a != root_b {
            return -1.0;
        }
        // a/b = (a/root) / (b/root) = weight[a] / weight[b]
        self.weight[a] / self.weight[b]
    }
}
```

```rust
// a/b = 2, b/c = 3  =>  a/c = 6, c/a = 1/6, x unknown.
let mut wuf = WeightedUnionFind::new();
wuf.unite("a", "b", 2.0);
wuf.unite("b", "c", 3.0);

println!("{}", wuf.query("a", "c"));  // 6.0
println!("{}", wuf.query("c", "a"));  // 0.1666...
println!("{}", wuf.query("a", "x"));  // -1.0  (x not present)
```

> For **additive** problems (parity, "differences between variables") replace
> the multiplications with additions and the ratio with an offset, and `query`
> returns `weight[a] - weight[b]`.

### 4b. Rollback / persistent DSU

For **offline dynamic connectivity** (edges appear and disappear over time) or
**DSU on tree / small-to-large with undo**, you need to *undo* unions in
reverse order. This requires:

- **Union by rank/size only — no path compression.** Path compression mutates
  arbitrarily many nodes per `find`, which cannot be undone in O(1). Without it,
  each `unite` changes at most one `parent` entry and one `rank` entry, so it is
  cheaply reversible. The trade-off is **O(log n)** per `find` instead of
  inverse Ackermann.
- A **history stack** recording exactly what each `unite` changed.

```rust
// Rollback (undoable) Disjoint Set Union using union by rank WITHOUT path
// compression. Every successful unite can be reversed in O(1) via
// rollback(), in last-in-first-out order.
//
// find is O(log n) because path compression is intentionally
// omitted to keep unions undoable.

// One reversible change: the root that was attached, plus whether the
// surviving root's rank was incremented during this unite.
struct Change {
    attached_root:  usize, // root whose parent was changed
    surviving_root: usize, // root that remained a root
    rank_increased: bool,  // did the surviving root's rank go up?
}

pub struct RollbackUnionFind {
    parent: Vec<usize>,
    rnk:    Vec<usize>,
    cnt:    usize,
    // history stack (Vec used as LIFO stack via push/pop)
    history: Vec<Change>,
}

impl RollbackUnionFind {
    // Creates n singleton sets {0}, ..., {n-1}.
    pub fn new(n: usize) -> Self {
        RollbackUnionFind {
            parent:  (0..n).collect(),
            rnk:     vec![0; n],
            cnt:     n,
            history: Vec::new(),
        }
    }

    // Returns the root of x WITHOUT path compression.
    pub fn find(&self, mut x: usize) -> usize {
        while x != self.parent[x] {
            x = self.parent[x];
        }
        x
    }

    // Merges the sets of a and b by rank, pushing the change
    // onto the history stack so it can be undone.
    // Returns true if a merge happened (a rollback point was pushed),
    // false if already connected (nothing pushed).
    pub fn unite(&mut self, a: usize, b: usize) -> bool {
        let mut root_a = self.find(a);
        let mut root_b = self.find(b);
        if root_a == root_b {
            return false; // no change recorded
        }
        // Ensure root_a is the higher (or equal) rank root.
        if self.rnk[root_a] < self.rnk[root_b] {
            std::mem::swap(&mut root_a, &mut root_b);
        }
        // Attach root_b under root_a.
        let rank_increased = self.rnk[root_a] == self.rnk[root_b];
        self.parent[root_b] = root_a;
        if rank_increased {
            self.rnk[root_a] += 1;
        }
        self.cnt -= 1;
        self.history.push(Change { attached_root: root_b, surviving_root: root_a, rank_increased });
        true
    }

    // Undoes the most recent successful unite. Call only when at least
    // one unite is on the history stack.
    pub fn rollback(&mut self) {
        let c = self.history.pop().unwrap();
        self.parent[c.attached_root] = c.attached_root; // detach: make it a root again
        if c.rank_increased {
            self.rnk[c.surviving_root] -= 1; // undo the rank bump
        }
        self.cnt += 1;
    }

    // Reports whether a and b are currently connected.
    pub fn connected(&self, a: usize, b: usize) -> bool {
        self.find(a) == self.find(b)
    }

    // Returns the current number of components.
    pub fn count(&self) -> usize {
        self.cnt
    }

    // Returns the number of undoable unions currently on the history stack.
    pub fn history_size(&self) -> usize {
        self.history.len()
    }
}
```

```rust
let mut ruf = RollbackUnionFind::new(4);
ruf.unite(0, 1);
ruf.unite(2, 3);
println!("{}", ruf.count());          // 2
ruf.unite(1, 2);
println!("{}", ruf.connected(0, 3));  // true,  count == 1
ruf.rollback();                        // undo unite(1, 2)
println!("{}", ruf.connected(0, 3));  // false, count == 2
```

> A persistent/rollback DSU plugged into a **segment tree over the timeline**
> answers offline dynamic-connectivity queries (edges with [start, end] life
> spans) in **O((n + q) log q · log n)**.

---

## 5. When to pick which variant

| Variant                  | Node type        | `find` cost     | Path compression | Undo support | Typical problems |
|--------------------------|------------------|-----------------|------------------|--------------|------------------|
| `UnionFind` (array)      | ints `[0, n)`    | O(α(n)) amort.  | Yes              | No           | Kruskal MST, #provinces, cycle detection, redundant connection |
| `UnionFind<T>` (generic) | any hashable `T` | O(α(n)) amort.* | Yes              | No           | Accounts merge, string/coordinate grouping, sentence similarity |
| `WeightedUnionFind`      | relational       | O(α(n)) amort.  | Yes (weighted)   | No           | Evaluate Division, equation equality, parity / 2-color checks |
| `RollbackUnionFind`      | ints `[0, n)`    | O(log n)        | No (required)    | Yes          | Offline dynamic connectivity, DSU on tree, MST with edge deletions |

**Rules of thumb**

- Default to the **array `UnionFind`** whenever node IDs are small dense
  integers — it is the fastest and simplest.
- Reach for **`UnionFind<T>`** only when relabeling to integers is awkward;
  otherwise map your labels to `0..n-1` once and use the array version.
- Use **`WeightedUnionFind`** when you must answer *relationships*, not just
  membership.
- Use **`RollbackUnionFind`** only when you genuinely need to remove edges /
  undo merges — you pay an extra `log n` factor and lose path compression.

> **Last Updated:** 2026-06-26
