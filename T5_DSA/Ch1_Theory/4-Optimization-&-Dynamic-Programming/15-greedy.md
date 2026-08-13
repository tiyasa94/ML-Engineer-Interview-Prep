# Greedy — Complete Theory, Patterns & Interview Mastery

**Phase 4 — DP, Intervals, Greedy, Topic 15** | Practice problems: NeetCode's Greedy section, after this document, not before.

---

# Part A — What Greedy Actually Is, and Its Central Risk

A greedy algorithm makes the locally optimal choice at each step, **without reconsidering it later**, hoping this leads to a globally optimal solution. Unlike Phase 3's Backtracking (which can undo any choice) and Topic 13's DP (which considers all choices and combines them), greedy **commits with no do-overs**. This is either exactly the right tool (when provably safe) or silently wrong (when not) — greedy correctness must be **proven**, per problem, never assumed from intuition or from passing a few test cases.

---

# Part B — The Theory: When Greedy Is Actually Correct

## The two required properties (the greedy mirror of DP's two properties)

- **Greedy-choice property**: a globally optimal solution can be reached by making the locally optimal choice *first*, then optimally solving whatever remains — **without ever needing to reconsider that first choice**. This is what distinguishes greedy from DP: DP considers every choice and combines the best outcomes; greedy commits to exactly one choice, provably without loss.
- **Optimal substructure**: same requirement as DP (Topic 13) — the remaining subproblem, after the greedy choice, must itself be optimally solvable the same way.

## Proving greedy correctness: the exchange argument

The standard proof technique (previewed in Topic 14's activity-selection proof): assume an optimal solution that does **not** make the greedy choice; show it can always be modified ("exchanged") to include the greedy choice instead, without making the result worse. This establishes the greedy choice is always at least as good as any alternative — a complete correctness proof, not an appeal to intuition.

## The canonical cautionary tale: fractional vs. 0/1 knapsack (directly from CLRS)

CLRS uses this exact pair of problems (§16.2) to illustrate precisely when greedy works and when it doesn't:
- **Fractional knapsack** (items can be split): greedily taking items in order of highest value-per-weight ratio **is** provably optimal — the greedy-choice property holds, since a partial item can always be exchanged for a fractionally better one without loss.
- **0/1 knapsack** (items must be taken whole or not at all): the identical greedy strategy (highest value-per-weight first) **fails** — a high-ratio item that doesn't fit can force a worse combination than a more careful, non-greedy selection would achieve. The greedy-choice property does not hold here; DP (Topic 16) is required instead.

This single contrast — same-looking problem, same-looking greedy strategy, provably correct in one case and provably wrong in the other — is the most important thing to internalize in this entire topic: **greedy "looking reasonable" is never sufficient justification.**

---

# Part C — The Core Patterns

### 1. Interval Scheduling (Direct Extension of Topic 14)

Activity selection (Topic 14, Pattern 3) is this topic's foundational, fully-proven example — revisit its exchange argument as the template for reasoning about every other greedy problem here.

### 2. Sorting-Based Greedy

Sort by some criterion suggested by the problem, then make sequential greedy choices based on that order — the general shape most greedy problems take, echoing Topic 14's "choose the right sort key" design decision.
**Signals:** "maximize/minimize", with a natural sort criterion suggesting itself (by size, by ratio, by deadline).

### 3. Sequential Feasibility / Reachability Greedy

At each position, greedily track the furthest reachable point (or similar running bound) achievable so far, without needing to explore alternate paths explicitly.
**Signals:** "jump game" (can you reach the end / minimum jumps to reach the end), "gas station" (can a circular route be completed).

### 4. Greedy + Heap (Direct Extension of Topic 9)

When the "next best choice" changes dynamically as items are processed, a heap maintains it efficiently — already covered structurally in Topic 9, Pattern 5.
**Signals:** "scheduling", "task", "priority", revisited from Topic 9's list.

### 5. Known Greedy Failures — Worth Memorizing as Explicitly as the Successes

- **0/1 Knapsack** (Part B) — needs DP, not greedy.
- **Dijkstra's shortest path with negative edge weights** (Topic 12) — the greedy finalize-on-pop step is provably unsound the moment a negative edge exists; Bellman-Ford is required instead.

Knowing *where greedy fails*, specifically and by name, is as valuable an interview signal as knowing where it succeeds — it demonstrates the boundary is understood, not just the technique.

---

# Part D — Optimization Principles

- **When correct, greedy is typically strictly cheaper than the DP solution to a similarly-shaped problem** — often `O(n log n)` (dominated by an initial sort) or `O(n)`, versus DP's polynomial-but-larger complexity for a problem that does require considering multiple choices. This is precisely *why* proving greedy correctness (when possible) is valuable, rather than defaulting to the more general, more expensive DP approach for everything.
- **The actual "optimization" this topic is testing**: recognizing when a cheaper, greedy algorithm can *provably* replace a more expensive, more general DP one — not just writing a plausible-looking greedy loop.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "maximize/minimize" + an obvious sort criterion | Sorting-Based Greedy (Pattern 2) |
| "can you reach", "minimum jumps", "complete the circuit" | Sequential Feasibility (Pattern 3) |
| Interval/scheduling-flavored | Interval Scheduling (Pattern 1), strong correlation with greedy generally |
| A locally "obviously best" choice feels available | **Verify with an exchange argument or a deliberate counter-example search before trusting this — do not skip this step** |

---

# Part F — Common Gotchas & Interview Traps

- **Assuming greedy works without proof or counter-example testing** — the single largest risk in this entire topic; a greedy approach that produces correct answers on several hand-checked examples can still be wrong in general (0/1 knapsack, Part B, is the textbook cautionary tale).
- **Confusing "a greedy-looking approach" with "a provably correct greedy algorithm"** — always attempt either the exchange argument or a deliberate search for a counter-example before committing to a greedy solution under interview time pressure; stating this verification step out loud is itself a strong signal.
- **No recovery mechanism**: unlike backtracking (Phase 3), a greedy algorithm has no way to reconsider an earlier choice — if the greedy-choice property does not actually hold for a given problem, there is no fix *within* the greedy framework; the correct response is recognizing this and switching to DP, not patching the greedy loop.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Sequential Feasibility:** Jump Game → Jump Game II → Gas Station

**Sorting-Based:** Partition Labels

**Revisit With This Lens:** Non-overlapping Intervals, Meeting Rooms II (Topic 14) → Task Scheduler (Topic 9) — all genuinely greedy algorithms, now explicitly through the exchange-argument lens instead of just "the code that worked"

**Explicitly Study as a Counter-Example:** 0/1 Knapsack (Topic 16) — attempt the greedy value-per-weight approach, construct an input where it fails, and confirm DP is required

---

# Part H — References

**Primary text:** CLRS, **Chapter 16, "Greedy Algorithms," in full** — the clearest chapter in the book for this topic's exact pedagogical purpose. §16.1 (activity selection, the worked exchange-argument proof, shared with Topic 14). §16.2, **"Elements of the Greedy Strategy"** — the formal definitions of the greedy-choice property and optimal substructure, and the explicit fractional-vs-0/1-knapsack contrast reproduced in Part B, directly from the source. §16.3 (Huffman codes) — a complete worked advanced example, less common verbatim in interviews but the deepest classical illustration of greedy + exchange-argument proof technique.

**Video references** (search directly by name/series):
- **Abdul Bari** — clear coverage of the fractional knapsack / activity selection greedy proofs, matching CLRS's structure.
- **NeetCode** — the Greedy section of the roadmap; use after an independent attempt.
- **Back To Back SWE** — Jump Game and Gas Station walkthroughs, with the underlying feasibility argument made explicit.

---

# Self-Check / Mastery Criteria

- [ ] Can state the greedy-choice property and optimal substructure, and explain how the greedy-choice property differs from DP's approach
- [ ] Can reproduce the exchange-argument proof technique on at least one problem, not just describe it abstractly
- [ ] Can explain precisely why greedy works for fractional knapsack and fails for 0/1 knapsack
- [ ] Actively searches for a counter-example or exchange argument before trusting a greedy instinct, as a habit, not an afterthought
- [ ] Can name at least two specific, by-name instances where a greedy approach is known to fail (0/1 knapsack, Dijkstra with negative weights)
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 16

Next: **Phase 4, Topic 16 — 2-D Dynamic Programming**.
