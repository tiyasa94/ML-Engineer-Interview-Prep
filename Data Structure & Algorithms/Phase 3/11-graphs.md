# Graphs — Complete Theory, Patterns & Interview Mastery

**Phase 3 — Search Over Structures, Topic 11** | Practice problems: NeetCode's Graphs section, after this document, not before.

---

# Part A — What a Graph Actually Is

A graph is a set of **vertices** (nodes) connected by **edges**. **Directed** edges have a one-way relationship (`A → B` does not imply `B → A`); **undirected** edges are symmetric. **Weighted** edges carry a cost (Topic 12); here, edges are unweighted (cost 1, implicitly). A tree (Topic 7) is a special case of a graph: connected, acyclic, exactly `V-1` edges for `V` vertices.

## Two representations, a real tradeoff

| Representation | Space | Edge existence check | Iterate all neighbors |
|---|---|---|---|
| Adjacency matrix | O(V²) | O(1) | O(V) |
| Adjacency list | O(V+E) | O(degree) | O(degree) |

Adjacency **list** (a dict/array of lists) is the default for sparse graphs (`E` much less than `V²`, the common interview case). Adjacency **matrix** is worth reaching for when edge-existence checks dominate and the graph is dense, or when the graph is small enough that O(V²) space is a non-issue.

---

# Part B — Why Graph Traversal Needs More Care Than Tree Traversal

## The critical addition: a visited set

Tree DFS (Topic 7) never needs cycle protection — a tree has no cycles by definition. A **general graph can have cycles**, so DFS/BFS on a graph without a `visited` set can loop forever. This is the single structural difference that separates this topic from Topic 7, not a new traversal algorithm.

## BFS finds shortest paths in unweighted graphs — proved, not just asserted

**Claim**: in an unweighted graph, BFS from a source discovers every vertex in strictly non-decreasing order of distance from that source, and the first time a vertex is reached is via a shortest path to it.

**Proof sketch, by induction on distance**: all vertices at distance `0` (the source itself) are trivially discovered first. Assume all vertices at distance `d` have been discovered before any vertex at distance `d+1` (inductive hypothesis). BFS processes its queue in FIFO order, so every distance-`d` vertex is dequeued, and its neighbors — which are at distance `d+1` or possibly less (already visited) — get enqueued, before any distance-`d+1` vertex already in the queue is dequeued. So every distance-`(d+1)` vertex is discovered while processing distance-`d` vertices, before any distance-`(d+2)` vertex could be reached. This holds for all `d`, establishing the claim by induction. **This is exactly why BFS, not DFS, is the correct tool whenever "shortest path" and "unweighted" appear together** — DFS provides no such distance guarantee at all.

## Complexity: O(V + E)

Both DFS and BFS visit every vertex once (`O(V)`) and examine every edge once for a directed graph, or twice for undirected (`O(E)`) — giving `O(V + E)` total, the standard graph-traversal complexity baseline everything else in this topic is measured against.

## Topological sort — only meaningful for DAGs

A **topological order** is a linear ordering of vertices such that every directed edge `u -> v` has `u` appearing before `v`. This is only possible for a **Directed Acyclic Graph (DAG)** — a cycle would require some vertex to both precede and follow another, a contradiction. Two standard algorithms:
- **DFS-based**: run DFS, record each vertex when it *finishes* (postorder), then reverse that order. Correctness: a vertex only finishes after all vertices reachable from it have finished, so reversing guarantees every dependency appears before its dependents.
- **Kahn's algorithm (BFS-based)**: repeatedly remove vertices with in-degree 0 (no remaining unprocessed dependencies), decrementing the in-degree of their neighbors as they're removed. If not all vertices get removed by the end, a cycle exists — this doubles as a cycle-detection method.

## Cycle detection: directed graphs need three states, undirected need only two

- **Undirected**: a simple `visited` set plus tracking the parent (the node just arrived from) is sufficient — a cycle exists if a DFS encounters an already-visited neighbor that is **not** the immediate parent.
- **Directed**: two states (`visited`/`unvisited`) are **not sufficient** — a directed graph can have an edge to an already-fully-processed node without that indicating a cycle. The correct approach uses **three states**: unvisited, currently-being-processed ("on the current DFS path"), and fully-processed. A cycle exists specifically when DFS encounters an edge to a node that is currently-being-processed (a "back edge" to an ancestor still on the recursion stack) — this distinction is the most commonly missed piece of graph cycle-detection theory.

---

# Part C — The Core Patterns

### 1. DFS/BFS Traversal & Connected Components

```
def dfs(graph, start, visited):
    visited.add(start)
    for neighbor in graph[start]:
        if neighbor not in visited:
            dfs(graph, neighbor, visited)

def count_components(graph, vertices):
    visited = set()
    count = 0
    for v in vertices:
        if v not in visited:
            dfs(graph, v, visited)
            count += 1
    return count
```
**Signals:** "connected components", "number of provinces/islands/groups".

### 2. Grid as an Implicit Graph

A 2D grid is a graph where each cell is a vertex, and edges connect adjacent cells (typically up/down/left/right) — no explicit adjacency list needed; neighbors are computed by index arithmetic.
```
def dfs_grid(grid, r, c, visited):
    if (r < 0 or r >= len(grid) or c < 0 or c >= len(grid[0])
            or (r, c) in visited or grid[r][c] == BLOCKED):
        return
    visited.add((r, c))
    for dr, dc in [(-1,0),(1,0),(0,-1),(0,1)]:
        dfs_grid(grid, r + dr, c + dc, visited)
```
**Signals:** "grid", "islands", "flood fill", "connected regions" — extremely common in interviews specifically because a grid is a graph in disguise, and recognizing this immediately is a strong signal.

### 3. Topological Sort

Part B's algorithms, applied directly. **Signals:** "course schedule", "prerequisites/dependencies", "build order", "task ordering".

### 4. Cycle Detection

Part B's directed (3-state) vs. undirected (2-state + parent) distinction, applied directly. **Signals:** "detect a cycle", "can all courses be finished" (a course-schedule cycle is exactly an impossible-to-satisfy prerequisite chain).

### 5. Union-Find (Disjoint Set Union) — Foundational Version

An alternative to DFS/BFS specifically for **undirected** connectivity questions: maintain a "which group does this vertex belong to" structure, merging groups (`union`) as edges are processed, and answering "are these connected" (`find`) by comparing group representatives.
```
parent = list(range(n))
def find(x):
    while parent[x] != x:
        x = parent[x]
    return x
def union(a, b):
    ra, rb = find(a), find(b)
    if ra != rb:
        parent[ra] = rb
```
This basic version is O(n) worst case per operation without further optimization (Topic 12 covers the two optimizations — union by rank and path compression — that bring this down to near-O(1) amortized). **Signals:** "redundant connection", "number of connected components" as a streaming/incremental question (edges added one at a time, connectivity queried along the way).

### 6. Multi-Source BFS

Start BFS from **multiple sources simultaneously** (all pushed into the queue before the first pop) instead of running a separate BFS per source — this answers "distance/time to reach every point from the nearest of several starting points" in a single O(V+E) pass, instead of `O(sources x (V+E))`.
**Signals:** "rotting", "spreading", "infection", "nearest of several sources".

### 7. Bipartite Check (2-Coloring)

A graph is bipartite if its vertices can be split into two groups such that every edge connects vertices in *different* groups — checked via BFS/DFS, alternating colors as traversal proceeds; a conflict (an edge connecting two same-colored vertices) means the graph is not bipartite.
**Signals:** "can be divided into two groups", "bipartite".

---

# Part D — Optimization Principles

- **Recognizing a grid as a graph** is itself the main "optimization" in Pattern 2 — treating it as a general search problem rather than something bespoke unlocks every traversal tool in this document immediately.
- **Multi-source BFS's real win**: `O(V+E)` total instead of `O(sources x (V+E))` — worth stating this complexity contrast explicitly, since the *un*-optimized "BFS from each source separately" version is a common, plausible-looking, strictly worse first instinct.
- **Union-Find vs. DFS/BFS for connectivity**: DFS/BFS answers "are these connected" by re-traversing from scratch, `O(V+E)` per query; Union-Find answers it near-O(1) amortized per query once built (Topic 12) — the right choice specifically when connectivity is queried repeatedly as edges are incrementally added, rather than computed once on a static, fully-known graph.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "grid", "islands", "connected regions", "flood fill" | Grid as Graph (Pattern 2) |
| "shortest path" + unweighted | BFS (Part B's proof applies directly) |
| "course schedule", "prerequisites", "dependencies", "build order" | Topological Sort (Pattern 3) |
| "detect a cycle", "can all be completed" | Cycle Detection (Pattern 4) |
| "connected components", "provinces", "redundant connection" | Union-Find (Pattern 5) or DFS/BFS (Pattern 1) |
| "rotting", "spreading", "nearest of multiple sources" | Multi-Source BFS (Pattern 6) |
| "two groups", "bipartite" | Bipartite Check (Pattern 7) |

---

# Part F — Common Gotchas & Interview Traps

- **Forgetting the visited set** (Part B) — the single most consequential graph-specific bug; a tree-DFS habit that doesn't carry over safely.
- **Using 2-state cycle detection on a directed graph** (Part B) — silently gives wrong answers rather than crashing; the 3-state (unvisited/in-progress/done) distinction is not optional for directed graphs.
- **BFS visited-marking timing**: a node must be marked visited the moment it is **enqueued**, not when it is later dequeued — marking on dequeue allows the same node to be enqueued multiple times before its first processing, wasting work and, in some formulations, producing wrong distances.
- **Undirected adjacency list construction**: an edge `(u, v)` in an undirected graph must be added to **both** `u`'s and `v`'s neighbor lists — a very easy detail to drop when building the graph from a raw edge list.
- **Grid boundary checks**: always check array bounds *before* indexing into the grid during recursive/BFS exploration, not after — reversing this order crashes instead of correctly stopping the traversal.
- **Confusing "connected" with "reachable"** in directed graphs — `u` being able to reach `v` does not imply `v` can reach `u`; "connected components" as a term properly applies to undirected graphs, while directed graphs need the distinct concept of "strongly connected components" (CLRS §22.5, beyond this document's scope but worth knowing the terms aren't interchangeable).

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Grid Traversal:** Number of Islands → Max Area of Island → Flood Fill → Pacific Atlantic Water Flow (a trickier multi-source-from-the-edges variant)

**General Graph Traversal:** Clone Graph → Number of Connected Components in an Undirected Graph

**Topological Sort:** Course Schedule → Course Schedule II → Alien Dictionary (a harder application — infer ordering constraints from adjacent word comparisons, then topologically sort)

**Cycle Detection:** Course Schedule (revisit specifically through the cycle-detection lens — an unsatisfiable prerequisite chain IS a cycle)

**Union-Find:** Graph Valid Tree → Redundant Connection

**Multi-Source BFS:** Rotting Oranges

**Bipartite:** Is Graph Bipartite?

---

# Part H — References

**Primary text:** CLRS, **Chapter 22, "Elementary Graph Algorithms," in full**: §22.1 (representations), §22.2 (BFS, including the shortest-path proof reproduced in Part B), §22.3 (DFS, including the edge-classification framework underlying cycle detection), §22.4 (topological sort). **Chapter 21, "Data Structures for Disjoint Sets"** for Union-Find's formal treatment (the full optimized version is Topic 12's focus, but the foundational structure here comes directly from this chapter).

**Video references** (search directly by name/series):
- **WilliamFiset** — a specifically excellent, comprehensive graph theory playlist, covering BFS/DFS, topological sort, and Union-Find with real rigor.
- **Abdul Bari** — clear coverage of graph traversal and topological sort fundamentals.
- **MIT OpenCourseWare, 6.006** — the graph traversal lectures, with the same BFS shortest-path proof rigor as CLRS.
- **NeetCode** — the Graphs section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can explain why a visited set is required for graphs but not trees
- [ ] Can reproduce the inductive proof that BFS finds shortest paths in unweighted graphs
- [ ] Can explain precisely why directed-graph cycle detection needs three states while undirected needs only two, with an example showing why two states fail on a directed graph
- [ ] Can implement topological sort via both the DFS-postorder-reversal method and Kahn's algorithm
- [ ] Can recognize a grid problem as a graph problem immediately, without needing the word "graph" to appear in the prompt
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 12

Next: **Phase 3, Topic 12 — Advanced Graphs**.
