# DSA Pattern Master List — Interview Roadmap

## How to read this

A **data structure** (array, linked list, tree, heap...) is a container. A **pattern** is a
reusable *technique* for manipulating one or more data structures to solve a whole family of
problems in one shot. The reason pattern-first prep works is that ~90% of asked questions are a
thin skin over one of the ~30 patterns below — recognize the pattern, and you've solved the
problem before you've written a line of code.

Patterns are listed in **recommended learning order** — each phase leans on skills from the
phase before it (you can't do Topological Sort comfortably without BFS/DFS; you can't do most DP
without the recursion foundations already in your `0-Foundations` folder). Your foundational
theory (Big-O, Python toolkit, recursion) and core data structures (linked list, trees, tries,
heap) are prerequisites for everything here, not patterns themselves.

Each entry: **Signal** (how you recognize it in a problem statement) · **Core idea** (the actual
technique) · **Examples** (canonical problems to tag it, not solved here).

---

## Phase 1 — Array & String Patterns
*(≈ your repo: `2-Core-Problem-Solving-Patterns`)*

### 1. Hashing & Frequency Counting
**Signal:** need O(1) "have I seen this," "what's paired with this," "how many times," or
"group these by a derived key."
**Core idea:** trade space for time — dict/set turns an O(n) or O(n²) scan into O(n) via
constant-average-time membership/count lookups.
**Examples:** Two Sum, Contains Duplicate, Group Anagrams, Top K Frequent Elements, Longest
Consecutive Sequence.

### 2. Two Pointers
**Signal:** sorted array (or sortable), looking for a pair/triplet meeting a condition, or
comparing from both ends inward.
**Core idea:** two indices moving toward each other (or one fast/one slow) to exploit
sortedness/monotonicity — collapses O(n²) to O(n).
**Examples:** Two Sum II, 3Sum, Container With Most Water, Valid Palindrome.

### 3. Sliding Window
**Signal:** "contiguous subarray/substring" plus a constraint (max/min length, sum, distinct
count) to optimize over.
**Core idea:** maintain a window `[left, right]` that expands/contracts, updating state
incrementally instead of recomputing from scratch each time — O(n) instead of O(n²)/O(n³).
**Examples:** Longest Substring Without Repeating Characters, Minimum Window Substring, Max
Sliding Window (needs a monotonic deque), Longest Repeating Character Replacement.

### 4. Prefix Sum
**Signal:** repeated range-sum queries, or "subarray sum equals K"-style problems.
**Core idea:** precompute cumulative sums so any range sum is O(1); paired with a hashmap of
`prefix_sum → count/index`, solves subarray-sum problems in O(n).
**Examples:** Subarray Sum Equals K, Range Sum Query – Immutable, Product of Array Except Self.

### 5. Monotonic Stack
**Signal:** "next greater/smaller element," span problems, histogram-style area problems.
**Core idea:** keep a stack that's always increasing or decreasing; pop violators to find the
nearest greater/smaller element — O(n) total since each element is pushed/popped once.
**Examples:** Daily Temperatures, Largest Rectangle in Histogram, Next Greater Element,
Trapping Rain Water.

### 6. Binary Search (+ Binary Search on Answer)
**Signal:** sorted data (classic case), *or* the phrase "minimize the maximum / maximize the
minimum" over a numeric answer range even when the input array isn't sorted.
**Core idea:** repeatedly halve the search space using a monotonic true/false predicate — O(log n).
**Examples:** Search in Rotated Sorted Array, Koko Eating Bananas, Split Array Largest Sum.

### 7. Cyclic Sort
**Signal:** array holds numbers in a known range like `[1, n]` or `[0, n-1]`; asked for
missing/duplicate values in O(n) time, O(1) space.
**Core idea:** place each number at its "correct" index via swaps; one pass over the result
reveals mismatches.
**Examples:** Missing Number, Find All Duplicates in an Array, First Missing Positive.

---

## Phase 2 — Linked List Patterns
*(≈ your repo: `06-linked-list.md`)*

### 8. Fast & Slow Pointers
**Signal:** linked list (or any function-iteration sequence) + "does it cycle / find the cycle
start / find the middle."
**Core idea:** two pointers at different speeds must eventually meet if a cycle exists
(pigeonhole); the meeting point plus a distance argument locates the cycle start.
**Examples:** Linked List Cycle I & II, Middle of the Linked List, Happy Number.

### 9. In-place Reversal
**Signal:** "reverse a linked list," or "reverse in groups of k."
**Core idea:** rewire `next` pointers using three tracking pointers (prev, curr, next) — O(n)
time, O(1) space, no new nodes.
**Examples:** Reverse Linked List, Reverse Linked List II, Reverse Nodes in k-Group.

---

## Phase 3 — Tree & Graph Traversal Patterns
*(≈ your repo: `07-trees.md`, `10-backtracking.md`, `11-graphs.md`)*

### 10. Tree DFS
**Signal:** path from root to leaf, subtree properties, anything needing depth before breadth.
**Core idea:** recursive (or explicit-stack) pre/in/post-order traversal; pass state down via
arguments or accumulate up via return values.
**Examples:** Maximum Depth of Binary Tree, Path Sum, Validate BST, Lowest Common Ancestor.

### 11. Tree BFS
**Signal:** "level order," or shortest-path-in-hops phrased per level/distance.
**Core idea:** queue-based traversal processing one full level at a time.
**Examples:** Binary Tree Level Order Traversal, Binary Tree Right Side View, Minimum Depth of
Binary Tree.

### 12. Graph DFS/BFS
**Signal:** grid or explicit graph — "connected components," "number of islands," "shortest
path, unweighted."
**Core idea:** same traversal primitives as trees, plus a visited-set (graphs can cycle, trees
can't).
**Examples:** Number of Islands, Clone Graph, Word Ladder, Course Schedule (cycle detection).

### 13. Backtracking
**Signal:** "all possible," "generate all," subsets/permutations/combinations,
constraint-satisfaction (N-Queens, Sudoku).
**Core idea:** DFS through a decision tree — choose, recurse, un-choose — pruned by constraints
to skip dead branches early.
**Examples:** Subsets, Permutations, Combination Sum, N-Queens, Word Search.

### 14. Topological Sort
**Signal:** "task scheduling with prerequisites," "build order," ordering nodes in a DAG.
**Core idea:** Kahn's algorithm (BFS via in-degree counts) or DFS-with-finish-time — valid only
on directed acyclic graphs.
**Examples:** Course Schedule II, Alien Dictionary.

### 15. Union-Find (Disjoint Set)
**Signal:** "are these connected," incrementally merging groups, cycle detection in an
undirected graph while processing edges one at a time.
**Core idea:** forest of parent pointers; `find` (path compression) + `union` (by rank/size)
gives near-O(1) amortized operations.
**Examples:** Number of Provinces, Redundant Connection, Accounts Merge.

---

## Phase 4 — Heap-Based Patterns
*(≈ your repo: `09-heap-priority-queue.md`)*

### 16. Top-K Elements
**Signal:** "k largest/smallest/most frequent" — you don't need full sorted order, just the top k.
**Core idea:** maintain a heap of size k — O(n log k) instead of O(n log n) full sort.
**Examples:** Kth Largest Element in an Array, Top K Frequent Elements, K Closest Points to Origin.

### 17. K-way Merge
**Signal:** multiple already-sorted lists/arrays that need merging into one.
**Core idea:** min-heap holding the current head of each list; pop the smallest, push its
successor — O(n log k).
**Examples:** Merge k Sorted Lists, Kth Smallest Element in a Sorted Matrix.

### 18. Two Heaps
**Signal:** running median, or balancing a "lower half" against an "upper half" of streaming data.
**Core idea:** max-heap for the lower half + min-heap for the upper half, kept size-balanced —
O(1) median read, O(log n) insert.
**Examples:** Find Median from Data Stream, Sliding Window Median.

---

## Phase 5 — Interval & Trie Patterns
*(≈ your repo: `14-intervals.md`, `08-tries.md`)*

### 19. Merge Intervals
**Signal:** array of `[start, end]` ranges — merge overlaps, insert a new interval, find conflicts.
**Core idea:** sort by start time, linear-scan merging any interval overlapping the running
merged one — O(n log n), dominated by the sort.
**Examples:** Merge Intervals, Insert Interval, Meeting Rooms II, Non-overlapping Intervals.

### 20. Trie (Prefix Tree)
**Signal:** repeated prefix-based lookups — autocomplete, dictionary word search, longest
common prefix at scale.
**Core idea:** tree where each root-to-node path spells a prefix; insert/search is O(L) in word
length, independent of dictionary size.
**Examples:** Implement Trie, Word Search II, Design Add and Search Words Data Structure.

---

## Phase 6 — Dynamic Programming Family
*(≈ your repo: `13-dp-1d.md`, `16-dp-2d.md`)*

### 21. 1D DP
**Signal:** "number of ways to reach n," decisions along a single sequence where state at `i`
depends on a small window of previous states.
**Core idea:** `dp[i]` = best/count using the first i elements; find the recurrence relating
`dp[i]` to `dp[i-1], dp[i-2]...` — often reducible to O(1) space.
**Examples:** Climbing Stairs, House Robber, Decode Ways.

### 22. 0/1 Knapsack (subset-sum family)
**Signal:** choose a subset of items under a capacity/target constraint; each item usable once.
**Core idea:** `dp[i][cap]` = best value using the first i items within capacity; each item is
included or not — 2D table, collapsible to 1D iterated backwards.
**Examples:** Partition Equal Subset Sum, Target Sum, Ones and Zeroes.

### 23. Unbounded Knapsack (coin-change family)
**Signal:** like 0/1 knapsack, but each item can be reused unlimited times.
**Core idea:** same DP shape, but the inner loop runs forwards, allowing re-selection of the
same item within one pass.
**Examples:** Coin Change, Coin Change II, Combination Sum IV.

### 24. Grid / 2D DP
**Signal:** `m x n` grid, moving right/down (or similar constrained moves), asked for
paths/min-cost/max-value.
**Core idea:** `dp[i][j]` built from `dp[i-1][j]` and `dp[i][j-1]` — direct 2D generalization
of 1D DP.
**Examples:** Unique Paths, Minimum Path Sum, Edit Distance.

### 25. LCS / LIS Family (subsequence DP)
**Signal:** "longest common subsequence," "longest increasing subsequence," comparing two
sequences or finding the longest valid chain in one.
**Core idea:** LCS is 2D DP over both strings' prefixes; LIS is naively O(n²) 1D DP but has an
O(n log n) patience-sorting/binary-search optimization worth knowing explicitly.
**Examples:** Longest Common Subsequence, Longest Increasing Subsequence, Longest Palindromic
Subsequence.

### 26. Interval DP
**Signal:** answer for a range `[i, j]` depends on splitting it into two smaller sub-ranges at
some pivot `k`.
**Core idea:** `dp[i][j]` = best answer over the range i..j; iterate by increasing interval
length, trying every split point — typically O(n³).
**Examples:** Burst Balloons, Matrix Chain Multiplication, Palindrome Partitioning II.

### 27. Bitmask DP *(advanced)*
**Signal:** small n (usually ≤ 20), need to track "which subset of items has been
used/visited" as state.
**Core idea:** represent a subset as an integer bitmask; `dp[mask][i]` = best answer having
used exactly the items in `mask`, currently at i — O(2ⁿ · n) state space, viable only for small n.
**Examples:** Traveling Salesman (small n), Partition to K Equal Sum Subsets.

---

## Phase 7 — Greedy
*(≈ your repo: `15-greedy.md`)*

### 28. Greedy
**Signal:** "maximum/minimum number of X," and a locally-optimal choice at each step
provably never hurts the global optimum.
**Core idea:** sort by some criterion, make the obviously-best choice at each step, never
backtrack — correctness requires *proving* the greedy-choice property and optimal substructure,
which is the actual hard part in an interview, not the implementation.
**Examples:** Jump Game, Gas Station, Task Scheduler.

---

## Phase 8 — Advanced Graph Algorithms
*(≈ your repo: `12-advanced-graphs.md`)*

### 29. Dijkstra's Algorithm
**Signal:** weighted graph, all edge weights non-negative, shortest path from a single source.
**Core idea:** min-heap-driven BFS variant — always expand the currently-closest unvisited
node; O((V+E) log V) with a binary heap.
**Examples:** Network Delay Time, Path With Minimum Effort.

### 30. Bellman-Ford / Floyd-Warshall
**Signal:** negative edge weights (Bellman-Ford, also detects negative cycles), or all-pairs
shortest paths on a small graph (Floyd-Warshall).
**Core idea:** Bellman-Ford relaxes every edge V−1 times, O(VE); Floyd-Warshall is a
triple-nested-loop DP over "allowed intermediate nodes," O(V³).
**Examples:** Cheapest Flights Within K Stops, Find the City With the Smallest Number of
Neighbors at a Threshold Distance.

### 31. Minimum Spanning Tree (Prim's / Kruskal's)
**Signal:** "connect all nodes with minimum total edge weight."
**Core idea:** Kruskal's sorts edges, adds the cheapest one that doesn't create a cycle (via
Union-Find); Prim's grows one tree greedily from a start node using a min-heap of frontier edges.
**Examples:** Min Cost to Connect All Points, Connecting Cities With Minimum Cost.

---

## Phase 9 — Bit Manipulation & Math
*(≈ your repo: `18-bits-math.md`, `19-math-geometry.md`)*

### 32. Bit Manipulation
**Signal:** "without extra space" / constant-space constraints, or the problem is phrased in
terms of XOR/AND/OR directly.
**Core idea:** exploit properties like `a ^ a = 0` and `a ^ 0 = a` (great for "find the
single/missing number"), bit shifting for power-of-two checks, masking for subset enumeration.
**Examples:** Single Number, Counting Bits, Sum of Two Integers.

### 33. Math & Number Theory
**Signal:** the problem is really about GCD/LCM, primality, modular arithmetic, or
combinatorics dressed up as code.
**Core idea:** recognize the underlying math object — "this is really a sieve," "this is really
modular exponentiation" — rather than brute-forcing it.
**Examples:** Count Primes (Sieve of Eratosthenes), Pow(x, n) (fast exponentiation).

---

## Phase 10 — Specialized Range-Query Structures *(optional, hard-tier)*
*(≈ your repo: `17-advanced-trees.md`)*

### 34. Segment Tree / Fenwick Tree / Balanced BST
**Signal:** many range queries (sum/min/max) interleaved with point or range *updates* —
plain prefix sums break once updates are allowed.
**Core idea:** a tree where each node summarizes a range, giving O(log n) query and update
instead of O(n) per update.
**Examples:** Range Sum Query – Mutable, Count of Smaller Numbers After Self.

---

## Notes
- Your repo's `2-Core-Problem-Solving-Patterns` folder already matches Phase 1 items 1, 2, 3, 5,
  6 one-to-one. Prefix Sum (#4) and Cyclic Sort (#7) don't have dedicated files in your
  structure — they're likely worth folding into `01-arrays-hashing.md` as sub-sections, or
  giving their own short `.md` files if you want strict 1:1 mapping.
- This list is deliberately comprehensive (34 patterns) to match FAANG/Nvidia-tier breadth —
  not every company tests every phase equally. Phases 1–6 cover the large majority of *medium*
  questions across all of them; Phases 8 and 10 skew toward harder/graph-heavy interview loops
  (more common at Google, Amazon SDE-2+, Nvidia infra-adjacent roles).
- We'll go phase by phase, pattern by pattern, starting at #1, once you confirm this list is
  right. Nothing gets a deep-dive until you say go.
