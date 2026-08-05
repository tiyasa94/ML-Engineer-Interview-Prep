# Intervals — Complete Theory, Patterns & Interview Mastery

**Phase 4 — DP, Intervals, Greedy, Topic 14** | Practice problems: NeetCode's Intervals section, after this document, not before.

---

# Part A — What Interval Problems Are

Problems centered on ranges `[start, end]` — scheduling, merging, overlap detection, resource allocation. **The single most important realization in this entire topic**: sorting first (by start time, or by end time, depending on the problem) turns an otherwise complex pairwise-comparison problem into a simple linear scan. Choosing the correct sort key is the actual design decision this topic tests.

---

# Part B — Why Sorting First Works: Two Different Correctness Arguments

## Sorting by START enables merging — the monotonic-elimination argument

Once sorted by start time, overlapping intervals are guaranteed to be **adjacent** in the sorted order. Scanning left to right while maintaining a "current merged interval": a new interval either overlaps the current merge (its start ≤ the current merge's end) or it doesn't. The moment one doesn't overlap, **no future interval can retroactively overlap the earlier merge either** — every subsequent interval has an even later start (sorted order), so it can only be further away, never closer. This is the same "once eliminated, permanently eliminated" reasoning as Topic 2's two-pointer proof and Topic 3/4's monotonic structures, applied here to ranges instead of single values.

## Sorting by END enables greedy scheduling — the exchange argument (CLRS's own example)

For "select the maximum number of non-overlapping intervals" (the **activity-selection problem**), sort by **end** time and greedily take any interval that starts after the previously-taken one ends. **Correctness (CLRS §16.1, the textbook's own canonical greedy-algorithm example)**: consider any optimal solution. If it doesn't include the interval with the earliest finish time, that interval can always be swapped in for whatever the optimal solution *does* start with, without reducing the count and without breaking non-overlap — because finishing earliest leaves the maximum possible room for everything that follows. This exchange establishes that greedily taking the earliest-finishing interval first is always at least as good as any alternative — a direct preview of Topic 15's general exchange-argument technique for proving greedy algorithms correct.

---

# Part C — The Core Patterns

### 1. Merge Overlapping Intervals

```
intervals.sort(key=lambda iv: iv[0])          # sort by START
merged = [intervals[0]]
for start, end in intervals[1:]:
    if start <= merged[-1][1]:                  # overlaps the current merge
        merged[-1][1] = max(merged[-1][1], end)
    else:
        merged.append([start, end])
```
**Signals:** "merge overlapping intervals".

### 2. Insert Interval Into a Sorted, Non-Overlapping List

A three-part scan: intervals entirely before the new one (copy as-is), intervals overlapping the new one (merge into it), intervals entirely after (copy as-is) — a direct, structured application of Pattern 1's merge logic to a single insertion.
**Signals:** "insert interval", given a list already sorted and non-overlapping.

### 3. Interval Scheduling / Activity Selection (Sort by END)

```
intervals.sort(key=lambda iv: iv[1])          # sort by END
count, last_end = 0, float('-inf')
for start, end in intervals:
    if start >= last_end:
        count += 1
        last_end = end
```
Part B's exchange-argument-proven greedy, directly implemented. **Signals:** "maximum number of non-overlapping intervals", "minimum number of intervals to remove to make the rest non-overlapping" (the removal count is simply `total - count` from the algorithm above).

### 4. Minimum Rooms / Resources Needed (Sweep Line or Heap)

**Sweep-line version**: separate all start times and all end times into two sorted lists; scan through, incrementing a counter on each start and decrementing on each end (processed in time order, ends before starts at equal times per the problem's exact tie-breaking rule); the running maximum of that counter is the answer. **Heap version**: process intervals sorted by start; maintain a min-heap of currently-ongoing meetings' end times; if the earliest-ending ongoing meeting has already ended before the new one starts, reuse that room (pop it); otherwise a new room is needed (push). Both are valid, with a real tradeoff worth being able to state.
**Signals:** "minimum meeting rooms", "minimum resources/platforms needed simultaneously".

### 5. Interval Intersection (Two Sorted Lists)

A direct instance of Topic 2's merge-style two-pointer pattern: one pointer per list, advancing whichever interval ends first after recording any overlap between the current pair.
**Signals:** "intersection of two interval lists".

---

# Part D — Optimization Principles

- **The headline win**: brute-force pairwise overlap checking is `O(n²)`; sorting first brings the whole problem to `O(n log n)` — the dominant complexity cost becomes the sort itself, with the subsequent scan being linear.
- **The actual design decision is the sort key** — start time for merging (Pattern 1), end time for maximizing a non-overlapping count (Pattern 3) — get this backward and the algorithm is simply wrong, not just suboptimal.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "merge overlapping" | Merge (Pattern 1), sort by start |
| "insert interval" | Insert (Pattern 2) |
| "maximum non-overlapping", "minimum removals" | Activity Selection (Pattern 3), sort by end |
| "minimum rooms/resources needed simultaneously" | Sweep Line or Heap (Pattern 4) |
| "intersection of two interval lists" | Merge-Style Two Pointers (Pattern 5) |

---

# Part F — Common Gotchas & Interview Traps

- **Forgetting to sort at all** — the most common oversight; the instinct to jump straight to pairwise comparison is strong, and resisting it in favor of "sort first" is the core skill this topic tests.
- **Wrong sort key** — sorting by start when the problem actually needs end-time-based greedy selection (or vice versa) is a common, subtle wrong-answer source, not a crash.
- **Boundary/touching-interval ambiguity**: does `[1,2]` and `[2,3]` count as overlapping? This depends on the specific problem's definition (inclusive vs. exclusive endpoints) and must be checked explicitly rather than assumed — a frequent source of an off-by-one wrong answer.
- **Meeting Rooms' sweep-line implementation detail**: starts and ends must be sorted and swept as **separate** event lists, not as sorted whole intervals — conflating these is a common implementation slip.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Merge:** Merge Intervals → Insert Interval

**Activity Selection:** Non-overlapping Intervals

**Rooms/Resources:** Meeting Rooms → Meeting Rooms II (implement both the sweep-line and heap versions, and state the tradeoff)

**Intersection:** Interval List Intersections

---

# Part H — References

**Primary text:** CLRS, **Chapter 16, "Greedy Algorithms," §16.1, "An Activity-Selection Problem"** — the direct, dedicated, canonical treatment of Pattern 3, including the full exchange-argument correctness proof reproduced in Part B. Merging itself is not a named CLRS section but is a natural, simple application of sorting (any of CLRS's sorting chapters).

**Video references** (search directly by name/series):
- **Abdul Bari** — clear coverage of the activity-selection problem matching CLRS's own structure.
- **NeetCode** — the Intervals section of the roadmap; use after an independent attempt.
- **Back To Back SWE** — a solid walkthrough of Meeting Rooms II's two solution approaches (sweep line vs. heap).

---

# Self-Check / Mastery Criteria

- [ ] Defaults to sorting first, automatically, for any interval problem
- [ ] Can explain why sorting by start enables safe merging, using the monotonic-elimination argument
- [ ] Can reproduce the exchange-argument proof for why earliest-finish-first greedy scheduling is optimal
- [ ] Can implement Meeting Rooms II both via sweep line and via a heap, and state the tradeoff between them
- [ ] Checks boundary/touching-interval semantics explicitly rather than assuming a convention
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 15

Next: **Phase 4, Topic 15 — Greedy**.
