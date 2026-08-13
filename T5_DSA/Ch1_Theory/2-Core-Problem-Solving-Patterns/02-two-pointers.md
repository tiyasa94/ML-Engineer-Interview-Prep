# Two Pointers — Complete Theory, Patterns & Interview Mastery

**Phase 1 — Foundational Patterns, Topic 2** | Practice problems: NeetCode's Two Pointers section, after this document, not before.

Builds directly on Arrays & Hashing — several problems here are the *same underlying question* as a Topic 1 problem, solved differently once the input is sorted. That contrast is called out explicitly below, because it's one of the more valuable things to be able to say out loud in an interview.

---

# Part A — What the Two Pointers Technique Actually Is

Two index variables traverse a structure — usually an array or string, sometimes a linked list — instead of a nested loop checking every pair. The technique only works when the data has some **structural property** (almost always sortedness, or a monotonic relationship) that guarantees moving a pointer forward can never skip past the answer. Without such a property, two pointers is not a valid substitute for checking all pairs — this precondition is the single most important thing to verify before reaching for this pattern.

There are three distinct shapes, not one — worth telling apart precisely, since they have different correctness arguments and different failure modes.

## Shape 1: Opposite-direction (converging) pointers

One pointer starts at the left end, one at the right end, and they move toward each other based on a comparison, until they meet or cross. Requires the input to be **sorted** (or have an equivalent monotonic structure, like a palindrome check reading from both ends).

## Shape 2: Same-direction (fast/slow) pointers

Both pointers start at or near the beginning and move forward, but at different speeds or under different conditions — one advances every step, the other only advances when some condition holds. Used for in-place array modification (removing duplicates, partitioning) and for linked-list cycle detection.

## Shape 3: Merge-style pointers across two sequences

One pointer per sequence, both sequences sorted, advancing whichever pointer currently points at the smaller element. This is literally the merge step of merge sort, generalized to any "combine two sorted things" problem (merging, finding intersections, computing set operations on sorted arrays).

---

# Part B — Why This Works: The Correctness Arguments

Most explanations of two pointers state *that* it works without proving it — which is exactly the gap that shows up as hesitation when an interviewer asks "how do you know moving that pointer doesn't skip the answer?" Both arguments below are short and worth being able to reproduce from memory.

## Correctness of opposite-direction pointers on a sorted array (e.g. "two numbers that sum to target")

Array sorted ascending, `l` starts at index 0, `r` starts at the last index.

**Claim: if `arr[l] + arr[r] > target`, `r` can be safely decremented — no valid pair involving the current `r` and any index `≥ l` remains possible.**

Proof sketch: `arr[l]` is the *smallest* value among all currently-unexamined indices (`l..r`), because the array is sorted and everything before `l` has already been ruled out. If even the smallest available partner (`arr[l]`) paired with `arr[r]` overshoots the target, then EVERY other remaining index (all `≥ arr[l]`, since sorted) paired with `arr[r]` would overshoot by at least as much. So `r` cannot be part of any valid remaining pair — it is safe to discard entirely, not just skip past.

**Symmetric claim: if `arr[l] + arr[r] < target`, `l` can be safely incremented,** by the mirrored argument: `arr[r]` is the *largest* available partner, and if even that undershoots, no smaller partner paired with `l` will ever reach the target either.

This is a complete proof that the algorithm never discards a valid answer — each pointer move is justified by "the current candidate cannot possibly be part of ANY remaining valid pair," not just "let's try something else."

## Correctness of Floyd's cycle detection (fast/slow pointers on a linked list)

**Claim 1 — if a cycle exists, fast and slow are guaranteed to meet.** Once both pointers are inside the cycle, the gap between them (measured in steps around the cycle) decreases by exactly 1 every step, since fast gains one extra step of ground on slow per iteration. A strictly decreasing non-negative integer gap must hit 0 — so they meet, within at most one full cycle length of steps. They cannot skip over each other and miss a meeting, because the gap shrinks by exactly 1 at a time, never jumping past 0.

**Claim 2 — after they meet, restarting one pointer from the head finds the cycle's starting node.** Let `a` = distance from the list's head to the cycle's start, `b` = distance from the cycle's start to the meeting point, and `c` = the remaining distance around the cycle back to its start (so the cycle length is `b + c`). When they meet: slow has traveled `a + b` steps; fast has traveled `a + b + k(b+c)` for some integer `k ≥ 1` (fast has lapped the cycle `k` extra times). Since fast moves twice as fast as slow, fast's total distance is exactly double slow's:

```
2(a + b) = a + b + k(b + c)
a + b = k(b + c)
a = k(b + c) - b = (k-1)(b + c) + c
```

So `a` and `c` differ by a whole number of full cycle lengths — meaning if one pointer restarts from the head and another stays at the meeting point, both moving one step at a time, they will land on the cycle's starting node at exactly the same time, after `a` steps. This is not a coincidence to memorize; it falls directly out of the "fast moves twice as fast" setup.

---

# Part C — The Core Patterns

### 1. Opposite-Direction Pointers on a Sorted Array

```
l, r = 0, len(arr) - 1
while l < r:
    if condition(arr[l], arr[r]) == "too big":
        r -= 1
    elif condition(arr[l], arr[r]) == "too small":
        l += 1
    else:
        # found / record the answer
        l += 1; r -= 1     # (or however the specific problem needs to continue)
```
**Signals:** "sorted array", "pair/triplet that sums to...", "palindrome" (comparing from both ends inward).

### 2. Fixed Element + Two Pointers (extends Shape 1)

Fix one element with an outer loop, then run opposite-direction two pointers on the remainder — turns an O(n) problem into an O(n²) one, but that is still a full order of magnitude better than the O(n³) brute force the un-optimized three-nested-loop version would be.
```
for i in range(len(arr)):
    l, r = i + 1, len(arr) - 1
    while l < r:
        ...                  # exactly Pattern 1, on the subarray after i
```
**Signals:** "triplet"/"3Sum" style problems — any "k numbers that satisfy X" problem for small fixed k is usually (k-2) nested fixed loops wrapped around one Pattern-1 two-pointer pass.

### 3. Fast/Slow Pointers for In-Place Array Modification

Slow pointer marks "the boundary of the valid/processed region so far"; fast pointer scans ahead, and advances slow (writing into that position) only when a condition is met.
```
slow = 0
for fast in range(len(arr)):
    if keep(arr[fast]):
        arr[slow] = arr[fast]
        slow += 1
return slow                    # new length of the valid region
```
**Signals:** "remove duplicates in place", "move zeroes to the end", "remove all instances of value X" — anything asking for an in-place modification with O(1) extra space.

### 4. Fast/Slow Pointers for Cycle Detection (Floyd's algorithm)

```
slow = fast = head
while fast and fast.next:
    slow = slow.next
    fast = fast.next.next
    if slow == fast:
        # cycle found -- to locate the START of the cycle:
        slow2 = head
        while slow2 != slow:
            slow2 = slow2.next
            slow = slow.next
        return slow2            # the cycle's starting node
return None                     # no cycle
```
**Signals:** "linked list cycle", "duplicate number" problems reframed as implicit linked lists (an array where `arr[i]` is treated as "pointing to" index `arr[i]`), "find the start of a loop".

### 5. Merge-Style Pointers Across Two Sequences

```
i, j = 0, 0
result = []
while i < len(a) and j < len(b):
    if a[i] <= b[j]:
        result.append(a[i]); i += 1
    else:
        result.append(b[j]); j += 1
result.extend(a[i:]); result.extend(b[j:])   # append whichever sequence has leftovers
```
**Signals:** "merge two sorted...", "intersection of two sorted arrays", "given two sorted arrays, find...".

---

# Part D — Optimization Principles

- **The core trade this topic makes**: exploiting sortedness (or an equivalent monotonic property) to eliminate whole ranges of candidates per step, turning O(n²) pair-checking into O(n) — the array-scan analog of Arrays & Hashing's space-for-time trade, except this one trades *nothing* for time: two pointers is almost always **O(1) extra space**, which is its main advantage over a hashmap-based solution to the same problem.
- **Direct contrast worth stating explicitly in an interview**: Topic 1's Two Sum (unsorted input) needs a hashmap — O(n) time, O(n) space, because there's no structural property to exploit without one. Two Sum II (same problem, sorted input) needs no hashmap at all — O(n) time, **O(1) space** — because sortedness alone provides the elimination argument from Part B. The *same* problem, the *same* time complexity, but a strictly better space complexity, purely because of an input property. Recognizing "this is sorted, so I don't need the hashmap" is a genuinely strong interview signal.
- **When sorting first is still worth it**: if the input isn't sorted but the problem would otherwise reduce cleanly to Pattern 1, sorting costs O(n log n) up front — still strictly better than an O(n²) brute force, and still O(1) extra space if sorting in place. Whether to sort-then-two-pointer or hashmap-without-sorting is a real, explicit design decision worth narrating rather than defaulting to either one silently.
- **Duplicate-skipping as a correctness AND efficiency concern**: in fixed-element-plus-two-pointers problems (3Sum), after finding a valid triplet, advancing both `l` and `r` past any repeated values is necessary both to avoid duplicate answers in the output *and* to avoid wasted redundant work re-examining the same values.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "sorted array" + "pair"/"triplet"/"sum to target" | Opposite-direction pointers (Pattern 1/2) |
| "palindrome" | Opposite-direction pointers, comparing inward from both ends |
| "in place", "O(1) extra space", "remove"/"move" elements | Fast/slow, in-place modification (Pattern 3) |
| "linked list", "cycle", "loop" | Fast/slow, cycle detection (Pattern 4) |
| "merge two sorted", "intersection of sorted" | Merge-style two sequences (Pattern 5) |
| "container", "trapping water", "area between" | Opposite-direction pointers with a "move the shorter/limiting side" elimination argument (a direct variant of Part B's proof) |

If the input is **not sorted and cannot cheaply be sorted** (e.g. sorting would destroy needed index information the answer depends on), this is a signal two pointers may not apply directly, and Arrays & Hashing's hashmap approach, or a different topic entirely, is more likely correct.

---

# Part F — Common Gotchas & Interview Traps

- **Boundary condition**: `while l < r` vs `while l <= r` changes behavior at the point where the pointers meet — get this wrong and either a valid single-middle-element case is missed, or a pointer is compared against itself incorrectly. Decide deliberately, don't default by habit.
- **Forgetting to skip duplicate values** in triplet/quadruplet problems — after finding a valid combination, failing to advance past repeated values produces duplicate entries in the result, a very common source of "almost correct" submissions.
- **Applying two pointers to unsorted data without justification** — the single most common conceptual error; always state (even silently, to yourself) *what* structural property is being relied on before writing the pointer logic.
- **Linked list null-pointer checks**: `fast.next.next` without first checking `fast and fast.next` crashes on an odd-vs-even-length list at the boundary — the `while fast and fast.next:` guard in Part C.4 is not optional boilerplate, it is load-bearing.
- **Off-by-one when returning a new length after in-place modification** (Pattern 3) — `slow` at loop end IS the new length (since it was incremented past the last valid write), not `slow - 1`; a common source of an out-by-one wrong answer despite otherwise-correct logic.
- **Assuming fast/slow pointers must move at exactly speeds 1 and 2** — the *ratio* matters for the cycle-detection math in Part B, but other fast/slow problems (in-place modification) use "slow moves conditionally, fast moves every step," a different relationship entirely; conflating the two leads to copying the wrong template.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Opposite-Direction, Sorted Array:** Two Sum II (Input Array Is Sorted) → Valid Palindrome → 3Sum → 3Sum Closest → Container With Most Water → Trapping Rain Water

**Fast/Slow, In-Place Modification:** Remove Duplicates from Sorted Array → Remove Element → Move Zeroes → Sort Colors (Dutch National Flag — a 3-way partition, worth doing once the 2-way version is comfortable)

**Fast/Slow, Cycle Detection:** Linked List Cycle → Linked List Cycle II (find the start — implement Part B's proof directly) → Find the Duplicate Number (the same algorithm, reframed on an array instead of a linked list — a strong "did you actually understand the technique, or just the problem" test)

**Merge-Style, Two Sequences:** Merge Sorted Array → Intersection of Two Arrays II → Squares of a Sorted Array

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS). Unlike Arrays & Hashing, Two Pointers is **not a dedicated CLRS chapter** — it is a technique/pattern rather than a named data structure or single canonical algorithm, so CLRS does not treat it as its own topic. The one direct, honest CLRS touchpoint: **§2.3.1**, the `MERGE` procedure inside Merge Sort, which is exactly Part C's Pattern 5 (merge-style two pointers), described rigorously with a loop invariant proof — worth reading specifically for how CLRS structures a correctness argument, as a model for the proofs in Part B above.

Floyd's cycle-detection algorithm is classically attributed to Robert W. Floyd (1967) and is treated rigorously in Knuth, *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (in the context of cycle detection in iterated functions) — the standard reference if CLRS's absence of this topic leaves a gap worth filling with a textbook treatment rather than a course/video one.

**Video references** (search these directly by name/series — exact URLs not guessed at):
- **Abdul Bari** — clear coverage of array-based two-pointer problems and the underlying elimination-argument reasoning.
- **MIT OpenCourseWare, 6.006** — the Merge Sort lecture covers the `MERGE` step (Part C Pattern 5) with the same rigor as CLRS §2.3.1.
- **Tushar Roy (Coding Made Simple)** — has a specifically strong walkthrough of Floyd's cycle detection with the meeting-point math worked out visually, useful alongside Part B's derivation above.
- **NeetCode** — the Two Pointers section of the roadmap; use after an independent attempt, per the Problem-Solving Framework.

---

# Self-Check / Mastery Criteria

- [ ] Can state, and reproduce without notes, the elimination-argument proof for why moving a pointer on a sorted array never skips a valid answer
- [ ] Can derive (not just recite) why Floyd's algorithm guarantees a meeting point, and why restarting one pointer from the head finds the cycle start
- [ ] Can articulate the direct space-complexity contrast between Two Sum (Topic 1, hashmap, O(n) space) and Two Sum II (sorted, two pointers, O(1) space) as a single coherent explanation
- [ ] Can identify, within about a minute, which of the 5 core patterns a new problem statement maps to
- [ ] Knows the boundary-condition (`<` vs `<=`) decision is deliberate, not habitual, for every two-pointer loop written
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 3

Next: **Phase 1, Topic 3 — Sliding Window**.
