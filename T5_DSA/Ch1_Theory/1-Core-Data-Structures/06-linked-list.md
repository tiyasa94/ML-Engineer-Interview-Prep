# Linked List — Complete Theory, Patterns & Interview Mastery

**Phase 2 — Linked Structures & Trees, Topic 6** | Practice problems: NeetCode's Linked List section, after this document, not before.

---

# Part A — What a Linked List Actually Is

A linked list is a sequence of **nodes**, each holding a value and a pointer to the next node (singly linked), or pointers to both the next and previous nodes (doubly linked). Unlike an array (Topic 1), memory is **not contiguous** — each node can live anywhere in memory, connected only by pointers. This single difference drives every tradeoff in this topic:

| Operation | Array | Singly Linked List |
|---|---|---|
| Access by index | O(1) | O(n) — must traverse from the head |
| Insert/delete at a KNOWN node | O(n) (shifting) | O(1) |
| Insert/delete at the front | O(n) | O(1) |
| Search | O(n) | O(n) |
| Extra memory per element | none | one (or two) pointers |

The core tradeoff: a linked list gives up random access entirely, in exchange for O(1) insertion/deletion anywhere a reference is already held — the mirror image of the array's tradeoff.

## Singly vs. doubly linked

Singly linked: one `next` pointer per node, O(1) forward traversal only, cannot move backward without restarting from the head. Doubly linked: `next` and `prev` pointers, O(1) traversal in both directions and O(1) removal of a node given only a reference to it (no need to find its predecessor by scanning) — at the cost of double the pointer memory and more bookkeeping on every insert/delete (both directions must stay consistent).

**CLRS reference:** Chapter 10, §10.2, "Linked Lists" — direct, dedicated coverage including sentinel/dummy-node variants, worth reading directly for the formal insert/delete pseudocode.

---

# Part B — Key Theoretical Tools

## The dummy (sentinel) node technique

A large fraction of linked-list bugs come from special-casing the head of the list (deleting/inserting there needs different code from every other position, since there's no "previous" node to update). A **dummy node** — an extra node placed before the real head, whose `next` points to the actual head — eliminates this special case entirely: every real node, including the first, now has a "previous" node to update through. CLRS calls this a sentinel; it is a standing recommendation, not just a trick for one problem.

## Why iterative reversal is a 3-pointer rewiring, and why order matters

Reversing a list means flipping every `next` pointer. The danger: overwriting a node's `next` pointer **before** its original destination has been saved elsewhere loses the rest of the list permanently. The correct order requires three pointers in play simultaneously:
```
prev, curr = None, head
while curr:
    next_node = curr.next        # SAVE the rest of the list before it's lost
    curr.next = prev              # now safe to rewire
    prev = curr
    curr = next_node
return prev                        # prev ends on the new head
```
The invariant, provable by induction: after each iteration, `prev` is the head of the correctly-reversed portion processed so far, and `curr` is the unprocessed remainder — exactly the loop-invariant style proof from Binary Search (Topic 5), applied here.

## Recursive traversal: correctness and the space cost

A recursive reversal or traversal is correct by the same base-case/recursive-case reasoning as Phase 0's Recursion document, but costs O(n) **space** from the call stack — worth stating explicitly as a real tradeoff against the O(1)-space iterative version, not just a stylistic choice.

## Fast/slow pointers — this topic's biggest crossover

Middle-finding and cycle detection on linked lists are **Topic 2's Two Pointers Pattern 4, unchanged** — if that correctness proof (the meeting-point derivation) isn't solid, revisit it now rather than re-deriving it differently here.

---

# Part C — The Core Patterns

### 1. Dummy Node for Insert/Delete

```
dummy = Node(0, next=head)
prev = dummy
while prev.next and should_remove(prev.next):
    prev.next = prev.next.next     # unlink, no special-casing the real head
    # (do not advance prev here if multiple consecutive removals are possible)
return dummy.next                   # the (possibly new) real head
```
**Signals:** any "remove/insert node(s) matching a condition", especially when the head itself might need removing.

### 2. Iterative Reversal (Part B, above) and its variants

Reverse an entire list, or **reverse a sublist** between two positions (same 3-pointer rewiring, bounded to a range, with careful re-attachment to the untouched parts before and after), or **reverse in groups of k** (repeated bounded reversal, k nodes at a time).
**Signals:** "reverse", "reverse between", "reverse in groups of k".

### 3. Fast/Slow Pointers (imported directly from Topic 2)

Middle-finding: slow moves 1 step, fast moves 2 — when fast reaches the end, slow is at the middle. Cycle detection: Floyd's algorithm, full proof already established in Topic 2.
**Signals:** "middle of the list", "cycle", "loop", "palindrome linked list" (find middle, reverse second half, compare).

### 4. Two-Pointer Offset (N-th Node From the End)

Advance one pointer `n` steps ahead first, then move both pointers together — when the lead pointer reaches the end, the trailing pointer is exactly `n` nodes from the end. A single pass, no need to first count the list's length in a separate pass.
```
fast = slow = dummy               # dummy simplifies removing the actual head, if n == length
for _ in range(n):
    fast = fast.next
while fast.next:
    fast = fast.next
    slow = slow.next
# slow.next is the n-th node from the end
```
**Signals:** "n-th node from the end", "remove the k-th last element" — solvable in one pass instead of two.

### 5. Merge-Style Two Pointers (imported directly from Topic 2)

Merging two sorted linked lists is Topic 2's Pattern 5, with pointer rewiring instead of building a new array.
**Signals:** "merge two sorted lists", "merge k sorted lists" (this extends to a heap-based approach — Topic 9).

### 6. Combining a Hash Map with a Linked List

Some problems need O(1) access to an arbitrary node BY VALUE (which a linked list alone cannot give — Part A) combined with O(1) reordering (which arrays cannot give). The fix: a hash map from value/key to node reference, alongside a doubly linked list — the map provides O(1) lookup, the list provides O(1) reordering once a node is found. This exact combination is the classic LRU Cache design (Part G).

---

# Part D — Optimization Principles

- **O(n) space (build a new list / dump into an array) vs. O(1) space (rewire in place)**: nearly every linked-list problem admits both, and defaulting to the O(1) in-place version, when achievable, is the expected "optimized" answer — worth stating both and explaining the tradeoff.
- **One pass vs. two passes**: the N-th-from-end offset trick (Pattern 4) is the canonical example of collapsing "first count the length, then traverse again" into a single pass — the same "avoid redundant scanning" theme recurring since Topic 1.
- **Iterative vs. recursive**: recursive solutions are often shorter to write but cost O(n) stack space; know both, and be able to state the space cost of whichever is offered as the "clean" solution.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "reverse", "reverse between", "reverse in groups" | Reversal (Pattern 2) |
| "middle", "cycle", "loop", "palindrome list" | Fast/Slow Pointers (Pattern 3) |
| "n-th from the end", "k-th last" | Two-Pointer Offset (Pattern 4) |
| "merge sorted lists" | Merge-Style (Pattern 5) |
| "remove node(s) matching..." | Dummy Node (Pattern 1) |
| "O(1) get AND O(1) put/evict, in order" | Hashmap + Doubly Linked List (Pattern 6) |

---

# Part F — Common Gotchas & Interview Traps

- **Losing the rest of the list**: overwriting `curr.next` before saving it elsewhere (Part B) — the single most common reversal bug, and the reason the 3-pointer template exists.
- **Forgetting `None`/null checks**: `curr.next.next` without confirming `curr.next` exists first crashes on the last node — exactly Topic 2's fast/slow pointer guard, reapplied.
- **Dummy node off-by-one**: forgetting to return `dummy.next` (not `dummy` itself, and not the original `head`, which may no longer be the head after removals).
- **Doubly linked list desynchronization**: updating `next` without updating the corresponding `prev` on the neighboring node (or vice versa) leaves the list in an inconsistent state that only manifests as a bug when traversed in the *other* direction.
- **Reversing a sublist**: forgetting to correctly re-attach the reversed segment's new head/tail to the untouched portions before and after it — a very common source of a silently truncated or disconnected list.
- **Mutating a list while iterating over it** without a saved reference to what comes next (a more general form of the first gotcha).

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Reversal:** Reverse Linked List → Reverse Linked List II (sublist) → Reverse Nodes in k-Group

**Fast/Slow:** Middle of the Linked List → Linked List Cycle → Linked List Cycle II → Palindrome Linked List

**Two-Pointer Offset:** Remove Nth Node From End of List

**Merge-Style:** Merge Two Sorted Lists → Merge k Sorted Lists (revisit after Topic 9, Heaps)

**Dummy Node:** Remove Linked List Elements → Remove Duplicates from Sorted List II

**Other essential:** Add Two Numbers (digit-by-digit with carry) → Copy List with Random Pointer (hashmap of old→new node references) → Reorder List (find middle + reverse second half + merge, a strong combined-pattern problem)

**Capstone (combines this entire topic with Topic 1's hashing):** LRU Cache — implement using a hash map + doubly linked list (Pattern 6), from scratch.

---

# Part H — References

**Primary text:** CLRS, **Chapter 10, §10.2, "Linked Lists"** — direct, dedicated coverage, including sentinel-based (dummy node) implementations described formally.

**Video references** (search directly by name/series):
- **mycodeschool** — classically well-regarded, clear visual walkthroughs of linked list operations and reversal specifically.
- **Abdul Bari** — linked list fundamentals and complexity comparisons against arrays.
- **Back To Back SWE** — LRU Cache design walkthrough (Part G's capstone).
- **NeetCode** — the Linked List section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can state the array-vs-linked-list operation complexity table from memory, and explain each row
- [ ] Can implement iterative reversal from scratch without losing the list, explaining why the save-before-overwrite order matters
- [ ] Can explain the dummy node technique and why it eliminates head-special-casing
- [ ] Can implement the n-th-from-end offset trick in one pass
- [ ] Can design LRU Cache's hashmap + doubly-linked-list combination and explain why each piece is necessary
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 7

Next: **Phase 2, Topic 7 — Trees**.
