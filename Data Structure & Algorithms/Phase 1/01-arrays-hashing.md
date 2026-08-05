# Arrays & Hashing — Complete Theory, Patterns & Interview Mastery

**Phase 1 — Foundational Patterns, Topic 1** 

Practice problems: NeetCode's Arrays & Hashing section, after this document, not before [https://neetcode.io/roadmap].

This document assumes nothing beyond Phase 0. Read it start to finish once, then use it as the reference to return to before every Arrays & Hashing problem until the patterns are automatic.

---

# Part A — Arrays, From First Principles

## What an array actually is

An array is a block of **contiguous memory**, divided into equal-sized slots, one per element. Because every slot is the same size and they sit back-to-back, the address of element `i` can be computed directly:

```
address(i) = base_address + i × element_size
```

This single fact is *why* array indexing is O(1) — it is arithmetic, not a search. Contrast this with a linked list, where reaching element `i` means following `i` pointers one at a time, because there is no formula for where element `i` lives in memory.

## Static vs. dynamic arrays

- **Static array** — fixed size, decided at creation. C's `int arr[10]` is the canonical example. Cannot grow; if more space is needed, a new, larger array must be allocated and everything copied over.
- **Dynamic array** — Python's `list`, Java's `ArrayList`, C++'s `std::vector`. Backed by a static array internally, but automatically reallocates to a bigger static array (typically by **doubling** capacity) when full, and copies existing elements across. This is exactly the amortized-O(1) `append()` behavior covered in Phase 0's Big-O notebook — most appends are cheap, occasionally one triggers an O(n) copy, and the average stays O(1) across many calls.

## Operation complexity (memorize, this is Phase 0's table, specialized to arrays)

| Operation | Complexity | Why |
|---|---|---|
| Access by index | O(1) | direct address arithmetic |
| Search (unsorted) | O(n) | no better option without extra structure |
| Search (sorted) | O(log n) | binary search — Phase 1 Topic 5 |
| Insert/delete at the end | O(1) amortized | occasional resize, averaged out |
| Insert/delete at the front or middle | O(n) | every subsequent element must shift |

## Multi-dimensional arrays: memory layout matters

A 2D array (matrix) is still, underneath, laid out as one contiguous 1D block. **Row-major order** (used by Python, C, C++) stores an entire row contiguously before moving to the next row; **column-major order** (Fortran, MATLAB) does the opposite. This matters for two reasons worth knowing even outside interviews: iterating row-by-row is cache-friendlier than column-by-column in a row-major layout (relevant if performance ever comes up), and it is *why* `matrix[i][j]` addresses as `base + (i × num_cols + j) × element_size` — the same address-arithmetic idea as 1D arrays, one level deeper. This becomes directly relevant again in Phase 4's 2D dynamic programming.

## Core array techniques used constantly across the whole syllabus

- **Prefix sum** — precompute `prefix[i] = arr[0] + arr[1] + ... + arr[i-1]` once, in O(n). Afterward, the sum of any subarray `arr[i:j]` is `prefix[j] - prefix[i]`, in O(1) — turning O(n) per-query range-sum questions into O(1) per query, after an O(n) one-time cost. This is the single most reused array technique in the whole syllabus (subarray-sum problems, Sliding Window's fixed-size variant, 2D range-sum problems in Phase 4).
- **Suffix sum / suffix product** — the mirror of prefix sum, built right-to-left. Combining a prefix pass and a suffix pass (two O(n) passes) solves problems like "product of array except self" without division and without recomputation.
- **Difference array** — the reverse trick: instead of prefix summing values, record the *change* at each index (`diff[l] += v`, `diff[r+1] -= v`), then prefix-sum the diff array once at the end to recover the final values. Useful for "apply many range updates, then read final state once" problems (a Phase 1 Intervals-adjacent technique, worth knowing here since it is built on prefix sums).
- **In-place marking** — when extra space is disallowed, the input array itself can sometimes double as a hash set, by negating values at visited indices or using the sign bit as a "seen" flag (only valid when values map cleanly to valid indices, e.g. `1..n` range problems like "find all missing numbers in `1..n`"). This is the array-specific version of the space-time tradeoff from Phase 0.

**CLRS reference:** *Introduction to Algorithms* (Cormen, Leiserson, Rivest, Stein) treats arrays as a known prerequisite rather than devoting an early chapter to them — its explicit array-adjacent coverage starts at **Chapter 10, "Elementary Data Structures"** (§10.1 covers implementing stacks and queues on top of arrays). The array fundamentals above are standard CS-curriculum material predating CLRS's own starting point, included here because "obvious once known" is exactly the kind of gap that shows up as hesitation in an interview.

---

# Part B — Hashing, From First Principles

This is the theory-heavy half of this document, and the more interview-relevant half — "explain how a hash table works" and "what happens on a collision" are genuinely, frequently asked as their own standalone interview questions, not just assumed background.

## The problem hashing solves

Arrays give O(1) access **by index**, but O(n) search **by value**. Balanced binary search trees (Phase 2) improve search to O(log n). A hash table's goal is more aggressive: O(1) **average-case** search, insert, and delete, by value — achieved by converting a value directly into an array index via a **hash function**, instead of searching for it.

## Direct addressing — the idea in its simplest form

If keys are small integers in a known range `0..m-1`, you could just use an array of size `m` and store each key at `array[key]` directly — O(1) access, no hash function needed at all. This is **direct addressing** (CLRS §11.1), and it is the conceptual starting point hashing generalizes: direct addressing breaks down the moment keys are not small dense integers (strings, large sparse integers, arbitrary objects) — a hash function is what bridges an arbitrary key space down to a small array index.

## The hash function

A hash function `h(key)` maps an arbitrary key to an index in `0..m-1` (`m` = table size). A good hash function needs:
- **Determinism** — the same key always produces the same index (otherwise lookups would never find what insert stored).
- **Uniform distribution** — keys should spread evenly across all `m` slots; a hash function that clusters many keys into few slots defeats the entire point.
- **Speed** — computing the hash itself must be cheap, or the O(1) claim is hollow.

Two classical hash function families (CLRS §11.3): the **division method** (`h(key) = key mod m`, simple, but sensitive to poor choices of `m` — a power of 2 for `m` interacts badly with keys that share low-order bit patterns) and the **multiplication method** (multiply the key by a constant in `(0,1)`, take the fractional part, scale to the table size — less sensitive to the choice of `m`). Production hash tables (including Python's) use more sophisticated hash functions in practice, but these two classical methods are what "explain a hash function" answers are expected to reference.

## Collisions are inevitable — the pigeonhole principle guarantees it

With more possible keys than table slots (almost always true), **some two keys must hash to the same slot** eventually — this is not a flaw to be engineered away, it is mathematically unavoidable, and every real hash table design is really a design for *handling* collisions well, not preventing them. Two standard strategies:

### Chaining (CLRS §11.2)

Each array slot holds a **linked list** (or similar structure) of all keys that hashed there. Insert: hash the key, prepend to that slot's list — O(1). Search: hash the key, then linearly scan that slot's list — O(1) average if collisions are rare, degrading toward O(n) in the worst case if many keys collide into the same slot (e.g. every key hashing to the same slot turns the whole table into one linked list).

### Open addressing (CLRS §11.4)

No separate lists — every key lives directly in the array itself. On a collision, **probe** for the next open slot using a defined sequence: **linear probing** (try `slot+1`, `slot+2`, ...), **quadratic probing** (try `slot+1²`, `slot+2²`, ...., reduces the "clustering" that plain linear probing suffers from), or **double hashing** (use a second hash function to determine the probe step size, spreading collisions out more evenly than either of the above). Open addressing avoids the extra memory of linked lists but requires care on deletion (naively marking a slot "empty" can break the probe chain for keys that were inserted after it, since search relies on probing until an EMPTY slot is hit — a deleted-but-not-specially-marked slot can wrongly terminate a search early).

## Load factor and resizing

**Load factor** `α = n/m` (number of stored keys ÷ table size) governs performance directly: the higher `α`, the more collisions, the further average-case performance drifts from O(1) toward O(n). Real hash tables monitor `α` and **rehash** — allocate a bigger table (commonly doubling, mirroring dynamic array growth) and reinsert every existing key — once `α` crosses a threshold (commonly around 0.7). This rehash is O(n) when it happens, but amortized across all the operations between rehashes, it contributes only O(1) amortized per operation — the exact same amortized-analysis idea from Phase 0, applied to a different structure.

## Worst case is real, not just theoretical — and how it is mitigated

Every hash table operation described as "O(1) average" is genuinely **O(n) worst case** — if every key happens to collide into the same slot (whether by bad luck or by a deliberately adversarial input designed to exploit a known hash function), performance degrades to a linked-list scan. This is a real, documented attack class ("hash-flooding" denial-of-service attacks against naive hash table implementations), which is *why* Python randomizes the hash of strings by default (`PYTHONHASHSEED`, randomized per-process unless explicitly fixed) — specifically to prevent an attacker from crafting inputs guaranteed to collide. Worth having ready as a specific, concrete fact if a "what could go wrong with a hash table" question comes up.

## How Python's `dict` and `set` actually work

CPython's `dict` (and `set`) use **open addressing** internally (not chaining), with a probing sequence that mixes bits of the hash to reduce clustering. Since Python 3.6, `dict` additionally maintains **insertion order** as a language guarantee — implemented via a separate compact array of the actual key-value entries, with the open-addressed table storing only indices into that compact array, not the entries themselves. Two contract rules worth stating precisely:
- **Hashable requirement**: an object used as a dict key or set member must implement `__hash__` and `__eq__` consistently — two objects that compare equal (`==`) **must** have equal hashes, or dict/set lookups silently break (two equal objects could end up treated as different keys).
- **Mutability rule**: a key's hash must never change while it is in use as a key — which is exactly why `list` (mutable) is unhashable, while `tuple` (immutable, provided its contents are also hashable) is. This is not an arbitrary restriction; it is a direct consequence of how the underlying table locates a key by its hash.

**CLRS reference:** Chapter 11, "Hash Tables" — §11.1 Direct-address tables, §11.2 Hash tables (chaining), §11.3 Hash functions, §11.4 Open addressing, §11.5 Perfect hashing (a guarantee of zero collisions when the full key set is known in advance — a nice fact to know exists, rarely asked in depth in interviews).

---

# Part C — The Core Patterns

Everything in this topic reduces to a small number of shapes. Recognizing which shape a new problem is, is the actual skill being tested — not memorizing individual problems.

### 1. Frequency Counting

Count occurrences of each element with a `dict`/`Counter` in one O(n) pass. The foundation almost every other pattern below builds on.
```
count = {}
for x in arr:
    count[x] = count.get(x, 0) + 1
```
**Signals:** "most frequent", "occurs more than once", "majority element", "anagram" (frequency of characters).

### 2. Complement / Pair Lookup ("have I seen what I need?")

For each element, check whether the value that would COMPLETE the answer has already been seen, using a hash set/map built incrementally in a single pass.
```
seen = {}
for i, x in enumerate(arr):
    needed = target - x            # or whatever "completes" the answer
    if needed in seen:
        return (seen[needed], i)
    seen[x] = i
```
**Signals:** "two numbers that sum to...", "pair with difference...", any "find X and Y such that X op Y == target" phrasing.

### 3. Prefix Sum + Hashmap (subarray sum problems)

Combine Part A's prefix sum with Part C.2's complement lookup: track prefix sums seen so far in a hashmap (value → count or value → earliest index); for each new prefix sum, check whether `current_prefix - target` has been seen before — if so, a subarray ending here sums to `target`.
```
prefix_sum = 0
seen = {0: 1}                       # empty prefix (sum 0) has occurred once, by default
count = 0
for x in arr:
    prefix_sum += x
    count += seen.get(prefix_sum - target, 0)
    seen[prefix_sum] = seen.get(prefix_sum, 0) + 1
```
**Signals:** "subarray sum equals k", "continuous subarray", "number of subarrays with property X".

### 4. Canonical Key Grouping

Transform each item into a canonical form that is IDENTICAL for all items that should be grouped together, and use that canonical form as a dict key.
```
groups = defaultdict(list)
for item in items:
    key = canonicalize(item)        # e.g. sorted characters, or a count-signature tuple
    groups[key].append(item)
```
**Signals:** "group anagrams", "group by some derived property", "find items that are equivalent under some transformation".

### 5. Set for Existence / Uniqueness / Deduplication

The simplest pattern, and the most under-used relative to how often it applies: any "have I seen this / is this a duplicate / what's unique" question reduces to a `set`, O(n) instead of an O(n²) or O(n log n) alternative.
**Signals:** "contains duplicate", "distinct elements", "longest sequence of consecutive [unique] elements".

### 6. Bounded-Range Counting Array (when a dict is overkill)

When values are known to fall in a small, fixed range (e.g. lowercase letters, digits 0–9, values `1..n`), a plain **array** used as a counter is faster in practice than a dict (no hashing overhead, better cache locality) and is worth defaulting to over a dict specifically in that situation — this is the same idea behind Top K Frequent Elements' bucket sort (Phase 0/Phase 1 crossover) and behind counting sort generally.

### 7. Two-Pass Prefix/Suffix Arrays

Build a prefix-aggregate array left-to-right and a suffix-aggregate array right-to-left (Part A), then combine them — solves "for each position, something about everything except this position" problems in O(n) with O(1) extra work per combination step, without ever recomputing from scratch.
**Signals:** "except itself", "excluding the current element", "product/sum of all others".

---

# Part D — Optimization Principles, Applied to This Topic

- **The default upgrade path**: nested-loop brute force (O(n²), checking every pair) → single hashmap pass (O(n)) is the single most common "can you do better" answer in this entire topic. State it explicitly: "trading O(n) space for a hash map to bring the time down from O(n²) to O(n)."
- **Sorting as a alternate space-time tradeoff**: several array/hashing problems (Contains Duplicate, for instance) also admit an O(n log n) time / O(1) *extra* space solution via sorting first, as an alternative to the O(n) time / O(n) space hashmap solution — genuinely worth offering as a second option when an interviewer probes on space constraints, not just a fallback.
- **Prefer an array-as-counter over a dict** specifically when the value range is small and known — smaller constant factor, same asymptotic complexity, and it signals awareness of the tradeoff rather than reflexively reaching for a dict every time.
- **One pass beats two passes, when achievable** — many problems that look like they need a "build a structure, then query it" two-pass shape can be collapsed into one pass by checking-before-inserting (Part C.2's ordering) instead of inserting-everything-then-checking. Not always possible, but worth actively looking for.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

Signal words/phrases and what they usually mean, roughly in order of how strongly they point here:

| Phrase in the problem | Likely pattern |
|---|---|
| "two numbers/elements that..." | Complement/Pair Lookup |
| "duplicate", "unique", "distinct" | Set |
| "anagram", "group by" | Canonical Key Grouping |
| "frequency", "most/least common", "top k" | Frequency Counting (+ heap/bucket sort for the top-k step) |
| "subarray sum", "continuous subarray" | Prefix Sum + Hashmap |
| "first missing", "all numbers from 1 to n" | Bounded-range array trick, or in-place marking |
| "except itself", "excluding current element" | Two-pass prefix/suffix |
| "longest consecutive sequence" | Set, with a trick: only start counting from a number whose `n-1` is NOT in the set (avoids re-scanning the same run repeatedly) |

If none of these signals are present, this is a strong hint the problem belongs to a *different* Phase 1 topic (Two Pointers, Sliding Window) rather than Arrays & Hashing specifically — worth explicitly ruling this topic out rather than forcing it.

---

# Part F — Common Gotchas & Interview Traps

- **A value pairing with itself**: "two numbers that sum to target" needs two *distinct indices*, not two distinct *values* — `nums=[3,3], target=6` is valid (indices 0 and 1), but a naive implementation that checks `x == target - x` without also checking index distinctness can incorrectly accept or reject this case.
- **Unhashable keys**: grouping by a `list` or by multiple loose values needs conversion to a `tuple`/`frozenset`/sorted-string first — a `list` cannot be a dict key (Part B's mutability rule).
- **Off-by-one in prefix sums**: deciding whether `prefix[i]` means "sum of the first `i` elements" or "sum up to and including index `i`" needs to be fixed and consistent — this single ambiguity is the most common source of subtle bugs in this entire pattern family. Seeding `seen = {0: 1}` for the "empty prefix" case (Part C.3) is easy to forget and causes an off-by-one undercount.
- **"First occurrence" vs "count" vs "all pairs"**: read the problem carefully for whether it wants the first valid answer (return immediately), a count of all valid answers (accumulate), or every valid answer explicitly listed (collect into a result structure) — the same underlying pattern, three different loop-ending conditions.
- **Hash collision worst case is a legitimate answer to "when would this be slow"** — do not claim hash table operations are unconditionally O(1); "O(1) average, O(n) worst case, and here is why" is the complete, correct statement.
- **Negative numbers and zero** can break naive tricks that assume all values are positive (e.g. using sign-flipping for in-place marking, or assuming index-as-value tricks) — check the constraints for the actual allowed value range before reaching for these.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

Work through these on NeetCode/LeetCode directly, using the patterns above as the lens, in roughly this order:

**Frequency Counting / Set:** Contains Duplicate → Valid Anagram → Design HashSet/HashMap (build one from scratch once, for the theory to stop being abstract)

**Complement/Pair Lookup:** Two Sum → Two Sum II (sorted input — compare against a two-pointer solution once Topic 2 is done) → 4Sum II

**Canonical Key Grouping:** Group Anagrams

**Prefix Sum + Hashmap:** Subarray Sum Equals K → Continuous Subarray Sum → Contiguous Array

**Two-Pass Prefix/Suffix:** Product of Array Except Self

**Bounded-Range / In-Place Trick:** Find All Numbers Disappeared in an Array → First Missing Positive (harder — combines several ideas above)

**Set with a Twist:** Longest Consecutive Sequence (the "only start from a true sequence start" trick, Part E)

**Top-K / Bucket Sort Crossover (Phase 0 preview):** Top K Frequent Elements

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS), 3rd or 4th edition. Chapter 10 (§10.1) for array-backed structures, **Chapter 11 in full** for hash tables — this chapter alone is worth reading end to end once, not skimmed; it is the standard reference every "explain how a hash table works" interview question ultimately traces back to.

**Video references** (search these directly — channel/series names given rather than links, since exact URLs are not something to guess at):
- **Abdul Bari** — "Hashing" playlist on YouTube; clear, whiteboard-style walkthroughs of hash functions, chaining, and open addressing that map directly onto CLRS Chapter 11.
- **MIT OpenCourseWare, 6.006 Introduction to Algorithms** — the hashing lecture(s) from this freely-available MIT course go deeper into hash function theory (universal hashing) than most interview prep needs, but are the rigorous version if the "why" behind the practical rules is wanted.
- **NeetCode** — the Arrays & Hashing section of the NeetCode roadmap (primary roadmap reference) pairs each problem with a short explained walkthrough; use these *after* attempting a problem independently, per the DSA Problem-Solving Framework, not before.
- **William Fiset's algorithms playlist** — has a solid hash table implementation walkthrough (including open addressing probing sequences in detail) for anyone who wants to see the data structure actually built, not just described.

---

# Self-Check / Mastery Criteria

- [ ] Can explain why array indexing is O(1) in terms of address arithmetic, not just state that it is
- [ ] Can explain chaining vs. open addressing, and state a real tradeoff between them
- [ ] Can state load factor, and explain what happens as it grows, and why rehashing is amortized O(1)
- [ ] Can explain why Python `dict`/`set` require hashable, immutable-while-in-use keys, from the mechanism, not as a memorized rule
- [ ] Can identify which of the 7 core patterns (Part C) a new, unseen problem statement maps to, within about a minute
- [ ] Can state the worst-case complexity of a hash table operation honestly (O(n)), not just quote the average case
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 2

Next: **Phase 1, Topic 2 — Two Pointers**.
