# Binary Search — Complete Theory, Patterns & Interview Mastery

**Phase 1 — Foundational Patterns, Topic 5 (final topic of Phase 1)** | Practice problems: NeetCode's Binary Search section, after this document, not before.

---

# Part A — What Binary Search Actually Is

Binary search repeatedly halves a search space by comparing its middle element against a target, discarding the half that provably cannot contain the answer. The precondition is **not**, strictly, "the array must be sorted" — that is the classic special case. The true, general precondition is **monotonicity**: some way of looking at any candidate and determining, definitively, whether the answer lies to its left or its right (or whether that candidate itself satisfies some property that only holds within a contiguous range). Sortedness is the most common source of this monotonicity, but far from the only one — Part C's "binary search on the answer" pattern is built entirely on this distinction, and is the single most valuable generalization in this entire topic.

Complexity: **O(log n)** — the direct realization of Phase 0's "halving" complexity class.

---

# Part B — Why This Works: Loop Invariant Proof & Complexity Derivation

## Correctness, via loop invariant (CLRS's own methodology)

CLRS proves algorithm correctness using a three-part **loop invariant** argument (introduced with insertion sort in Chapter 2): state a property that holds before the loop starts, show it's preserved by each iteration, and show it implies correctness when the loop ends. Binary search is one of the cleanest possible examples of this method:

**Invariant**: *if the target exists in the array, it lies within `arr[lo..hi]`.*

- **Initialization**: before the first iteration, `lo=0, hi=n-1` — the invariant holds trivially, since `arr[lo..hi]` is the entire array.
- **Maintenance**: each iteration computes `mid`, compares `arr[mid]` to the target, and shrinks the range to either `arr[lo..mid-1]` or `arr[mid+1..hi]` — specifically discarding only the half that, given the array is sorted, **cannot** contain the target if `arr[mid]` doesn't equal it. The invariant is preserved by construction: the target is never in the discarded half.
- **Termination**: the loop ends when `lo > hi` (search space empty) or `arr[mid]` equals the target. By the invariant, if the target existed, it would still be within `arr[lo..hi]` — an empty range at termination means it does not exist. This is a complete, watertight correctness argument, not an appeal to intuition.

## Complexity, via recurrence

Each step does O(1) work (one comparison) and recurses into a problem of size `n/2`: `T(n) = T(n/2) + O(1)`. Applying Phase 0's Master Theorem reasoning (`a=1, b=2, d=0`, and `log_2(1) = 0 = d`) gives `T(n) = O(log n)` directly. Concretely worth having ready: for `n = 10^6`, binary search takes at most about 20 comparisons (`log_2(10^6) ≈ 19.9`) — a number worth being able to produce on the spot when asked "how fast is this, really."

---

# Part C — The Core Patterns

### 1. Classic Binary Search (exact match in a sorted array)

```
lo, hi = 0, len(arr) - 1
while lo <= hi:
    mid = lo + (hi - lo) // 2        # NOT (lo + hi) // 2 -- see Part F on overflow
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        lo = mid + 1
    else:
        hi = mid - 1
return -1                              # not found
```
**Signals:** "sorted array", "find target", "find index of".

### 2. Boundary Search (leftmost / rightmost occurrence)

A different template from Pattern 1 — instead of stopping the moment a match is found, keep searching *past* it in a specific direction to find the first or last occurrence. This is exactly what `bisect_left`/`bisect_right` (Phase 0's DSA toolkit) implement.
```
def leftmost(arr, target):
    lo, hi = 0, len(arr)              # note: hi = len(arr), not len(arr)-1, for this template
    while lo < hi:                     # note: strict <, not <=
        mid = lo + (hi - lo) // 2
        if arr[mid] < target:
            lo = mid + 1
        else:
            hi = mid                   # do NOT exclude mid -- it might BE the answer
    return lo
```
**Signals:** "first/last position", "leftmost/rightmost", "insertion point", "how many elements are ≤/< x".

### 3. Modified Binary Search on Rotated/Structured Arrays

The array isn't fully sorted, but retains enough structure that, at any `mid`, at least one of the two halves IS guaranteed to be properly sorted — determine which half is sorted (by comparing `arr[lo]` to `arr[mid]`), then check whether the target falls within that sorted half's range to decide which half to keep.
**Signals:** "rotated sorted array", "sorted then rotated", "pivot".

### 4. Binary Search on the Answer — the Big Generalization

**This is the pattern most people never fully generalize to, and it is where "hard" binary search problems live.** The array being searched is not literally the input at all — it's the **space of possible answers** to an optimization question ("minimum X such that condition(X) holds" / "maximum X such that condition(X) holds"), where `condition()` is monotonic in `X` (once true, it stays true for all larger — or all smaller — `X`).

```
def can_do_it(capacity):               # a monotonic YES/NO feasibility check, usually O(n) itself
    ...
    return True_or_False

lo, hi = min_possible_answer, max_possible_answer
while lo < hi:
    mid = lo + (hi - lo) // 2
    if can_do_it(mid):
        hi = mid                        # mid works -- but can something smaller also work?
    else:
        lo = mid + 1                    # mid doesn't work -- need something bigger
return lo                                # the smallest value for which can_do_it() is True
```
**The mental shift this requires**: stop asking "what index am I searching?" and start asking "what is the RANGE of possible answers, and is there a yes/no check on a candidate answer that is monotonic?" — the moment a problem can be rephrased as "find the smallest/largest value such that some check passes," and that check's result flips exactly once as the value increases, binary search applies, whether or not anything resembling a sorted array is anywhere in sight.
**Signals:** "minimize the maximum...", "maximize the minimum...", "smallest/largest value such that...", "find the minimum capacity/speed/threshold such that [task] can be completed within [constraint]".

### 5. Binary Search on a 2D Matrix

If a matrix is sorted such that each row is sorted and the first element of each row is greater than the last element of the previous row, it is really a 1D sorted array in disguise — apply Pattern 1 directly, converting a flat index `i` to `(i // num_cols, i % num_cols)`. If rows and columns are independently sorted but the matrix as a whole is not flat-sortable, a different technique (staircase search, starting from a corner) applies instead.

---

# Part D — Optimization Principles

- **The headline win**: O(n) linear scan → O(log n), the direct payoff of exploiting monotonicity, exactly parallel to Two Pointers needing sortedness to earn its speedup.
- **Recognizing binary search applies even with no explicit array**: Pattern 4 is the concrete instance of "monotonicity is the true precondition, not literal sortedness" — the single highest-value reframe in this document.
- **The common O(n log n) shape**: "binary search on the answer" problems typically combine an O(log(range)) outer binary search with an O(n) feasibility check inside each iteration, for O(n log(range)) total — worth stating this composed complexity explicitly rather than leaving "how fast is the whole thing" implicit.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "sorted array", "find target/index" | Classic Binary Search (Pattern 1) |
| "first/last position", "leftmost/rightmost", "insertion point" | Boundary Search (Pattern 2) |
| "rotated sorted array", "pivot" | Modified Binary Search (Pattern 3) |
| "minimize the maximum...", "maximize the minimum...", "smallest/largest value such that..." | Binary Search on the Answer (Pattern 4) — the strongest, most-overlooked signal in this entire document |
| "search a 2D matrix", "sorted matrix" | 2D Binary Search (Pattern 5) |
| Explicit "in O(log n) time" in the problem statement | Any of the above — a direct hint the intended solution is binary search, even if the problem doesn't otherwise look like a search |

---

# Part F — Common Gotchas & Interview Traps (Binary Search Is Infamous for These)

- **`(lo + hi) // 2` vs `lo + (hi - lo) // 2`**: mathematically identical in Python (arbitrary-precision integers), but `(lo + hi)` can **overflow** in fixed-width-integer languages (C++, Java) — worth using the overflow-safe form as a habit regardless of language, and worth mentioning this awareness explicitly if the interview is in a language where it matters.
- **`lo <= hi` vs `lo < hi`, and matching boundary updates**: Pattern 1 (exact match) uses `<=` with `hi = mid - 1` / `lo = mid + 1`. Pattern 2 (boundary search) uses `<` with `hi = mid` (not `mid - 1`) when `mid` might itself be the answer. **Mixing these two templates is the single most common source of binary search bugs** — infinite loops (when `mid` never gets excluded on a branch that should exclude it) or off-by-one wrong answers (when a valid boundary gets excluded incorrectly).
- **Infinite loop from a stuck `mid`**: with floor division, if `lo = hi - 1`, `mid` computes to `lo` — if the update on that branch sets `lo = mid` (instead of `mid + 1`) or `hi = mid` (instead of `mid - 1`) incorrectly, the range never shrinks and the loop never terminates. Always double check: does the chosen branch's update **exclude** `mid` when `mid` has been ruled out, or only when appropriate?
- **Empty array / single element**: verify manually before trusting any binary search implementation on the smallest possible inputs — a very common source of "works on the example, fails on edge cases."
- **Rotated array with duplicates**: when `arr[lo] == arr[mid] == arr[hi]`, it becomes impossible to determine which half is properly sorted in O(1) — the worst case degrades to O(n) (must shrink the range by one at a time instead of by half) — worth knowing this edge case exists and being able to state it honestly rather than claiming a guaranteed O(log n) that doesn't actually hold with duplicates present.
- **Binary search on the answer, wrong direction**: getting `if can_do_it(mid): hi = mid` backwards (should it be `lo = mid` instead, searching for a *maximum* feasible value rather than a *minimum* one?) is the most common Pattern-4-specific bug — always work out, on paper, which direction the feasibility check should push the boundary *before* writing the loop.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Classic:** Binary Search → Search Insert Position

**Boundary Search:** Find First and Last Position of Element in Sorted Array

**Modified/Rotated:** Search in Rotated Sorted Array → Search in Rotated Sorted Array II (duplicates — the O(n) worst-case edge case from Part F) → Find Minimum in Rotated Sorted Array

**Binary Search on the Answer (do these deliberately, as the highest-value pattern in this document):** Koko Eating Bananas → Capacity To Ship Packages Within D Days → Split Array Largest Sum → Sqrt(x)

**2D:** Search a 2D Matrix

**Hardest classic binary search problem, attempt only once everything above is comfortable:** Median of Two Sorted Arrays

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS). Binary search itself appears as a canonical exercise in **Chapter 2** ("Getting Started" — the same chapter that introduces the loop-invariant proof methodology used in Part B, via insertion sort), asking for exactly the O(log n) proof worked through above. The formal machinery for solving its recurrence — the substitution method, recursion-tree method, and the Master Theorem — is **Chapter 4, "Divide-and-Conquer"** (§4.3–4.5), the same Master Theorem already introduced informally in Phase 0's Big-O document.

**Video references** (search these directly by name/series — exact URLs not guessed at):
- **Abdul Bari** — clear coverage of binary search fundamentals and, notably, several "binary search on the answer" style problems, which is the pattern most other resources under-cover.
- **MIT OpenCourseWare, 6.006** — an early lecture on the search problem, formalizing binary search's correctness and complexity with the same rigor as CLRS.
- **Back To Back SWE** — a good walkthrough specifically of Median of Two Sorted Arrays, the hardest problem in Part G.
- **NeetCode** — the Binary Search section of the roadmap; use after an independent attempt, per the Problem-Solving Framework.

---

# Self-Check / Mastery Criteria

- [ ] Can produce the full three-part loop invariant proof (initialization, maintenance, termination) for classic binary search, unaided
- [ ] Can derive O(log n) from the recurrence `T(n) = T(n/2) + O(1)`, not just quote the result
- [ ] Can explain the difference between the exact-match template (`<=`) and the boundary-search template (`<`), and why mixing them causes bugs
- [ ] **Can recognize a "binary search on the answer" problem that does not mention arrays or sorting at all** — the strongest single test of whether this topic has actually generalized past "search a sorted array"
- [ ] Knows the rotated-array-with-duplicates edge case degrades to O(n), and can state why honestly rather than overclaiming O(log n) unconditionally
- [ ] Has independently solved at least one problem from each pattern category in Part G, including at least two from the Binary-Search-on-the-Answer group specifically

---

# Phase 1 Complete

All five Foundational Patterns are done: Arrays & Hashing → Two Pointers → Sliding Window → Stack → Binary Search. Before moving on, worth doing one pass back through all five Self-Check sections cold, since Phase 2 (Linked Lists, Trees, Tries, Heaps) builds on several of these directly — fast/slow pointers reappear immediately in linked list cycle problems, and monotonic-structure reasoning reappears in heap-based problems.

Next: **Phase 2, Topic 6 — Linked List**.
