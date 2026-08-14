# Sliding Window — Complete Theory, Patterns & Interview Mastery

**Phase 1 — Foundational Patterns, Topic 3** | Practice problems: NeetCode's Sliding Window section, after this document, not before.

Sliding Window is, structurally, a *specific application* of Topic 2's same-direction (fast/slow) pointers, extended with the idea of maintaining running state — a sum, a count, a frequency map — over the range between the two pointers. If Two Pointers wasn't fully solid, that gap will show up here.

---

# Part A — What Sliding Window Actually Is

A **window** is a contiguous range `[left, right]` over an array or string. The technique maintains this range as it "slides" across the input — expanding (`right` moves forward) and/or contracting (`left` moves forward) — while tracking some aggregate over the current window (a sum, a count, a set of characters), so that every contiguous subarray/substring of interest is examined without ever recomputing that aggregate from scratch.

**The critical precondition, easy to state and easy to forget under pressure: sliding window only applies to CONTIGUOUS ranges** — subarrays and substrings, never arbitrary subsequences or subsets. A problem about subsequences (elements can be skipped, order preserved) is a Dynamic Programming problem (Phase 4), not a sliding window problem, no matter how similar the wording looks.

Two shapes:

## Fixed-size window

The window size `k` is given directly by the problem. Slide by exactly one position at a time: remove the element leaving on the left, add the element entering on the right, update the aggregate incrementally.

## Variable-size (dynamic) window

The window has no fixed size — it grows and shrinks based on whether its current contents satisfy some condition. The classic shape: expand `right` to grow the window; when it becomes invalid (or, depending on the problem, once it becomes valid), shrink from `left` until it's valid again (or until shrinking further would break it), recording the best window seen along the way.

---

# Part B — Why This Is O(n), Not O(n²): The Amortized Argument

A variable-size sliding window looks, at first glance, like a for-loop with a nested while-loop — which reads as O(n²) on sight. It is not, and understanding precisely *why* is the single most important piece of theory in this topic, because it is exactly the kind of thing an interviewer asks to check real understanding versus memorized code.

**The argument:** both `left` and `right` move **only forward**, never backward, across the *entire* run of the algorithm — not just within one outer-loop iteration. `right` advances at most `n` times total (once per outer loop iteration, by construction). `left` also advances at most `n` times total, summed across *every* inner while-loop's execution combined — because `left` never resets backward, each of the `n` possible positions for `left` is visited at most once, ever, over the whole algorithm. Total work is therefore `(at most n moves of right) + (at most n moves of left)` = O(n), not O(n) *per* outer iteration.

This is a genuine instance of **amortized analysis** (Phase 0, and formally CLRS Chapter 17) — the *aggregate method* specifically: bound the total cost of a sequence of operations as a whole, rather than bounding each operation individually and multiplying. Individual inner-loop iterations are not each O(1) in a way that's obvious from a single outer-loop pass, but the *aggregate* count of inner-loop iterations across the whole algorithm is bounded by `n`, giving O(n) total, O(1) amortized per outer step.

## The correctness precondition: monotonic validity

Sliding window's "expand right, shrink left until valid again" structure is only *correct* when the window's validity is **monotonic** in a specific sense: once a window `[l, r]` becomes invalid, it stays invalid for every `r' > r` at that same `l` — i.e., adding more elements without removing any cannot fix an invalid window; only shrinking from the left can. If a problem's validity condition does not have this property (adding an element could sometimes fix an otherwise-invalid window), the "never re-expand after shrinking" logic is unsound, and sliding window does not directly apply — this needs to be checked for a new problem, not assumed, the same way Two Pointers required checking for sortedness first.

---

# Part C — The Core Patterns

### 1. Fixed-Size Window: Incremental Aggregate Update

```
window_sum = sum(arr[0:k])                 # O(k) once, to initialize
best = window_sum
for r in range(k, len(arr)):
    window_sum += arr[r] - arr[r - k]      # add entering element, remove leaving element -- O(1)
    best = max(best, window_sum)
```
This is the sliding-window special case of Topic 1's **prefix sum** technique — instead of computing `prefix[r] - prefix[l]` from a precomputed array, the window sum is maintained incrementally as it moves, which is cheaper when only ever examining *adjacent* windows (as sliding always does) rather than arbitrary `(l, r)` range queries.
**Signals:** "subarray of size k", "every window of size k", "k consecutive elements".

### 2. Variable-Size Window: Expand, Then Shrink While Invalid

```
left = 0
window_state = initialize()                # e.g. a running sum, or a frequency dict
best = ...
for right in range(len(arr)):
    add(arr[right], window_state)            # expand
    while not is_valid(window_state):         # shrink until valid again
        remove(arr[left], window_state)
        left += 1
    best = update_best(best, left, right)      # record the best window seen at THIS right
```
**Signals:** "longest/shortest substring/subarray such that...", "at most K distinct", "sum less than/at least target".

### 3. Variable-Size Window With a Frequency Map

The most common concrete instance of Pattern 2 — the window's "state" is a `dict`/`Counter` of element frequencies, and validity is a condition on that map (number of distinct keys, whether all required characters are present, etc.).
```
freq = {}
for right in range(len(arr)):
    freq[arr[right]] = freq.get(arr[right], 0) + 1
    while <condition on freq is violated>:
        freq[arr[left]] -= 1
        if freq[arr[left]] == 0:
            del freq[arr[left]]              # keep the map's KEY COUNT accurate, not just values
        left += 1
```
**Signals:** "longest substring without repeating characters", "longest substring with at most K distinct characters", "minimum window substring containing all of...".

### 4. Monotonic Deque for Sliding Window Maximum/Minimum

A distinct, more advanced technique — maintaining the max (or min) of a fixed-size window in O(1) amortized per step, without a heap. Maintain a `deque` of **indices**, kept such that their corresponding values are in strictly decreasing order from front to back.

```
from collections import deque
dq = deque()                    # stores INDICES, values at those indices are decreasing front-to-back
result = []
for i, x in enumerate(arr):
    while dq and arr[dq[-1]] <= x:      # correctness: a smaller, EARLIER element can never
        dq.pop()                         # be the max again once a larger, LATER element exists
    dq.append(i)
    if dq[0] <= i - k:                   # front index has fallen out of the current window
        dq.popleft()
    if i >= k - 1:
        result.append(arr[dq[0]])         # front of deque = current window's max
```

**Correctness of the eviction rule**: if `arr[j] <= arr[i]` for some earlier index `j < i`, then `arr[j]` can never again be the maximum of any window that still contains index `i` — a later, larger-or-equal element makes every earlier, smaller-or-equal element permanently irrelevant, for as long as both remain in the window. Discarding it is not an approximation, it is provably safe. Because every index is pushed once and popped **at most once** across the entire run, total deque operations are bounded by `2n` — the same amortized argument as Part B, applied to a different structure.

---

# Part D — Optimization Principles

- **The core trade this topic makes**: incremental aggregate maintenance instead of recomputing from scratch per window — turning a naive O(n·k) (fixed window) or O(n²) (variable window, recomputing validity from scratch each time) into O(n). This is the sliding-window instance of the same "avoid redundant recomputation" theme running through prefix sums (Topic 1) and memoization (Phase 0's recursion notebook) — worth naming explicitly as the same underlying idea reappearing in a new shape.
- **Choosing the window state structure deliberately**: a running number (sum/count) suffices when only an aggregate matters; a frequency map is needed the moment *which* elements are present matters (distinct-count problems, "contains all of X" problems); a monotonic deque is needed specifically for running max/min, where neither a plain aggregate nor a frequency map gives an O(1)-per-step answer.
- **The `while` vs `if` decision when shrinking**: shrinking must be a `while` loop, not a single `if`, whenever more than one element might need to leave the window to restore validity — a common source of subtly wrong output that still passes several test cases before failing on one requiring multiple shrinks in a row.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "subarray of size k", "every window of size k" | Fixed-size window (Pattern 1) |
| "longest/shortest substring/subarray such that..." | Variable-size window (Pattern 2/3) |
| "at most/exactly/at least K distinct" | Variable-size window with a frequency map (Pattern 3) |
| "contains all characters of", "minimum window containing..." | Variable-size window with a frequency map, tracking "how many required conditions are currently satisfied" |
| "maximum/minimum of every window of size k" | Monotonic deque (Pattern 4) |
| **"subsequence" (not "substring"/"subarray")** | **Not sliding window** — contiguity is broken, this is Dynamic Programming (Phase 4) territory instead |

The subsequence-vs-substring distinction is worth over-emphasizing: it is the most common reason a fundamentally sound sliding-window instinct gets applied to a problem it cannot solve.

---

# Part F — Common Gotchas & Interview Traps

- **Subsequence vs. substring/subarray** (Part E) — the single most consequential mistake in this topic; always confirm contiguity is actually required before starting.
- **Window length off-by-one**: the window `[left, right]` (inclusive both ends) has length `right - left + 1`, not `right - left` — a very common source of an answer that's off by exactly one.
- **`if` instead of `while` when shrinking** (Part D) — silently wrong on inputs needing more than one shrink step per expansion.
- **Frequency map cleanup**: forgetting to `del` a key once its count hits zero corrupts any logic that checks "number of distinct keys currently in the map" — the map still reports a key as present with count 0, inflating the distinct count.
- **Minimum window substring-style problems specifically**: the validity check needs to track "how many of the *required, distinct* characters are currently satisfied in sufficient quantity," not just "total character count" — a frequent, subtle bug is checking total counts instead of the number of *distinct required characters* whose individual counts have been met.
- **Monotonic deque storing values instead of indices**: without the index, there is no way to check whether the front of the deque has aged out of the current window (`dq[0] <= i - k`) — this check is impossible to express correctly with values alone.
- **Initializing a fixed-size window incorrectly**: forgetting the O(k) initialization step before the incremental O(1) updates begin, or starting the sliding loop from the wrong index (`k`, not `k-1` or `0`).

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Fixed-Size Window:** Maximum Average Subarray I → Sliding Window Maximum (monotonic deque — do this one specifically to implement Part C.4's proof directly, not just call a library)

**Variable-Size, Simple Aggregate:** Minimum Size Subarray Sum → Longest Subarray with Sum at Most K

**Variable-Size, Frequency Map:** Longest Substring Without Repeating Characters → Longest Repeating Character Replacement → Fruit Into Baskets (equivalently, "Longest Substring with At Most 2 Distinct Characters") → Permutation in String

**Variable-Size, Hardest Tier:** Minimum Window Substring (the canonical "hard" sliding window problem — combines a frequency map with the "distinct required characters satisfied" counting trap from Part F; worth attempting only after every problem above feels comfortable)

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS). Like Two Pointers, "Sliding Window" is **not a named CLRS chapter** — it is a modern interview-pattern label for a technique, not a classical algorithm CLRS catalogs by that name. The honest, rigorous CLRS touchpoint for the *theory underneath* this entire topic is **Chapter 17, "Amortized Analysis"** — specifically the aggregate method, which is precisely the argument used in Part B to justify the O(n) claim, and the accounting/potential methods (§17.2–17.3) which formalize the same "some individual steps are expensive, but the total across all steps is bounded" reasoning used again in Part C.4's monotonic deque.

**Video references** (search these directly by name/series — exact URLs not guessed at):
- **Abdul Bari** — strong on the amortized-analysis theory (Chapter 17 territory) that underpins why sliding window is O(n).
- **Back To Back SWE** — has a specifically well-regarded, detailed walkthrough of Minimum Window Substring, the hardest problem in Part G, working through the frequency-map counting trap explicitly.
- **MIT OpenCourseWare, 6.006** — the amortized analysis lecture, for the formal aggregate/accounting/potential method treatment behind Part B.
- **NeetCode** — the Sliding Window section of the roadmap; use after an independent attempt, per the Problem-Solving Framework.

---

# Self-Check / Mastery Criteria

- [ ] Can explain, precisely, why a nested for/while sliding window loop is O(n) and not O(n²) — the aggregate-movement argument, not just "it's known to be O(n)"
- [ ] Can state the monotonic-validity precondition that must hold for sliding window to be correct, and check for it before applying the technique
- [ ] Can explain why the monotonic deque's eviction rule is provably correct, not just recite the code
- [ ] Can distinguish, on sight, a substring/subarray (contiguous — sliding window territory) problem from a subsequence (Dynamic Programming territory) problem
- [ ] Knows when to reach for a plain running aggregate vs. a frequency map vs. a monotonic deque, for a new problem, within about a minute
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 4

Next: **Phase 1, Topic 4 — Stack**.
