# Advanced Graphs — Complete Theory, Patterns & Interview Mastery

**Phase 3 — Search Over Structures, Topic 12 (final topic of Phase 3)** | Practice problems: NeetCode's Advanced Graphs section, after this document, not before.

---

# Part A — What Makes These "Advanced": Weighted Edges

Topic 11's BFS shortest-path guarantee (Part B's proof) relies entirely on every edge costing exactly 1. The moment edges carry **different weights**, a path with more edges can legitimately be shorter than a path with fewer, and BFS's level-order guarantee no longer holds at all. This topic is, structurally, "what replaces BFS once edges are weighted" (Dijkstra, Bellman-Ford, Floyd-Warshall), plus the analogous question for building a minimum-cost connected structure instead of a shortest path (Minimum Spanning Trees).

---

# Part B — Shortest Paths With Weighted Edges

## Dijkstra's Algorithm: greedy + heap, non-negative weights only

Maintain a min-heap of `(distance_so_far, vertex)`, always expanding the currently-closest unvisited vertex next, updating neighbors' distances when a shorter path through the current vertex is found.

```
import heapq
def dijkstra(graph, source):
    dist = {source: 0}
    heap = [(0, source)]
    finalized = set()
    while heap:
        d, u = heapq.heappop(heap)
        if u in finalized:
            continue                      # stale heap entry (Python's heapq has no decrease-key)
        finalized.add(u)
        for v, weight in graph[u]:
            if d + weight < dist.get(v, float('inf')):
                dist[v] = d + weight
                heapq.heappush(heap, (dist[v], v))
    return dist
```

**Correctness, and precisely why it requires non-negative weights**: when a vertex is popped from the heap, it is popped with the smallest tentative distance among all remaining candidates. Because every other unvisited vertex has distance >= the popped vertex's distance (heap-ordering), and every edge weight is **non-negative**, any path reaching the popped vertex through a *different*, not-yet-finalized vertex could only be *longer*, never shorter. So the popped distance is provably final and correct at the moment of popping. **This entire argument collapses with a negative edge**: a large-but-finalized distance could later be beaten by a path through a vertex that looked worse at the time but leads to a large negative edge — exactly why Dijkstra silently produces wrong answers (not an error, a wrong number) on graphs with negative weights.

**Complexity**: `O((V+E) log V)` with a binary heap.

## Bellman-Ford: handles negative weights (not negative cycles)

Relax **every** edge, `V-1` times:
```
def bellman_ford(edges, V, source):
    dist = [float('inf')] * V
    dist[source] = 0
    for _ in range(V - 1):
        for u, v, w in edges:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
    for u, v, w in edges:                  # one more pass -- if anything still improves, a negative cycle exists
        if dist[u] + w < dist[v]:
            raise ValueError("negative cycle detected")
    return dist
```
**Correctness**: any shortest path (in a graph with no negative cycle) visits at most `V-1` edges — revisiting a vertex would mean the path contains a cycle, which (absent negative cycles) can only add non-negative cost, so an optimal path never does. After `V-1` full relaxation rounds, every possible shortest path (at most `V-1` edges long) has been fully accounted for. A `V`-th round that still finds an improvement is only possible if a **negative cycle** exists, which this doubles as a direct, standard detection method for. **Complexity**: `O(V*E)` — worse than Dijkstra, but strictly more general.

## Floyd-Warshall: all-pairs shortest paths, via dynamic programming

`dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j])`, iterating `k` over all vertices as an allowed intermediate stop, for every pair `(i, j)`. This is a direct preview of Phase 4's dynamic programming: the recurrence asks "is there a shorter path from `i` to `j` that goes through `k`," building up the answer for larger allowed-intermediate sets from smaller ones. **Complexity**: `O(V^3)` — appropriate specifically when all-pairs distances are needed and `V` is small enough that cubic time is acceptable; a poor choice for single-source queries on a large graph, where Dijkstra or Bellman-Ford are the right tool instead.

## DAG shortest (or longest) path: no heap needed at all

If the graph is known to be a **DAG** (Topic 11), process vertices in topological order, relaxing each vertex's outgoing edges exactly once as it's processed — correct because, by definition of topological order, every predecessor of a vertex has already been finalized by the time that vertex is reached. **Complexity**: `O(V+E)`, strictly better than Dijkstra's `O((V+E) log V)` — worth explicitly checking "is this a DAG?" before defaulting to Dijkstra, since the answer changes the achievable complexity class, not just the constant factor.

---

# Part C — Minimum Spanning Trees (MST)

A **spanning tree** connects all vertices of a (weighted, undirected, connected) graph using exactly `V-1` edges and no cycles; the **minimum** spanning tree does so at the lowest possible total edge weight.

## The Cut Property — the correctness backbone for both MST algorithms

**Claim**: for any partition ("cut") of the graph's vertices into two non-empty groups, the minimum-weight edge crossing that cut belongs to **some** MST. This is provable by an exchange argument: if an MST did not contain that minimum crossing edge, swapping it in for whatever edge the MST *does* use to cross that same cut cannot increase the total weight (since the swapped-in edge is, by assumption, the minimum crossing edge) — producing another valid spanning tree of equal or lower cost. Both algorithms below are, at their core, repeated direct applications of this single property.

## Kruskal's Algorithm: sort edges, Union-Find to avoid cycles

```
def kruskal(edges, V):                     # edges: list of (weight, u, v)
    edges.sort()
    uf = UnionFind(V)                       # with union by rank + path compression, see below
    mst_weight = 0
    for w, u, v in edges:
        if uf.find(u) != uf.find(v):        # adding this edge would NOT create a cycle
            uf.union(u, v)
            mst_weight += w
    return mst_weight
```
Process edges from lightest to heaviest, greedily adding any edge that connects two currently-separate components — by the Cut Property, this greedy choice is always safe. **Complexity**: `O(E log E)`, dominated by the sort.

## Prim's Algorithm: grow one tree greedily, heap-based

Structurally identical to Dijkstra, with one key difference: track the minimum edge weight to reach each vertex **from the growing tree**, not the cumulative path distance from a source. Repeatedly extract the cheapest edge connecting the current tree to a new vertex.
**Complexity**: `O((V+E) log V)` with a binary heap — the natural choice when the graph is dense (many edges relative to vertices), where Kruskal's sort becomes the bottleneck.

## Union-Find, fully optimized: union by rank + path compression

Topic 11 introduced the basic version. Two optimizations bring it to near-constant time:
- **Union by rank**: always attach the smaller (lower-rank) tree under the root of the larger one, keeping tree height logarithmic instead of letting it degenerate into a chain.
- **Path compression**: during `find`, make every node on the path point directly to the root, flattening future lookups.
```
def find(self, x):
    if self.parent[x] != x:
        self.parent[x] = self.find(self.parent[x])    # path compression
    return self.parent[x]

def union(self, a, b):
    ra, rb = self.find(a), self.find(b)
    if ra == rb: return
    if self.rank[ra] < self.rank[rb]: ra, rb = rb, ra
    self.parent[rb] = ra
    if self.rank[ra] == self.rank[rb]: self.rank[ra] += 1
```
**Combined, these give amortized `O(alpha(n))` per operation** — where `alpha` is the inverse Ackermann function, which grows so slowly that `alpha(n) < 5` for any `n` conceivably encountered in practice. This is, for all practical purposes, constant time, and is a specific, precise fact worth stating exactly this way rather than a vague "it's basically O(1)."

---

# Part D — Optimization Principles

- **Weighted shortest path algorithm selection is a decision tree worth having memorized**: non-negative weights, single source -> Dijkstra. Possible negative weights, single source -> Bellman-Ford (and it doubles as negative-cycle detection). All-pairs needed, graph small -> Floyd-Warshall. Known DAG -> topological-order relaxation, strictly faster than any of the above.
- **Kruskal's vs. Prim's**: both correct, both from the same Cut Property; Kruskal's `O(E log E)` favors sparser graphs (fewer edges to sort), Prim's `O((V+E) log V)` favors denser ones — worth being able to state this as an explicit tradeoff rather than defaulting to one by habit.
- **The concrete Union-Find payoff**: naive Union-Find (Topic 11) is `O(n)` worst case per operation; fully optimized (this document) is amortized `O(alpha(n)) ~= O(1)` — worth stating this contrast with the specific inverse-Ackermann fact, a genuine, precise, quotable result.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "shortest path" + weighted, non-negative | Dijkstra |
| "negative edge weights", "detect a negative cycle" | Bellman-Ford |
| "all pairs shortest path" | Floyd-Warshall |
| Known/stated DAG + shortest or longest path | Topological-order relaxation |
| "minimum spanning tree", "minimum cost to connect all" | Kruskal's or Prim's |
| "redundant connection", incremental connectivity queries | Union-Find (fully optimized) |

---

# Part F — Common Gotchas & Interview Traps

- **Using Dijkstra on a graph with negative weights** — the single most serious error in this topic: it does not crash, it returns a *wrong* number, confidently. Always confirm non-negative weights before defaulting to Dijkstra.
- **Running Bellman-Ford for only one relaxation pass** instead of the full `V-1` — an easy off-by-one that silently under-relaxes longer shortest paths.
- **Skipping the extra verification pass** in Bellman-Ford — without it, a negative cycle silently produces an incorrect "shortest path," rather than being detected and reported.
- **Union-Find without both optimizations** — union by rank alone, or path compression alone, still gives a real asymptotic improvement over the naive version, but only *both together* achieve the amortized `O(alpha(n))` bound; dropping either one is a common incomplete implementation.
- **Reaching for Floyd-Warshall's `O(V^3)`** when only single-source distances are actually needed — a correct but needlessly expensive choice when Dijkstra or Bellman-Ford would suffice.
- **Stale heap entries in a `heapq`-based Dijkstra**: Python's `heapq` has no `decrease-key` operation, so outdated `(distance, vertex)` entries can remain in the heap after a shorter distance was found — the `if u in finalized: continue` guard (Part B's code) is necessary, not defensive boilerplate.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Dijkstra:** Network Delay Time → Path with Minimum Effort → Swim in Rising Water (a variant solvable via Dijkstra-style greedy expansion, or via binary search + BFS — worth attempting both, per the "state multiple valid approaches" habit from earlier topics)

**Bellman-Ford / Modified Shortest Path:** Cheapest Flights Within K Stops (a Bellman-Ford variant bounded by a hop count, rather than run to full convergence)

**Union-Find (Fully Optimized):** Redundant Connection → Number of Connected Components in an Undirected Graph (revisit Topic 11 with the optimized implementation) → Graph Valid Tree (revisit)

**Minimum Spanning Tree:** Min Cost to Connect All Points (solve with both Kruskal's and Prim's, and be able to state the tradeoff between them)

**Topological Order Application:** Alien Dictionary (revisit Topic 11 — the ordering-inference step feeds directly into a topological sort), Course Schedule (revisit once more, now explicitly as a DAG-detection problem)

---

# Part H — References

**Primary text:** This is the most thoroughly, directly CLRS-covered topic in this entire syllabus. **Chapter 24, "Single-Source Shortest Paths"**: §24.1 (Bellman-Ford), §24.2 (DAG shortest paths), §24.3 (Dijkstra). **Chapter 25, "All-Pairs Shortest Paths"**: §25.2 (Floyd-Warshall). **Chapter 23, "Minimum Spanning Trees"**: §23.1 (the Cut Property, formally, as "growing an MST"), §23.2 (Kruskal's and Prim's). **Chapter 21, "Data Structures for Disjoint Sets"**: the full union-by-rank-plus-path-compression analysis, including the formal inverse-Ackermann-function amortized bound referenced in Part C.

**Video references** (search directly by name/series):
- **Abdul Bari** — clear, classic walkthroughs of Dijkstra, Bellman-Ford, Floyd-Warshall, and both MST algorithms, closely matching CLRS's structure.
- **WilliamFiset** — a particularly strong, implementation-focused treatment of the same algorithms, plus Union-Find's optimizations in detail.
- **MIT OpenCourseWare, 6.006** — the shortest-paths and MST lectures, with the same formal correctness arguments as CLRS.
- **NeetCode** — the Advanced Graphs section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can explain precisely why Dijkstra's correctness proof requires non-negative weights, and what specifically breaks with a negative edge
- [ ] Can explain why Bellman-Ford needs exactly `V-1` relaxation rounds, and how a `V`-th round detects negative cycles
- [ ] Can state the Cut Property and explain how both Kruskal's and Prim's rely on it
- [ ] Can implement Union-Find with both union by rank and path compression, and state the amortized complexity precisely (inverse Ackermann, not just "basically constant")
- [ ] Can choose the correct shortest-path algorithm for a new problem based on its specific constraints (weights, negative edges, single-source vs. all-pairs, known DAG or not) within about a minute
- [ ] Has independently solved at least one problem from each pattern category in Part G

---

# Phase 3 Complete

Backtracking → Graphs → Advanced Graphs are done — the "Search Over Structures" phase, the deepest and most theory-dense phase so far. Before Phase 4, revisit every Self-Check across Phases 0–3; Dynamic Programming builds directly on Backtracking's recursion (Topic 10) and Floyd-Warshall's DP recurrence (Part B of this document) — anyone who found Floyd-Warshall's recurrence natural has already done real DP thinking without the label.

Next: **Phase 4, Topic 13 — 1-D Dynamic Programming**.
