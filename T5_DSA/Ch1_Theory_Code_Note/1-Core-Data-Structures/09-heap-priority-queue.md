# Heap / Priority Queue — Complete Theory, Patterns & Interview Mastery

**Phase 2 — Linked Structures & Trees, Topic 9 (final topic of Phase 2)** | Practice problems: NeetCode's Heap section, after this document, not before.

---

# Part A — What a Heap Actually Is

A heap is a **complete binary tree** (every level fully filled, except possibly the last, which fills left to right with no gaps) satisfying the **heap property**: in a min-heap, every parent is ≤ both its children (so the root is always the minimum); in a max-heap, every parent is ≥ both its children (root is the maximum).

**The heap property is deliberately weaker than full sorting** — it only constrains parent-child relationships; two sibling subtrees can be in either order relative to each other. This is not a limitation, it is the *entire reason* a heap is faster to maintain than a sorted structure: enforcing less order costs less work.

## The array representation — no pointers needed

Because a heap is always a *complete* tree (no gaps), it can be stored directly in a flat array, with parent/child relationships computed by index arithmetic instead of pointers:
```
parent(i) = (i - 1) // 2
left_child(i) = 2*i + 1
right_child(i) = 2*i + 2
```
This works precisely *because* completeness guarantees no gaps in the level-order array — every index from `0` to `n-1` corresponds to a real node, with no holes to account for. This is why Python's `heapq` operates on a plain `list`.

## Priority Queue vs. Heap — a distinction worth being precise about

A **priority queue** is an *abstract data type*: insert an item with a priority, extract the highest (or lowest) priority item. A **heap** is the *standard concrete implementation* of a priority queue — the same relationship as "list" (abstract) to "array" or "linked list" (concrete implementations).

---

# Part B — Key Theoretical Results

## Sift-down (heapify) correctness

To restore the heap property at a node whose children's subtrees already satisfy it, but which may itself violate it: compare the node against its children, swap with the larger (max-heap) or smaller (min-heap) child if violated, and recurse into the affected subtree. **Correctness relies on the children's subtrees already being valid heaps** — this precondition is exactly why building a heap from scratch must process nodes bottom-up (Part B, next), not top-down.

## Building a heap is O(n), not O(n log n) — a genuinely famous result, proved properly

Naively, inserting `n` elements one at a time (each an O(log n) sift-up) gives O(n log n). But **building a heap from an already-existing array, by heapifying from the last non-leaf node backward to the root, is O(n)** — this is a classic, precise result (CLRS §6.3), not a hand-wave, and the proof is worth reproducing:

Most nodes in a heap are near the *bottom*, and a node at height `h` (distance to its farthest leaf) costs at most O(h) to sift down. At height `h`, there are at most `ceil(n / 2^(h+1))` nodes. Total work is bounded by:

```
Sum over h=0..log(n) of  [n / 2^(h+1)] * O(h)   =   O( n * Sum over h=0..infinity of h / 2^h )
```

The sum `h/2^h` (for `h` from 0 to infinity) is a standard convergent series equal to exactly **2**. So total work is `O(n * 2) = O(n)`. The key intuition, without the algebra: there are exponentially *more* nodes at small heights (cheap to fix) than at large heights (expensive to fix, but rare) — the two effects trade off to a linear, not linearithmic, total.

## Insert and extract, and why both are O(log n)

- **Insert**: append to the end of the array, then **sift up** (swap with parent while the heap property is violated) — bounded by the tree's height, O(log n), since a complete tree with `n` nodes has height `O(log n)` (Topic 7's height/complexity relationship, reapplied).
- **Extract-min/max**: swap the root with the last element, remove the last element (this is the old root, now returned), then **sift down** from the new root — also O(log n), same reasoning.

## Heapsort, as the historically motivating application

Build a heap in O(n) (Part B above), then extract the max `n` times, each O(log n): `O(n) + O(n log n) = O(n log n)` total, in-place, O(1) extra space. This is CLRS's Chapter 6 in full — heaps are introduced there specifically to build this sorting algorithm, and the priority-queue use case this document focuses on is presented as a secondary application.

---

# Part C — The Core Patterns

### 1. Basic Operations (Python's `heapq`)

Min-heap only (Phase 0's DSA toolkit — negate for a max-heap). `heapify()` is O(n) (Part B); `heappush`/`heappop` are O(log n).

### 2. Top-K (Maintain a Heap of Size K)

```
import heapq
heap = []
for x in stream:
    heapq.heappush(heap, x)
    if len(heap) > k:
        heapq.heappop(heap)          # discard the smallest, keeping only the k LARGEST seen so far
# heap now holds the k largest elements; heap[0] is the smallest OF those k
```
O(n log k) — meaningfully better than a full O(n log n) sort when `k` is much smaller than `n`. **Signals:** "kth largest/smallest", "top k", "k closest".

### 3. K-Way Merge

Merging `k` sorted lists directly extends Topic 2's two-sequence merge pattern: instead of two pointers, maintain a heap holding the current front element of each of the `k` lists; repeatedly pop the smallest, advance that list's pointer, push its next element. `O(n log k)`, where `n` is the total number of elements across all lists.
**Signals:** "merge k sorted lists/arrays".

### 4. Two-Heap Pattern (Running Median)

Maintain a **max-heap** for the smaller half of numbers seen so far and a **min-heap** for the larger half, kept balanced in size (differing by at most 1). The median is always at the top of one heap (odd total count) or the average of both tops (even count).
```
small = []   # max-heap (negated), holds the SMALLER half
large = []   # min-heap, holds the LARGER half
def add_num(x):
    heapq.heappush(small, -x)
    heapq.heappush(large, -heapq.heappop(small))    # ensure every 'small' element <= every 'large' element
    if len(large) > len(small):
        heapq.heappush(small, -heapq.heappop(large))
def find_median():
    if len(small) > len(large): return -small[0]
    return (-small[0] + large[0]) / 2
```
**Signals:** "median", especially "running"/"streaming" median.

### 5. Greedy Scheduling with a Heap

Many greedy algorithms (Phase 4) need "always process the next-highest-priority item," where priorities change dynamically as items are processed — exactly a priority queue's job. Task scheduling, meeting room allocation, and similar problems typically push all candidates into a heap and repeatedly pop the best next choice.
**Signals:** "scheduling", "meeting rooms", "task", "priority".

---

# Part D — Optimization Principles

- **The core niche a heap fills**: repeated insert AND extract-min/max, both needed, both dynamically, over time — an unsorted array gives O(1) insert but O(n) extract-min; a sorted array gives O(1) extract-min but O(n) insert; a heap gives O(log n) for **both**, the balanced middle ground that wins whenever both operations recur.
- **Top-K's O(n log k) vs. full-sort's O(n log n)**: worth stating the two complexities side by side explicitly — meaningfully different when `k` is small relative to `n` (e.g. `k=10` out of `n=10^6`).
- **Build-heap's O(n) vs. n individual inserts' O(n log n)** (Part B): if all elements are known upfront, `heapify()` on the full array is asymptotically better than pushing them one at a time — a genuinely underused optimization worth defaulting to.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "kth largest/smallest", "top k" | Top-K (Pattern 2) |
| "merge k sorted..." | K-Way Merge (Pattern 3) |
| "median", especially of a stream | Two-Heap (Pattern 4) |
| "scheduling", "meeting rooms", "priority", "task" | Greedy + Heap (Pattern 5) |
| "k closest points" | Top-K variant (Pattern 2), with a custom distance key |

---

# Part F — Common Gotchas & Interview Traps

- **Python `heapq` is min-heap only** — max-heap needs value negation (Phase 0's DSA toolkit); forgetting this silently returns the k *smallest* when the k *largest* were wanted.
- **Assuming full sort order from a heap** — only the parent-child relationship is guaranteed; `heap[1]` is **not** necessarily the second-smallest element (it's merely one of the root's two children, either of which could hold that title, or not).
- **Two-heap rebalancing off-by-one**: forgetting to rebalance heap sizes after every insertion silently breaks the median calculation the moment the two heaps' sizes diverge by more than one.
- **Reaching for a heap when a single running variable suffices** — if only the overall min/max across one static pass is needed (no repeated dynamic insert/extract), a heap is unnecessary machinery; this is a real over-engineering signal worth catching in oneself.
- **Heaps of tuples**: Python compares tuples element-by-element — if the first (priority) elements tie, it falls through to compare the second elements, which can crash or misbehave if those aren't directly comparable (e.g. two dicts). Include a tie-breaking key (like an insertion index) as the second tuple element specifically to avoid this.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Basic / Top-K:** Kth Largest Element in an Array → Kth Largest Element in a Stream → K Closest Points to Origin → Top K Frequent Elements (revisit Topic 1 — compare the heap-based O(n log k) approach against the bucket-sort O(n) approach solved there, and be able to state the tradeoff)

**K-Way Merge:** Merge k Sorted Lists (revisit Topic 6 — implement with a heap this time)

**Two-Heap:** Find Median from Data Stream

**Greedy + Heap:** Task Scheduler → Meeting Rooms II

---

# Part H — References

**Primary text:** CLRS — **Chapter 6, "Heapsort," in full**: §6.1 (heaps), §6.2 (maintaining the heap property — sift-down), §6.3 (building a heap — the exact O(n) proof reproduced in Part B), §6.4 (the heapsort algorithm), §6.5 (priority queues). This is one of the most directly, thoroughly covered topics in this entire syllabus relative to CLRS — worth reading the whole chapter, not excerpting it.

**Video references** (search directly by name/series):
- **Abdul Bari** — a clear, classic heapsort and heap-operations walkthrough matching CLRS's structure closely.
- **MIT OpenCourseWare, 6.006** — the heaps lecture, covering the same build-heap complexity proof with the same rigor.
- **mycodeschool** — visual heap operation walkthroughs (sift-up/sift-down).
- **NeetCode** — the Heap section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can explain the array representation's index math and why completeness is what makes it valid
- [ ] Can reproduce, at least in outline, the proof that building a heap is O(n), not O(n log n)
- [ ] Can state why insert and extract are both O(log n), tied to the tree's height
- [ ] Can explain when a heap is the right choice versus an unsorted array, a sorted array, or a single running variable
- [ ] Can implement the two-heap running-median pattern, including correct rebalancing
- [ ] Has independently solved at least one problem from each pattern category in Part G

---

# Phase 2 Complete

Linked List → Trees → Tries → Heap/Priority Queue are done. Before Phase 3, revisit each Self-Check section cold — Phase 3 (Backtracking, Graphs, Advanced Graphs) leans heavily on tree/graph DFS from Topic 7 and on the recursion theory from Phase 0.

Next: **Phase 3, Topic 10 — Backtracking**.
