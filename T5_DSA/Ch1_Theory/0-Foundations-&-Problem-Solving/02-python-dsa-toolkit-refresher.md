# Python DSA Toolkit — Theory, Patterns & Interview Tricks

**Phase 0 — Setup** | Companion notebook: [`02-python-dsa-toolkit-refresher.ipynb`](./02-python-dsa-toolkit-refresher.ipynb) (measured benchmarks: `deque` vs. `list` front-ops, 28x)

---

## Complexity table (memorize cold)

| Structure | Index | Search | Insert/Delete end | Insert/Delete front |
|---|---|---|---|---|
| `list` | O(1) | O(n) | O(1) amortized | O(n) |
| `dict` / `set` | — | O(1) avg | O(1) avg | O(1) avg |
| `deque` | O(1) ends, O(n) middle | O(n) | O(1) | O(1) |
| `heapq` (list) | O(1) peek min | O(n) | O(log n) push/pop | — |

The row that changes the most decisions: `list` is fast at the end, slow at the front; `deque` is fast at **both** ends — the default choice for a BFS queue (Phase 2).

---

## Decision cheat sheet — the actual interview skill

Given a problem, the tool should be near-instant to pick:

| Need | Reach for | Why |
|---|---|---|
| "Have I seen this value before?" | `set` | O(1) average membership |
| Frequency counting | `dict` / `Counter` | O(1) average increment |
| Fast add/remove at BOTH ends (BFS, sliding window) | `deque` | O(1) both ends, `list` is O(n) at the front |
| Repeatedly need current min/max, with updates | `heapq` | O(log n) push/pop vs. O(n log n) re-sorting |
| Search/insertion point in a SORTED sequence | `bisect` | O(log n) vs. O(n) linear scan |
| All subsets/orderings, as a brute-force baseline | `itertools` | correct in one line, no hand-rolled recursion needed to sanity-check |
| Group values by a key | `defaultdict(list)` | no manual "if key not in dict" boilerplate |
| A quick memoization cache | `functools.lru_cache` or a plain `dict` | turns exponential recursion into linear |

---

## Interview Trick: heapq only gives you a min-heap

Python's `heapq` has no max-heap variant. The standard workaround — negate values going in, negate again coming out — is worth having completely automatic, since "find the k largest" comes up constantly and the natural instinct (push raw values) silently gives the k *smallest* instead:

```python
max_heap = []
for x in values:
    heapq.heappush(max_heap, -x)     # store negated
largest = -max_heap[0]                # negate back on read
```

For a fixed k, maintaining a min-heap of size k (popping whenever it exceeds k) is the standard O(n log k) "top-k" pattern — better than sorting everything (O(n log n)) when k is small relative to n.

## Interview Trick: bisect turns a linear scan into a binary search almost for free

Any time a problem involves a **sorted** array and a "find the position/count/nearest value" question, `bisect_left`/`bisect_right` are usually a one-line O(log n) replacement for a manual O(n) scan — and this applies even when binary search is not the *obvious* framing of the problem (e.g. "how many elements are ≤ x" is `bisect_right(arr, x)`, not a loop).

## Interview Trick: sorting with a custom key, correctly, the first time

`sorted(items, key=lambda x: (...))` with a **tuple** key sorts by multiple criteria in priority order — the most common way this gets asked wrong is forgetting that a tuple key sorts lexicographically (first element decides ties on the second, etc.), and forgetting `reverse=True` applies to the *whole* comparison, not selectively to one field (use a negated numeric key, e.g. `-x`, to reverse just one field while sorting another ascending).

```python
# sort by frequency descending, then alphabetically ascending on ties
sorted(word_counts.items(), key=lambda pair: (-pair[1], pair[0]))
```

## Interview Trick: `itertools` as a brute-force sanity check, not just a shortcut

Per the Big-O notebook's space-time-tradeoff framing, the correct workflow is *always* brute force first. `itertools.combinations`/`permutations`/`product` make that brute force a one-liner instead of hand-rolled recursion — cheap insurance against a subtly wrong optimized solution, and a real fallback answer if time runs out.

---

## Common Gotchas / Rapid-Fire

- `list.pop(0)` and `list.insert(0, x)` are O(n) — a very common accidental bottleneck when a `list` is used as a queue; use `deque.popleft()`/`appendleft()` instead.
- `sorted()` returns a new list (O(n) extra space); `.sort()` sorts in place — confusing the two is a common source of "why did my original array change" bugs.
- `heapq.heapify()` is O(n), not O(n log n) — building a heap from an existing list is cheaper than pushing elements one at a time.
- A `dict`/`set` requires **hashable** keys — a `list` cannot be a dict key or set member, but a `tuple` of hashable elements can (used constantly as a memoization key for multi-argument recursive functions).
- `Counter(iterable) - Counter(iterable2)` gives a genuinely useful "what's left after removing these counts" result, not just set difference — worth knowing beyond `.most_common()`.

---

## Self-Check

- [ ] Can pick the correct structure from the decision table in under 10 seconds for a new problem
- [ ] Can build a max-heap via negation without re-deriving it each time
- [ ] Can replace a linear "find position in sorted array" scan with `bisect` on sight
- [ ] Can write a multi-criteria sort key (tuple, with selective reversal) correctly on the first attempt
- [ ] Knows why a `list` cannot be a dict key/set member but a `tuple` can

Next: [`03-recursion-fundamentals.md`](./03-recursion-fundamentals.md)
