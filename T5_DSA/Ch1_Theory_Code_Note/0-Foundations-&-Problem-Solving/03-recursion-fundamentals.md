# Recursion Fundamentals — Theory, Patterns & Interview Tricks

**Phase 0 — Setup** | Companion notebook: [`03-recursion-fundamentals.ipynb`](./03-recursion-fundamentals.ipynb) (naive vs. memoized Fibonacci measured at 657x, a real `RecursionError` demonstrated)

---

## Base case + recursive case

Every correct recursive function needs a base case (stops the recursion) and a recursive case (reduces the problem, calls itself). Calls go **down** the call stack until the base case, then returns unwind back **up** in reverse order — every one of those stack frames is real memory sitting in place until it returns, which is the O(depth) space cost that is easy to forget (Big-O notebook).

## The three recursion shapes

- **Linear recursion** — one recursive call per invocation (factorial, sum of a list, linked list traversal). Complexity is usually straightforward: O(depth) calls, O(depth) space.
- **Tree / branching recursion** — more than one recursive call per invocation (naive Fibonacci, any tree traversal, backtracking). Complexity is governed by the recurrence relation (Big-O notebook's Master Theorem section) — naive Fibonacci is `T(n) = T(n-1) + T(n-2) + O(1)`, which works out to O(2ⁿ) since each call spawns two more.
- **Mutual recursion** — two or more functions calling each other. Rare in interviews but worth recognizing when it appears (e.g. parsing expression grammars).

## Recursion → memoization → dynamic programming, as one pipeline

This is the single most important recursion-related interview connection. Naive tree recursion recomputes overlapping subproblems exponentially; caching each subproblem's result (`functools.lru_cache`, or a manual `dict`) turns it linear — **this cache-augmented recursion IS top-down dynamic programming.** Bottom-up DP (Phase 4) is the same recurrence, just computed iteratively from the base cases upward instead of recursively from the top down. Seeing naive recursion, memoized recursion, and iterative DP as three points on one spectrum — not three unrelated techniques — is worth being able to say explicitly in an interview.

```python
# naive: O(2^n)              # memoized: O(n), same recurrence, cached
def fib(n):                   @lru_cache(maxsize=None)
    if n <= 1: return n        def fib(n):
    return fib(n-1)+fib(n-2)       if n <= 1: return n
                                    return fib(n-1)+fib(n-2)
```

## The backtracking template (preview of Phase 3)

Nearly every backtracking problem (subsets, permutations, combinations, N-Queens, word search) fits one shape — worth internalizing now, since Phase 3 is just this template applied to increasingly specific constraints:

```
def backtrack(path, choices):
    if base_case_reached(path):
        record(path)
        return
    for choice in choices:
        if not valid(choice, path):        # pruning -- skip invalid branches early
            continue
        path.append(choice)                 # choose
        backtrack(path, remaining_choices)  # explore
        path.pop()                          # un-choose (backtrack)
```

The "choose → explore → un-choose" shape, and pruning invalid branches as early as possible, is the actual pattern being tested — the specific problem is just which `valid()` and `base_case_reached()` look like.

## Converting recursion to iteration

Any recursive traversal can be rewritten using an explicit stack instead of the call stack:

```python
stack = [start]
while stack:
    node = stack.pop()
    process(node)
    stack.extend(children_of(node))     # push children instead of recursing into them
```

This is exactly how iterative DFS is written (Phase 2–3), and it sidesteps Python's recursion depth limit entirely — worth reaching for whenever a recursive approach risks exceeding it (deep linked lists, deep trees, large graphs).

## No tail-call optimization in Python — a real, practical constraint

Some languages optimize a recursive call in tail position into a loop, using no extra stack space. CPython deliberately does not — every recursive call costs a real frame, tail position or not. The default recursion limit (~1000) is a genuine ceiling, not a theoretical one; a *correct* recursive solution can still fail at scale in Python specifically. Worth mentioning out loud when proposing a deeply recursive approach on a problem with a large input bound — proposing the iterative-with-explicit-stack alternative proactively reads as real engineering awareness.

---

## Interview Trick: Stating Recursive Complexity Precisely

"It's recursive, so it's exponential" is a common **wrong** reflex — plenty of recursive solutions are linear (any recursion with one call per level and O(1) work per level) or O(n log n) (divide-and-conquer with linear merge work, like merge sort). The correct habit: write down the recurrence relation first (`T(n) = a·T(n/b) + O(n^d)` or `T(n) = T(n-1) + T(n-2) + O(1)`, etc.), *then* state the complexity — not the other way around.

## Interview Trick: Recursion Tree as the Debugging/Explanation Tool

Sketching (or narrating) the recursion tree — how many calls at each depth, how much work per call — is simultaneously how you *derive* the complexity and how you *explain* it to an interviewer. "At depth k there are 2^k calls, each doing O(1) work outside their own recursive calls, and the tree has depth n, so total work is O(2ⁿ)" is a complete, precise answer that also happens to be the fastest way to spot that memoization would collapse the redundant branches.

---

## Common Gotchas / Rapid-Fire

- Forgetting the base case, or having an unreachable one, causes infinite recursion → `RecursionError`, not a hang (Python's stack limit catches it) — but a wrong base case that IS reachable, just wrong, produces a silently incorrect answer instead, which is worse.
- Slicing an input in a recursive call (`arr[1:]`) silently adds O(n) copying **per call**, changing the true complexity without changing what the code "looks like" it does — pass indices/bounds instead of slicing when this matters.
- Mutable default arguments (Python & DSA notebook 13) are an especially sharp trap in recursive helper functions specifically, since the same default object persists and accumulates across the *entire* chain of recursive calls, not just across separate top-level calls.
- A recursive solution that works on small test cases can still be exponential — always sanity-check against the constraints-based complexity expectation (Big-O notebook's constraint table) before considering it done.

---

## Self-Check

- [ ] Can identify base case and recursive case in unfamiliar recursive code immediately
- [ ] Can explain memoized recursion as literally top-down DP, not a separate technique
- [ ] Can recite the backtracking template (choose/explore/un-choose + pruning) from memory
- [ ] Can convert a simple recursive traversal to iterative using an explicit stack
- [ ] Can set up the recurrence relation for a recursive function BEFORE stating its complexity, not after guessing

Phase 0 complete. Next: **Phase 1 — Foundational Patterns**, starting with Arrays & Hashing.
