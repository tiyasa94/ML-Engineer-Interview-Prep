# Big-O Complexity Analysis — Theory, Patterns & Interview Tricks

**Phase 0 — Setup** | Companion notebook: [`01-big-o-complexity-analysis.ipynb`](./01-big-o-complexity-analysis.ipynb) (measured benchmarks proving several of the claims below)

---

## What Big-O actually measures

Growth rate as input size `n` gets large, ignoring constants and lower-order terms. `O(2n)` and `O(n)` are the same class; `O(n)` and `O(n²)` never converge no matter the constants, once `n` is large enough. Time complexity counts operations; space complexity counts **auxiliary** memory (not the input itself) — including the recursion call stack, which is the single most commonly forgotten term.

| Class | Name | Typical source |
|---|---|---|
| O(1) | constant | array index, hash lookup |
| O(log n) | logarithmic | binary search, balanced BST ops, halving |
| O(n) | linear | single pass |
| O(n log n) | linearithmic | comparison sorting, divide-and-conquer with linear merge |
| O(n²) | quadratic | nested loop over the same input |
| O(2ⁿ) | exponential | subsets, naive recursion without memoization |
| O(n!) | factorial | permutations, brute-force TSP |

---

## Deriving complexity from code — the rules

- **Sequential statements** → add: `O(n) + O(n) = O(n)`.
- **Nested loops** → multiply: `O(n) × O(n) = O(n²)`, regardless of whether the inner bound depends on the outer variable (a triangular loop is still O(n²), just a smaller constant).
- **A loop that halves the remaining problem each iteration** → O(log n).
- **A loop calling an O(n) operation each iteration** → multiply. This is the single most common *accidental* O(n²): `x in some_list` (O(n)) called inside a loop over `n` items. Swapping the list for a `set` fixes it — measured 175x speedup in the companion notebook from that one change alone.

## Recurrence relations — for recursive complexity specifically

A recursive function's complexity is a recurrence: `T(n) = a·T(n/b) + O(n^d)`, where `a` = number of recursive calls, `n/b` = size of each subproblem, `O(n^d)` = work done outside the recursive calls (e.g. merging). The Master Theorem intuition, without needing to prove it:

- Compare `d` to `log_b(a)`.
- If `d > log_b(a)` → the non-recursive work dominates: `O(n^d)`.
- If `d < log_b(a)` → the branching dominates: `O(n^(log_b a))`.
- If `d = log_b(a)` → `O(n^d · log n)`.

Merge sort: `T(n) = 2T(n/2) + O(n)` → `a=2, b=2, d=1`, `log_2(2)=1=d` → `O(n log n)`. Binary search: `T(n) = T(n/2) + O(1)` → `a=1, b=2, d=0`, `log_2(1)=0=d` → `O(log n)`.

## Amortized analysis

`list.append()` is O(1) **amortized**, not O(1) every single call — Python over-allocates and doubles capacity when full, so most appends are cheap writes into existing space, and occasionally one triggers an O(n) copy. Spread across many calls, the average stays constant (measured flat across a 100x range of `n` in the companion notebook). Contrast: `list.insert(0, x)` is genuinely O(n) **every** call — nothing amortizes it, because every existing element shifts. This is exactly why `collections.deque` exists for front operations.

## Best / worst / average case

The same algorithm can differ by input. Quicksort: O(n log n) average, O(n²) worst case on already-sorted input with a naive first-element pivot (measured ~40x slower in the companion notebook, same size input, same algorithm). In interviews, "what's the complexity" defaults to **worst case** unless stated otherwise — but naming average/best case too, and *why* they differ, is what separates memorized from understood.

---

## Interview Trick: Read the Constraints, Infer the Expected Complexity

Before designing an approach, the input bound in the problem statement tells you almost exactly what complexity is expected — this is one of the highest-leverage, least-taught interview habits:

| `n` up to... | Expected complexity | What that rules in/out |
|---|---|---|
| ~10–20 | O(2ⁿ) or O(n!) | brute force / backtracking over all subsets or permutations is fine |
| ~100–500 | O(n³) | triple nested loops acceptable |
| ~1,000–10,000 | O(n²) | double nested loops acceptable |
| ~100,000–1,000,000 | O(n log n) | sorting-based or heap-based approaches expected |
| ~10⁷–10⁸ | O(n) | single pass, hashing — no sort, no nested loop |
| beyond 10⁸, or huge value ranges | O(log n) or O(1) | binary search, math/formula-based |

Seeing `1 <= n <= 10^5` and reaching for an O(n²) solution is a signal to stop and reconsider *before* writing any code — and stating this reasoning out loud ("given n up to 10^5, I'm ruling out anything quadratic") reads, to an interviewer, as real engineering judgment rather than pattern-matched memorization.

## Interview Trick: Space-Time Tradeoffs Are the Real Optimization Story

The overwhelming majority of "can you do better than brute force" follow-ups resolve to one of:
- **Trade space for time**: a hash map/set turns an O(n) inner scan into O(1) (Arrays & Hashing, Phase 1).
- **Trade preprocessing for query time**: sorting once (O(n log n)) enables binary search per query (O(log n)) instead of a linear scan per query.
- **Trade recomputation for caching**: memoizing a recursive function turns exponential blowup into linear (measured 657x speedup, naive vs. memoized Fibonacci, in the Recursion notebook).

Naming *which* of these three the optimization is doing is a strong, concrete way to explain an approach.

## Common Gotchas / Rapid-Fire

- Two sequential O(n) loops is O(n), not O(2n) — do not overstate it as "twice as slow," it is the same class.
- String concatenation in a loop (`s += x`) is O(n²) *in principle*; CPython sometimes optimizes the common case away, but relying on that is fragile — `''.join(...)` is O(n) unconditionally (see Python & DSA notebook 01).
- Recursion depth **is** space complexity — a purely recursive O(n) algorithm is also O(n) *space*, from stack frames alone, even with zero explicit data structures.
- "Is this O(1) space?" — check whether an input is being copied/sliced anywhere; `nums[1:]` inside a recursive call is a hidden O(n) copy per call, easy to miss.

---

## Self-Check

- [ ] Can derive Big-O from unfamiliar code, by reading, in under a minute
- [ ] Can explain amortized O(1) append vs. genuinely O(n) front-insert
- [ ] Can set up and solve a simple recurrence relation (`T(n) = aT(n/b) + O(n^d)`) for a divide-and-conquer algorithm
- [ ] Given a problem's constraints (`n <= ...`), can immediately state the expected complexity class before designing an approach
- [ ] Can name which of the three space-time tradeoffs (hashing, preprocessing, caching) a given optimization is using

Next: [`02-python-dsa-toolkit-refresher.md`](./02-python-dsa-toolkit-refresher.md)
