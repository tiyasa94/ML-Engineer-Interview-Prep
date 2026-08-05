# Backtracking — Complete Theory, Patterns & Interview Mastery

**Phase 3 — Search Over Structures, Topic 10** | Practice problems: NeetCode's Backtracking section, after this document, not before.

---

# Part A — What Backtracking Actually Is

Backtracking builds candidate solutions incrementally, and abandons ("backtracks" from) a partial candidate the moment it can be proven invalid or unable to lead to a valid solution — instead of completing it and checking only at the end. Structurally, it is **DFS over an implicit decision tree**: each node is a partial solution, each edge is one more choice made, and each leaf is a complete candidate. This is the full realization of the template previewed in Phase 0's Recursion document.

```
def backtrack(path, choices):
    if is_complete(path):
        record(path)
        return
    for choice in choices:
        if not is_valid(choice, path):     # pruning -- skip provably-invalid branches
            continue
        path.append(choice)                 # CHOOSE
        backtrack(path, remaining(choices, choice))   # EXPLORE
        path.pop()                          # UN-CHOOSE (the "backtrack" step itself)
```

---

# Part B — Why This Works, and Why It Can Be Fast in Practice

## Correctness: exhaustive search, with early elimination

Backtracking explores the *entire* search space in the worst case — same guarantee as brute force, so it never misses a valid solution. **Pruning** (the `is_valid` check) does not change this guarantee; it only skips branches that have been *proven* incapable of leading to a valid solution, so nothing correct is ever discarded. This is the same "provably cannot contribute to the answer" reasoning as Topic 2's two-pointer elimination argument and Topic 3/4's monotonic structures — applied here to entire subtrees of the search space instead of individual array positions.

## Why the "un-choose" step is not optional

After exploring a choice fully, it must be undone before trying the next sibling choice — otherwise the shared `path`/state object carries stale information into branches that never actually made that choice. This is a correctness requirement, not cleanup: without it, sibling branches see contaminated state, and the algorithm silently produces wrong answers rather than crashing (Part F elaborates on the specific, very common Python manifestation of this).

## Complexity: inherently exponential, and that is expected, not a failure

Backtracking problems typically have genuinely exponential search spaces: subsets have `2^n` possible answers, permutations have `n!`, combinations have `C(n,k)`. Backtracking does not change this worst-case asymptotic complexity — it reduces the **practical** runtime by pruning, often dramatically, without changing the big-O ceiling. Stating this honestly ("this is exponential in the worst case, and pruning reduces the practical constant, not the asymptotic class") is the correct, complete answer when a complexity question comes up here — unlike most of Phase 1's topics, "can this be sub-exponential" is often simply "no," and knowing that is itself the correct depth of understanding.

---

# Part C — The Core Patterns

### 1. Subsets (Include/Exclude Each Element)

```
def subsets(nums):
    result = []
    def backtrack(i, path):
        if i == len(nums):
            result.append(path[:])          # COPY -- see Part F
            return
        path.append(nums[i]); backtrack(i + 1, path); path.pop()    # include
        backtrack(i + 1, path)                                       # exclude
    backtrack(0, [])
    return result
```
**Signals:** "all subsets", "power set".

### 2. Permutations (Choose the Next Unused Element)

```
def permutations(nums):
    result = []
    def backtrack(path, used):
        if len(path) == len(nums):
            result.append(path[:])
            return
        for i, x in enumerate(nums):
            if used[i]: continue
            used[i] = True
            path.append(x); backtrack(path, used); path.pop()
            used[i] = False
    backtrack([], [False] * len(nums))
    return result
```
**Signals:** "all permutations", "all orderings/arrangements".

### 3. Combinations (Fixed Size, No Reuse, No Duplicates by Construction)

The **start-index technique** — only ever choosing from index `i` onward — is what prevents both re-using earlier elements and generating the same combination in a different order, without needing to check for duplicates after the fact.
```
def combinations(n, k):
    result = []
    def backtrack(start, path):
        if len(path) == k:
            result.append(path[:])
            return
        for i in range(start, n + 1):
            path.append(i); backtrack(i + 1, path); path.pop()
    backtrack(1, [])
    return result
```
**Signals:** "all combinations of size k".

### 4. Combination Sum (Target-Sum With Pruning)

Extends Pattern 3 with a running sum, pruning branches the moment they exceed the target (only valid when values are non-negative — sorting the input first makes this pruning maximally effective, since later elements can only make an already-too-large sum worse).
**Signals:** "combinations that sum to target", allowing or disallowing element reuse.

### 5. Constraint Satisfaction (N-Queens, Sudoku)

Place one piece/value per recursive level, checking validity (row/column/diagonal conflicts for N-Queens; row/column/box conflicts for Sudoku) *before* recursing deeper — this validity check IS the pruning, and is what keeps these problems tractable in practice despite their enormous raw search spaces.
**Signals:** "N-Queens", "Sudoku", any problem placing items under positional constraints.

### 6. Partitioning (e.g., Palindrome Partitioning)

Choose where to "cut" a sequence into valid pieces, recursing on the remainder after each cut.
**Signals:** "partition into...", "split such that each part is...".

### 7. Handling Duplicate Input Values

Sort the input first, then at each recursive level, **skip a choice if it is equal to the previous sibling choice already tried at this exact depth** — this specific technique (not deduplicating the final results afterward) is the standard, efficient way to avoid duplicate output when the input itself contains duplicate values.
```
for i in range(start, len(nums)):
    if i > start and nums[i] == nums[i - 1]:      # SAME depth, not "seen anywhere" -- a common confusion
        continue
    ...
```

---

# Part D — Optimization Principles

- **Pruning is the entire optimization story here** — sorting the input to enable early termination and duplicate-skipping, and checking validity *before* recursing rather than after building a complete (and doomed) candidate, are both instances of the same idea: eliminate provably-dead branches as early as possible.
- **Backtracking can upgrade to Dynamic Programming** the moment overlapping subproblems appear (the same partial state reachable via multiple different choice sequences) — worth recognizing this boundary explicitly; it is exactly the preview flagged in Phase 0's Recursion document and is the direct bridge into Phase 4.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "all subsets", "power set" | Subsets (Pattern 1) |
| "all permutations/arrangements/orderings" | Permutations (Pattern 2) |
| "all combinations of size k" | Combinations (Pattern 3) |
| "combinations that sum to target" | Combination Sum (Pattern 4) |
| "N-Queens", "Sudoku", positional constraints | Constraint Satisfaction (Pattern 5) |
| "partition into valid pieces" | Partitioning (Pattern 6) |
| "generate all", "find all possible" | Nearly always backtracking, some variant of the above |

---

# Part F — Common Gotchas & Interview Traps

- **Appending a shared mutable reference instead of a copy** — `result.append(path)` stores a *reference* to the same list object that keeps being mutated afterward; by the time the function returns, every entry in `result` points to the same, now-empty (or wrong) list. Always `result.append(path[:])` or `path.copy()`. This is, by a wide margin, the single most common backtracking bug in Python specifically.
- **Forgetting the un-choose step** (Part B) — state leaks between sibling branches, producing subtly wrong results that can still pass several test cases.
- **Duplicate-skipping at the wrong scope**: skipping "if this value was used anywhere before" instead of "if this value was already tried at this exact recursion depth" (Part C.7) either wrongly forbids valid reuse or fails to prevent duplicate output — the distinction between "seen globally" and "seen at this depth" is the crux of getting this right.
- **Off-by-one in the start index** for combinations — using `start` instead of `start + 1` for the recursive call allows reusing the same element; using `start + 1` when reuse *should* be allowed (Combination Sum without the "distinct" constraint) incorrectly forbids it.
- **No pruning at all** — technically correct, but often the difference between a solution that finishes and one that times out; always add validity/bound checks before recursing deeper, not just at the leaves.
- **Grid-based backtracking (Word Search) forgetting to un-mark a visited cell on backtrack** — the same un-choose requirement (Part B), applied to grid state instead of a path list.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Subsets:** Subsets → Subsets II (duplicates — apply Part C.7)

**Permutations:** Permutations → Permutations II (duplicates)

**Combinations:** Combinations → Combination Sum → Combination Sum II (duplicates, no reuse)

**Constraint Satisfaction:** N-Queens → Sudoku Solver

**Partitioning:** Palindrome Partitioning

**Grid-Based (bridges into Phase 3's Graphs topic):** Word Search (revisit alongside Topic 8's Tries for the multi-word variant, Word Search II)

**Revisit With This Lens:** Generate Parentheses (Topic 4, Stack — re-examine as a backtracking problem: track unmatched opens as the pruning condition)

---

# Part H — References

**Primary text:** CLRS does **not** have a dedicated "Backtracking" chapter — like Two Pointers and Sliding Window, this is a technique/pattern rather than a single named algorithm CLRS catalogs. The most rigorous classical reference for backtracking and combinatorial search specifically is Knuth, *The Art of Computer Programming, Volume 4A: Combinatorial Algorithms* — considerably deeper than interview prep requires, but the definitive text if the fullest possible treatment is wanted. A more approachable, still-rigorous alternative: Skiena, *The Algorithm Design Manual*, which covers backtracking and pruning with a practical, problem-solving orientation closer to this document's style.

**Video references** (search directly by name/series):
- **Abdul Bari** — clear coverage of N-Queens and subset/permutation generation.
- **Back To Back SWE** — a particular strength of this channel; detailed walkthroughs of the harder backtracking problems (Sudoku Solver, Word Search).
- **NeetCode** — the Backtracking section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can write the choose/explore/un-choose template from memory and explain why each of the three steps is necessary
- [ ] Can state, honestly, that backtracking problems are typically exponential in the worst case, and explain what pruning does and does not change about that
- [ ] Knows to copy (not reference) a mutable path before adding it to a results list, and can explain why the bug happens if this is skipped
- [ ] Can implement the start-index technique for combinations and explain why it prevents both reuse and duplicate ordering
- [ ] Can implement the "skip duplicates at this depth" technique and explain why it differs from "skip if seen anywhere"
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 11

Next: **Phase 3, Topic 11 — Graphs**.
