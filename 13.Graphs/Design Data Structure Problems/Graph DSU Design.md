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

- **Merge the sets containing `a` and `b`.** (`union`)

### Why it matters for graphs

1. **Dynamic connectivity.** As edges are added one at a time, DSU tells you
   in near constant time whether two vertices are already connected. This is
   the backbone of **Kruskal's MST** (skip an edge if its endpoints are
   already connected, otherwise union them) and of detecting cycles in an
   undirected graph.
2. **Grouping / clustering.** Problems that ask for the *number of connected
   components*, *number of provinces*, *number of islands*, *accounts merge*,
   *friend circles*, or *redundant connection* are all DSU in disguise. You
   union related items and read off `count()` or group by representative.
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

```java
/**
 * Disjoint Set Union (Union-Find) over the integer universe [0, n).
 *
 * <p>Supports near-constant-time {@code union} and {@code find} via
 * path compression and union by rank. Also tracks the current number of
 * disjoint components.
 */
public class UnionFind {

    /** parent[x] is the parent of x; a root r satisfies parent[r] == r. */
    private final int[] parent;

    /** rank[x] is an upper bound on the height of the tree rooted at x. */
    private final int[] rank;

    /** Current number of disjoint sets (components). */
    private int count;

    /**
     * Creates a structure with {@code n} singleton sets {0}, {1}, ..., {n-1}.
     *
     * @param n the number of elements; must be non-negative
     * @throws IllegalArgumentException if {@code n < 0}
     */
    public UnionFind(int n) {
        if (n < 0) {
            throw new IllegalArgumentException("n must be non-negative: " + n);
        }
        this.parent = new int[n];
        this.rank = new int[n];
        this.count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;   // each element is its own root initially
            rank[i] = 0;
        }
    }

    /**
     * Returns the representative (root) of the set containing {@code x},
     * applying iterative path compression so that every node on the path
     * points directly at the root afterwards.
     *
     * @param x an element in [0, n)
     * @return the root of x's set
     * @throws IndexOutOfBoundsException if x is out of range
     */
    public int find(int x) {
        // First pass: locate the root.
        int root = x;
        while (root != parent[root]) {
            root = parent[root];
        }
        // Second pass: compress the path by repointing each node to root.
        while (x != root) {
            int next = parent[x];
            parent[x] = root;
            x = next;
        }
        return root;
    }

    /**
     * Merges the sets containing {@code a} and {@code b} using union by rank.
     *
     * @param a an element in [0, n)
     * @param b an element in [0, n)
     * @return {@code true} if a and b were in different sets and a merge
     *         occurred; {@code false} if they were already connected
     */
    public boolean union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false;   // already in the same set, nothing to do
        }
        // Attach the lower-rank tree under the higher-rank tree.
        if (rank[rootA] < rank[rootB]) {
            parent[rootA] = rootB;
        } else if (rank[rootA] > rank[rootB]) {
            parent[rootB] = rootA;
        } else {
            parent[rootB] = rootA;
            rank[rootA]++;   // equal ranks: pick one as root and bump its rank
        }
        count--;
        return true;
    }

    /**
     * Reports whether {@code a} and {@code b} belong to the same set.
     *
     * @param a an element in [0, n)
     * @param b an element in [0, n)
     * @return {@code true} if connected, otherwise {@code false}
     */
    public boolean connected(int a, int b) {
        return find(a) == find(b);
    }

    /**
     * Returns the current number of disjoint components.
     *
     * @return the number of sets
     */
    public int count() {
        return count;
    }
}
```

### Example: counting connected components

```java
// Graph with 5 vertices and edges (0-1), (1-2), (3-4).
UnionFind uf = new UnionFind(5);
uf.union(0, 1);
uf.union(1, 2);
uf.union(3, 4);

System.out.println(uf.count());          // 2  -> {0,1,2} and {3,4}
System.out.println(uf.connected(0, 2));  // true
System.out.println(uf.connected(0, 3));  // false
```

---

## 2. Generic `UnionFind<T>` for arbitrary nodes

When nodes are strings, longs, or custom objects (e.g. account emails,
city names, coordinate pairs), back the structure with hash maps instead of
arrays. Elements are created lazily on first reference.

```java
import java.util.HashMap;
import java.util.Map;

/**
 * Generic Disjoint Set Union over an arbitrary element type {@code T}.
 *
 * <p>Elements are introduced lazily: any element passed to {@code makeSet},
 * {@code find}, or {@code union} that is not yet known becomes a new
 * singleton set. {@code T} must have well-behaved {@code equals}/{@code hashCode}.
 *
 * @param <T> the element type
 */
public class UnionFind<T> {

    /** Maps each element to its parent; a root maps to itself. */
    private final Map<T, T> parent = new HashMap<>();

    /** Maps each root candidate to its rank (tree height upper bound). */
    private final Map<T, Integer> rank = new HashMap<>();

    /** Current number of disjoint sets. */
    private int count = 0;

    /**
     * Ensures {@code x} exists as a singleton set if it is not already known.
     *
     * @param x the element to register
     */
    public void makeSet(T x) {
        if (!parent.containsKey(x)) {
            parent.put(x, x);
            rank.put(x, 0);
            count++;
        }
    }

    /**
     * Returns the representative of {@code x}'s set, registering {@code x}
     * first if needed, with full path compression.
     *
     * @param x the element to look up
     * @return the root element of x's set
     */
    public T find(T x) {
        makeSet(x);
        // Locate the root.
        T root = x;
        while (!root.equals(parent.get(root))) {
            root = parent.get(root);
        }
        // Compress the path.
        T cur = x;
        while (!cur.equals(root)) {
            T next = parent.get(cur);
            parent.put(cur, root);
            cur = next;
        }
        return root;
    }

    /**
     * Merges the sets containing {@code a} and {@code b} by rank.
     *
     * @param a first element
     * @param b second element
     * @return {@code true} if a merge occurred, {@code false} if already joined
     */
    public boolean union(T a, T b) {
        T rootA = find(a);
        T rootB = find(b);
        if (rootA.equals(rootB)) {
            return false;
        }
        int rankA = rank.get(rootA);
        int rankB = rank.get(rootB);
        if (rankA < rankB) {
            parent.put(rootA, rootB);
        } else if (rankA > rankB) {
            parent.put(rootB, rootA);
        } else {
            parent.put(rootB, rootA);
            rank.put(rootA, rankA + 1);
        }
        count--;
        return true;
    }

    /**
     * Reports whether {@code a} and {@code b} are in the same set.
     *
     * @param a first element
     * @param b second element
     * @return {@code true} if connected
     */
    public boolean connected(T a, T b) {
        return find(a).equals(find(b));
    }

    /**
     * Returns the current number of disjoint sets.
     *
     * @return the component count
     */
    public int count() {
        return count;
    }
}
```

### Example: grouping strings

```java
UnionFind<String> uf = new UnionFind<>();
uf.union("alice@a.com", "alice@b.com");
uf.union("bob@x.com",   "bob@y.com");

System.out.println(uf.connected("alice@a.com", "alice@b.com")); // true
System.out.println(uf.connected("alice@a.com", "bob@x.com"));   // false
System.out.println(uf.count());                                 // 2
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
| `union(a, b)`            | O(α(n)) amortized | O(α(n)) amortized*     | two finds + a constant merge |
| `connected(a, b)`        | O(α(n)) amortized | O(α(n)) amortized*     | two finds |
| `count()`                | O(1)              | O(1)                   | maintained incrementally |
| Construction / `makeSet` | O(n) / O(1)       | O(1) per element       | array init vs. lazy insert |
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
the product of edge weights while compressing, and `union(a, b, ratio)` records
`a / b == ratio`.

```java
import java.util.HashMap;
import java.util.Map;

/**
 * Weighted (relational) Disjoint Set Union for ratio constraints of the form
 * {@code value(a) / value(b) == ratio}.
 *
 * <p>{@code weight.get(x)} stores {@code value(x) / value(parent.get(x))}.
 * Path compression multiplies weights along the path so each node ends up
 * storing its ratio directly to the root.
 */
public class WeightedUnionFind {

    private final Map<String, String> parent = new HashMap<>();
    private final Map<String, Double> weight = new HashMap<>();

    /** Registers {@code x} as a singleton with ratio 1.0 to itself. */
    public void makeSet(String x) {
        if (!parent.containsKey(x)) {
            parent.put(x, x);
            weight.put(x, 1.0);
        }
    }

    /**
     * Finds the root of {@code x} while accumulating the multiplicative weight
     * from {@code x} up to the root, and compresses the path so that
     * {@code weight.get(x)} becomes {@code value(x) / value(root)}.
     *
     * @param x the element to look up
     * @return the root of x's set
     */
    public String find(String x) {
        makeSet(x);
        if (parent.get(x).equals(x)) {
            return x;
        }
        String p = parent.get(x);
        String root = find(p);                 // recurse: now weight[p] = p/root
        weight.put(x, weight.get(x) * weight.get(p)); // x/root = x/p * p/root
        parent.put(x, root);                   // point directly at root
        return root;
    }

    /**
     * Records the relation {@code value(a) / value(b) == ratio} by merging the
     * two sets. If they are already connected this is treated as consistent
     * input and ignored.
     *
     * @param a     numerator element
     * @param b     denominator element
     * @param ratio the value of value(a) / value(b)
     */
    public void union(String a, String b, double ratio) {
        makeSet(a);
        makeSet(b);
        String rootA = find(a);
        String rootB = find(b);
        if (rootA.equals(rootB)) {
            return; // already related (assumes consistent input)
        }
        // We need: value(a)/value(b) = ratio.
        // weight[a] = a/rootA, weight[b] = b/rootB.
        // Attach rootA under rootB. Set weight[rootA] = rootA/rootB so that
        //   a/b = (a/rootA) * (rootA/rootB) * (rootB/b) = ratio
        // => rootA/rootB = ratio * (b/rootB) / (a/rootA)
        //               = ratio * weight[b] / weight[a]
        weight.put(rootA, ratio * weight.get(b) / weight.get(a));
        parent.put(rootA, rootB);
    }

    /**
     * Returns {@code value(a) / value(b)} if both are known and connected,
     * otherwise {@code -1.0} (the LeetCode "Evaluate Division" sentinel).
     *
     * @param a numerator element
     * @param b denominator element
     * @return the ratio, or -1.0 if undeterminable
     */
    public double query(String a, String b) {
        if (!parent.containsKey(a) || !parent.containsKey(b)) {
            return -1.0;
        }
        String rootA = find(a);
        String rootB = find(b);
        if (!rootA.equals(rootB)) {
            return -1.0;
        }
        // a/b = (a/root) / (b/root) = weight[a] / weight[b]
        return weight.get(a) / weight.get(b);
    }
}
```

```java
// a/b = 2, b/c = 3  =>  a/c = 6, c/a = 1/6, x unknown.
WeightedUnionFind wuf = new WeightedUnionFind();
wuf.union("a", "b", 2.0);
wuf.union("b", "c", 3.0);

System.out.println(wuf.query("a", "c"));  // 6.0
System.out.println(wuf.query("c", "a"));  // 0.1666...
System.out.println(wuf.query("a", "x"));  // -1.0  (x not present)
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
  each `union` changes at most one `parent` entry and one `rank` entry, so it is
  cheaply reversible. The trade-off is **O(log n)** per `find` instead of
  inverse Ackermann.
- A **history stack** recording exactly what each `union` changed.

```java
import java.util.ArrayDeque;
import java.util.Deque;

/**
 * Rollback (undoable) Disjoint Set Union using union by rank WITHOUT path
 * compression. Every successful {@code union} can be reversed in O(1) via
 * {@code rollback()}, in last-in-first-out order.
 *
 * <p>{@code find} is O(log n) because path compression is intentionally
 * omitted to keep unions undoable.
 */
public class RollbackUnionFind {

    private final int[] parent;
    private final int[] rank;
    private int count;

    /**
     * One reversible change: the root that was attached, plus whether the
     * surviving root's rank was incremented during this union.
     */
    private static final class Change {
        final int attachedRoot;       // root whose parent was changed
        final boolean rankIncreased;  // did the surviving root's rank go up?
        final int survivingRoot;      // root that remained a root

        Change(int attachedRoot, int survivingRoot, boolean rankIncreased) {
            this.attachedRoot = attachedRoot;
            this.survivingRoot = survivingRoot;
            this.rankIncreased = rankIncreased;
        }
    }

    private final Deque<Change> history = new ArrayDeque<>();

    /**
     * Creates {@code n} singleton sets {0}, ..., {n-1}.
     *
     * @param n the number of elements
     */
    public RollbackUnionFind(int n) {
        parent = new int[n];
        rank = new int[n];
        count = n;
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }
    }

    /**
     * Returns the root of {@code x} WITHOUT path compression.
     *
     * @param x the element to look up
     * @return the root of x's set
     */
    public int find(int x) {
        while (x != parent[x]) {
            x = parent[x];
        }
        return x;
    }

    /**
     * Merges the sets of {@code a} and {@code b} by rank, pushing the change
     * onto the history stack so it can be undone.
     *
     * @param a first element
     * @param b second element
     * @return {@code true} if a merge happened (a rollback point was pushed),
     *         {@code false} if already connected (nothing pushed)
     */
    public boolean union(int a, int b) {
        int rootA = find(a);
        int rootB = find(b);
        if (rootA == rootB) {
            return false; // no change recorded
        }
        // Ensure rootA is the higher (or equal) rank root.
        if (rank[rootA] < rank[rootB]) {
            int tmp = rootA;
            rootA = rootB;
            rootB = tmp;
        }
        // Attach rootB under rootA.
        boolean rankIncreased = (rank[rootA] == rank[rootB]);
        parent[rootB] = rootA;
        if (rankIncreased) {
            rank[rootA]++;
        }
        count--;
        history.push(new Change(rootB, rootA, rankIncreased));
        return true;
    }

    /**
     * Undoes the most recent successful {@code union}. Call only when at least
     * one union is on the history stack.
     *
     * @throws java.util.NoSuchElementException if there is nothing to roll back
     */
    public void rollback() {
        Change c = history.pop();
        parent[c.attachedRoot] = c.attachedRoot; // detach: make it a root again
        if (c.rankIncreased) {
            rank[c.survivingRoot]--;              // undo the rank bump
        }
        count++;
    }

    /**
     * Reports whether {@code a} and {@code b} are currently connected.
     *
     * @param a first element
     * @param b second element
     * @return {@code true} if connected
     */
    public boolean connected(int a, int b) {
        return find(a) == find(b);
    }

    /** @return the current number of components. */
    public int count() {
        return count;
    }

    /** @return the number of undoable unions currently on the history stack. */
    public int historySize() {
        return history.size();
    }
}
```

```java
RollbackUnionFind ruf = new RollbackUnionFind(4);
ruf.union(0, 1);
ruf.union(2, 3);
System.out.println(ruf.count());          // 2
ruf.union(1, 2);
System.out.println(ruf.connected(0, 3));  // true,  count == 1
ruf.rollback();                            // undo union(1, 2)
System.out.println(ruf.connected(0, 3));  // false, count == 2
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
