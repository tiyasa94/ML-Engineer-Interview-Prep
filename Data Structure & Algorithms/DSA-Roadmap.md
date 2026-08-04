# Data Structures & Algorithms Roadmap

> Primary Roadmap: **NeetCode Roadmap**
> [https://neetcode.io/roadmap]

> Goal: Master Data Structures & Algorithms for Machine Learning Engineer and Software Engineer interviews at top tech companies (NVIDIA, Google, Meta, Apple, Microsoft, Amazon, OpenAI, Anthropic).

---

# Learning Strategy

For each topic:

- Learn the underlying data structure or algorithm.
- Understand common problem-solving patterns.
- Implement from scratch in Python.
- Solve curated interview problems.
- Analyze time and space complexity.
- Revisit difficult problems after a few days.
- Practice under interview conditions.

---

# Problem-Solving Framework (per problem)

The topic-level strategy above governs a week; this governs a single problem, every time, no exceptions — including problems that look easy. Skipping steps is exactly what breaks down under real interview pressure.

1. **Read & restate** — read the problem twice. Restate it in your own words before writing anything. If you cannot restate it, you have not understood it.
2. **Clarify constraints** — input size, value ranges, duplicates allowed, sorted or not, negative numbers, empty input. Write assumptions down even when solo; this is a habit, not a formality.
3. **Work examples by hand** — 2-3 examples, including at least one edge case, before touching code.
4. **Brute force first** — always find *a* correct solution before the best one. State its time/space complexity out loud. This anchors the optimization that follows and is a real fallback if time runs out in an interview.
5. **Name the pattern** — which of the ~19 syllabus patterns below does this match? If none obviously fit, that itself is useful signal — flag the problem for the log in step 11.
6. **Optimize** — can a different data structure remove a loop? Can space be traded for time (hashing, memoization)? Is there redundant recomputation?
7. **Code cleanly** — meaningful names, small helper functions, no premature cleverness. Interview code is read by a human under time pressure, including future-you revisiting it.
8. **Test by tracing** — walk your own code through the examples from step 3 by hand, plus: empty input, single element, all-duplicates, already-sorted/reverse-sorted, negative numbers.
9. **State complexity, justified** — not just "O(n log n)" but *why*: which line dominates, what the recurrence is, what the space actually holds.
10. **Explain out loud** — say the approach as if to an interviewer, in under two minutes, before or instead of checking the editorial. If it cannot be said cleanly, it is not actually understood yet.
11. **Log and schedule a revisit** — one line: pattern used, what was missed, what would be done differently. Schedule a cold re-attempt in 3-5 days (see Revision System below).

---

# Syllabus: Topics, in Order

Based on the NeetCode roadmap, grouped into phases with a target timeline. Timeline assumes ~1.5-2 hrs/day, adjustable — the point is order and thoroughness, not the clock. Problem counts are minimums for first-pass coverage; revisits (Revision System, below) are additional.

## Phase 0 — Setup (Week 1)
- Big-O notation: time/space complexity analysis, amortized analysis, best/worst/average case.
- Python DSA toolkit: `list`, `dict`, `set`, `collections.deque`, `heapq`, `bisect`, `itertools` — already covered in `Python & DSA/02-data-structures-native.ipynb`; this week is a refresh + drilling the complexity table from memory, not re-learning from zero.
- Recursion refresher: base case, recursive case, call stack, converting recursion to iteration.

## Phase 1 — Foundational Patterns (Weeks 2-5)
| # | Topic | Core patterns | Target problems |
|---|---|---|---|
| 1 | Arrays & Hashing | frequency counting, prefix sums, hash map for O(1) lookup | 12-15 |
| 2 | Two Pointers | opposite-end pointers, fast/slow pointers | 8-10 |
| 3 | Sliding Window | fixed window, variable window, shrink/grow conditions | 8-10 |
| 4 | Stack | monotonic stack, matching/parsing, next-greater-element | 8-10 |
| 5 | Binary Search | classic, search-on-answer, rotated array, boundary search | 10-12 |

These five are the highest-frequency patterns across all companies on the list, including NVIDIA's SWE/infra-adjacent loops — mastery here before moving on is worth the time.

## Phase 2 — Linked Structures & Trees (Weeks 6-8)
| # | Topic | Core patterns | Target problems |
|---|---|---|---|
| 6 | Linked List | reversal, fast/slow (cycle detection), dummy node, merge | 10-12 |
| 7 | Trees | DFS (pre/in/post-order), BFS, recursion on trees, BST properties | 15-18 |
| 8 | Tries | prefix tree construction, word search, autocomplete-style problems | 4-5 |
| 9 | Heap / Priority Queue | k-largest/smallest, merge k sorted, two-heap median | 8-10 |

## Phase 3 — Search Over Structures (Weeks 9-11)
| # | Topic | Core patterns | Target problems |
|---|---|---|---|
| 10 | Backtracking | subsets, permutations, combinations, constraint satisfaction, pruning | 10-12 |
| 11 | Graphs | DFS/BFS on graphs, topological sort, union-find basics, grid-as-graph | 15-18 |
| 12 | Advanced Graphs | Dijkstra, Bellman-Ford, Floyd-Warshall, Prim's/Kruskal's MST | 6-8 |

## Phase 4 — Dynamic Programming, Intervals, Greedy (Weeks 12-15)
The historically hardest block — budget the most time here, not the least.

| # | Topic | Core patterns | Target problems |
|---|---|---|---|
| 13 | 1-D Dynamic Programming | top-down (memo) → bottom-up (tabulation), state definition, transitions | 15-18 |
| 14 | Intervals | sorting by start/end, merge, overlap detection | 6-8 |
| 15 | Greedy | exchange-argument intuition, when greedy provably works vs. does not | 6-8 |
| 16 | 2-D Dynamic Programming | grids, two-string DP (edit distance, LCS), state-space thinking | 10-12 |

## Phase 5 — Rounding Out (Week 16)
| # | Topic | Core patterns | Target problems |
|---|---|---|---|
| 17 | Advanced Trees | Union-Find with path compression/rank, Segment Tree, Binary Indexed Tree | 4-6 |
| 18 | Bit Manipulation | XOR tricks, bitmasking, counting bits | 5-6 |
| 19 | Math & Geometry | modular arithmetic, GCD/LCM, matrix rotation, line/point geometry | 5-6 |

## Phase 6 — Integration (Weeks 17-20, ongoing after)
Mock interviews, timed mixed-topic sets, weak-area revision, company-specific practice (below). This phase does not really end — it folds into ongoing maintenance once the initial 16 weeks are done.

**Total: ~180-220 problems for first-pass coverage**, roughly matching NeetCode 150/250 in scope, before revisits.

---

# Repository Structure

Same spirit as `Mathematical Foundations/` and `Python & DSA/`: one folder per topic, problems inside as individual notebooks, each following the same fixed template so the format itself becomes a habit.

```
DSA/
├── 00-big-o-and-python-toolkit/
├── 01-arrays-hashing/
├── 02-two-pointers/
├── 03-sliding-window/
├── 04-stack/
├── 05-binary-search/
├── 06-linked-list/
├── 07-trees/
├── 08-tries/
├── 09-heap-priority-queue/
├── 10-backtracking/
├── 11-graphs/
├── 12-advanced-graphs/
├── 13-dp-1d/
├── 14-intervals/
├── 15-greedy/
├── 16-dp-2d/
├── 17-advanced-trees/
├── 18-bit-manipulation/
├── 19-math-geometry/
├── mock-interviews/
└── revision-log.md
```

Each problem gets its own `.ipynb`, named `NN-problem-slug.ipynb`, with a fixed section order:

1. **Problem Statement** — restated in own words (Framework step 1).
2. **Constraints & Examples** — including the edge cases worked by hand (steps 2-3).
3. **Brute Force** — approach + complexity, even if never coded in full (step 4).
4. **Pattern Identified** — which syllabus pattern, and why (step 5).
5. **Optimized Approach** — plain-English explanation before any code (step 6).
6. **Pseudocode** — language-agnostic, a few lines.
7. **Python 3 Solution** — clean, tested, run against the examples plus edge cases in-notebook (steps 7-8).
8. **Complexity Analysis** — time and space, justified (step 9).
9. **Explanation Recording Note** — a written 2-minute-verbal-summary equivalent (step 10); optionally an actual recorded explanation, timed.
10. **Reflection** — what was missed, what to revisit, link to the `revision-log.md` entry (step 11).

---

# Complexity Cheat Sheet

Quick reference; full derivations already live in `Mathematical Foundations` and `Python & DSA/02-data-structures-native.ipynb`.

| Structure | Access | Search | Insert | Delete |
|---|---|---|---|---|
| Array / Python list | O(1) | O(n) | O(n) worst, O(1) amortized append | O(n) |
| Hash map / set | — | O(1) avg | O(1) avg | O(1) avg |
| Linked list | O(n) | O(n) | O(1) at known node | O(1) at known node |
| Binary search tree (balanced) | O(log n) | O(log n) | O(log n) | O(log n) |
| Heap | O(1) peek | O(n) | O(log n) | O(log n) |

| Algorithm family | Typical complexity |
|---|---|
| Binary search | O(log n) |
| Sorting (comparison-based) | O(n log n) |
| DFS/BFS on a graph | O(V + E) |
| Dijkstra (binary heap) | O((V+E) log V) |
| DP over 1 dimension of size n | O(n) to O(n²) depending on transition |
| DP over 2 strings of length m, n | O(mn) |

---

# Resources

- **NeetCode** — [neetcode.io/roadmap](https://neetcode.io/roadmap) (primary), NeetCode 150/250 problem lists, video explanations used *after* an honest independent attempt, never before.
- **LeetCode** — problem source and the actual interview-style timer/environment; used for the daily practice itself.
- **Books**: *Elements of Programming Interviews in Python* (Aziz, Lee, Prakash) — deep pattern coverage; *Cracking the Coding Interview* (McDowell) — broader interview-process coverage including behavioral.
- **This repo's own material**: `Python & DSA/01-fundamentals-syntax-control-flow.ipynb` and `02-data-structures-native.ipynb` for the language mechanics and native-structure complexity table that everything above assumes as background.

---

# Revision System (Spaced Repetition)

Log every problem in `revision-log.md` the day it is first solved:

| Date solved | Problem | Pattern | Difficulty | Result | Revisit 1 (+3d) | Revisit 2 (+10d) | Revisit 3 (+30d) |
|---|---|---|---|---|---|---|---|
| | | | | solved cold / needed hint / could not solve | | | |

- **Solved cold, no issues** → revisit at +10 days, then +30.
- **Needed a hint or made an error** → revisit at +3 days, then re-enter the same schedule from there.
- **Could not solve** → re-attempt within 24-48 hours after reviewing the approach, then treat as a fresh entry.

A problem is only considered *learned*, not just *solved*, after a cold, unaided, correctly-timed re-attempt.

---

# Mock Interview Practice Plan

Starts in Phase 3, once enough patterns exist to make a mixed set meaningful; becomes weekly by Phase 6.

- **Weeks 9-11**: one self-timed mock per week, single topic, 45 minutes, 1-2 problems — no notes, narrate out loud while solving (Framework step 10 under time pressure).
- **Weeks 12-16**: one mixed-topic mock per week, 45-60 minutes, unknown pattern in advance — closer to the real thing.
- **Weeks 17+**: two mocks per week if possible, at least one with another person (peer, mentor, or a platform with live interviewers) — solo mocks cannot replicate the pressure of explaining to someone actively judging the explanation.
- Every mock gets logged the same way as a regular problem, plus: communication quality, whether the pattern was identified within the first 2 minutes, and whether the explanation could stand on its own without the interviewer prompting for it.

---

# Company-Specific Notes

General patterns hold everywhere; these are the differences worth knowing going in.

- **NVIDIA** — SWE/MLE loops lean noticeably more toward strong CS fundamentals and systems-adjacent reasoning (memory, performance, sometimes C++-flavored questions even for Python-primary roles) alongside standard DSA; do not neglect Phase 5 (bit manipulation, math) or the complexity-analysis rigor in Framework step 9.
- **Google** — classic, pattern-heavy DSA loops; strong emphasis on clean code and on communicating the approach *before* coding (Framework steps 5-6 matter as much as the code itself).
- **Meta** — fast-paced, typically two DSA rounds; speed and correctness under time pressure matter — this is where the timed mocks (above) pay off most directly.
- **Amazon** — DSA plus a heavier behavioral component tied to their leadership principles woven into the same interview; solving the problem is necessary but not sufficient.
- **Apple** — loops vary more by team than most companies on this list; broad, well-rounded coverage across all 19 syllabus topics matters more than over-indexing on any one.
- **OpenAI / Anthropic** — DSA loops tend to be shorter/lighter relative to ML-systems- and research-engineering-depth rounds; treat this roadmap as necessary-but-not-sufficient and keep it running in parallel with the ML Engineer track elsewhere in this repo, not sequentially before it.

---

# Mastery Criteria (per topic)

A topic is not "done" when the target problem count is hit — it is done when, cold:

- [ ] The pattern can be identified from the problem statement alone, within ~2 minutes, without seeing the topic name in advance.
- [ ] A brute-force solution can be produced immediately, and the optimization from there can be explained before coding it.
- [ ] The optimized solution can be coded without major bugs on the first attempt, within a reasonable time budget (Easy: ~15 min, Medium: ~25 min, Hard: ~40 min).
- [ ] Time and space complexity can be stated and justified without hesitation.
- [ ] The approach can be explained out loud, start to finish, in under two minutes.
- [ ] At least 2-3 problem variations within the pattern have been solved, not just one representative problem.

---

# Status

- [ ] Phase 0 — Setup
- [ ] Phase 1 — Foundational Patterns
- [ ] Phase 2 — Linked Structures & Trees
- [ ] Phase 3 — Search Over Structures
- [ ] Phase 4 — DP, Intervals, Greedy
- [ ] Phase 5 — Rounding Out
- [ ] Phase 6 — Integration (ongoing)

Update this section as each phase closes; it is the fastest way for anyone else following this same path to see where things stand.
