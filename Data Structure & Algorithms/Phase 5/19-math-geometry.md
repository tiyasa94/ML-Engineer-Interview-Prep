# Advanced Trees, Bit Manipulation & Math/Geometry — Complete Theory, Patterns & Interview Mastery

**Phase 5 — Advanced Trees, Bit Manipulation & Math/Geometry, Topic 19** |  Practice problems: NeetCode's Advanced Graphs / Bit Manipulation / Math & Geometry sections, after this document, not before.

---

# Topic 19 — Math & Geometry

## Part A — What This Topic Actually Is

A grab-bag topic by nature — unlike Topics 17-18, there's no single unifying data structure or trick family. What unifies it is that these problems require recognizing a **specific mathematical fact** (a formula, an identity, a geometric property) that turns an otherwise-brute-force problem into a direct computation. The interview skill here is less "apply a general framework" and more "recognize which named piece of math this problem is secretly asking for."

## Part B — The Core Facts

### Modular arithmetic

Used whenever a problem asks for an answer "modulo `10^9 + 7`" (a conventional large prime, chosen specifically to avoid overflow while staying prime, which matters for some operations like modular inverse) — a strong signal that the true answer would be astronomically large (common in combinatorics/DP-counting problems) and only the remainder is wanted.

Key identities, useful to have memorized:
```
(a + b) mod m = ((a mod m) + (b mod m)) mod m
(a * b) mod m = ((a mod m) * (b mod m)) mod m
```
Critically, **modular division is not simply `(a / b) mod m`** — division requires the **modular multiplicative inverse** of `b` (which exists when `gcd(b, m) = 1`, guaranteed when `m` is prime and `b` is not a multiple of it), typically computed via Fermat's Little Theorem: `b^(m-2) mod m` (when `m` is prime), using fast exponentiation. Worth knowing this exists even if the specific problem doesn't require implementing it — it's a common "wait, can I just divide?" trap.

### GCD / LCM

- **Euclidean algorithm**: `gcd(a, b) = gcd(b, a mod b)`, base case `gcd(a, 0) = a` — `O(log(min(a,b)))`, exponentially faster than naive factor-checking.
- `lcm(a, b) = (a * b) / gcd(a, b)` — direct consequence of `gcd(a,b) * lcm(a,b) = a * b`.
- **Signals:** "simplify a fraction", "find the largest X that divides both", "repeating pattern/cycle length" (LCM of individual cycle lengths gives the combined cycle length — a recurring, less obvious LCM application).

### Prime numbers

- **Sieve of Eratosthenes**: find all primes up to `n` in `O(n log log n)` by iteratively marking multiples of each prime starting from 2 — the standard tool whenever a problem needs primality for *many* numbers up to some bound, as opposed to checking a single number (for which trial division up to `√n` is simpler and sufficient).
- Trial division up to `√n` for a single primality check: if `n` has a factor greater than `√n`, it must also have a corresponding factor less than `√n`, so checking only up to `√n` is sufficient — a small but frequently-tested piece of reasoning worth being able to justify, not just apply.

### Fast exponentiation (exponentiation by squaring)

Computing `x^n` in `O(log n)` instead of `O(n)`, using the identity `x^n = (x^(n/2))^2` (adjusting for odd `n` by pulling out one extra factor of `x`) — the same halving-the-problem idea as binary search (Topic 5), applied to exponentiation instead of search, and the building block for the modular-inverse computation above.

### Matrix rotation / transposition

Rotating an `n x n` matrix 90 degrees in place is standard **transpose, then reverse each row** (for clockwise; reverse each row then transpose, or transpose then reverse columns, for counterclockwise — worth being precise about which combination gives which direction rather than guessing under pressure). Achieves `O(1)` extra space instead of allocating a new matrix.

### Point/line geometry basics

- **Distance** between two points: standard Euclidean distance formula; when only *comparing* distances (not needing the actual value), skip the square root and compare squared distances directly — a small but real optimization worth mentioning.
- **Collinearity of three points**: check whether the slope between the first pair equals the slope between the second pair — done via **cross product** (`(y2-y1)*(x3-x1) == (y3-y1)*(x2-x1)`) rather than direct slope division, specifically to avoid division-by-zero on vertical lines.
- **Orientation of a turn** (clockwise / counterclockwise / straight) via the sign of the same cross-product expression — the foundational primitive behind convex hull algorithms (worth naming as the "next topic" in geometry if asked, though full convex hull is typically out of scope at this level).

## Part C — Optimization Principles

- **Sieve of Eratosthenes' headline win**: `O(n√n)` (checking each of `n` numbers individually via trial division) -> `O(n log log n)` (sieve) — the payoff specifically when many primality checks are needed against a shared bound, mirroring Topic 17's "many queries -> precompute a structure" theme, just applied to number theory instead of array queries.
- **Fast exponentiation's headline win**: `O(n)` (repeated multiplication) -> `O(log n)` — directly enables modular-inverse computation, which itself is the difference between a modular-arithmetic combinatorics problem being solvable in time or not.
- **In-place matrix rotation**: `O(n²)` extra space (new matrix) -> `O(1)` extra space via transpose+reverse — the standard "can you do it in-place" interview follow-up for this specific problem.

## Part D — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely fact needed |
|---|---|
| "answer modulo 10^9 + 7" | Modular arithmetic identities (and possibly modular inverse if division is involved) |
| "greatest common divisor", "simplify fraction", "cycle length" | GCD/LCM, Euclidean algorithm |
| "is X prime", "primes up to n" | Trial division (single check) vs. Sieve (many checks) |
| "compute x^n efficiently" | Fast exponentiation |
| "rotate the matrix/image in place" | Transpose + reverse |
| "three points on a line", "clockwise/counterclockwise" | Cross product |

## Part E — Common Gotchas & Interview Traps

- **Attempting modular division as plain division** — silently produces a wrong answer rather than crashing, which makes it a particularly dangerous bug; recognize the "modulo a prime" signal as the specific trigger to check whether division appears anywhere in the recurrence.
- **Using trial division inside a loop that needs many primality checks** — technically correct but too slow; this is exactly when the sieve's precomputation cost is worth paying (same "amortize setup over many queries" logic as Topic 17).
- **Floating-point precision in geometry problems** — comparing distances or slopes with floating-point division/square roots introduces precision error; prefer integer arithmetic (squared distances, cross products) whenever the inputs are integers and it's feasible.
- **Integer overflow in fixed-width-integer languages** — `a * b` before taking `mod m` can overflow 32-bit (and even 64-bit, for large enough inputs) integers in Java/C++; take the mod at each multiplication step rather than only at the end.
- **Off-by-one/direction errors in matrix rotation** — mixing up clockwise vs. counterclockwise transpose-then-reverse order is a very easy, very common mistake to make quickly and confidently in the wrong direction.

## Part F — Curated Practice List

**Modular arithmetic / combinatorics:** Pow(x, n) → Unique Paths (revisit through a combinatorics/modular lens)

**GCD/LCM:** basic GCD/LCM implementation problems, fraction-simplification problems

**Matrix:** Rotate Image → Spiral Matrix

**Geometry:** Max Points on a Line

---

# References (Topics 17-19)

**Primary text:** CLRS — **Chapter 21, "Data Structures for Disjoint Sets"** (Union-Find, including the formal proof of the inverse-Ackermann bound with both optimizations combined); Segment Tree and BIT are not covered in CLRS by name and are best learned from competitive-programming references (e.g., **CP-Algorithms**, cp-algorithms.com) rather than a textbook chapter. **Chapter 31, "Number-Theoretic Algorithms"** covers GCD (Euclidean algorithm), modular exponentiation, and modular arithmetic foundations rigorously.

**Video references** (search directly by name/series):
- **William Fiset** — a widely-regarded, specifically thorough YouTube series on Union-Find and both Segment Tree and Fenwick/BIT implementations, with visual walkthroughs of path compression and union by rank.
- **Tushar Roy** — clear coverage of Segment Tree and Fenwick Tree construction and query logic.
- **Back To Back SWE** — strong walkthroughs of bit manipulation tricks (XOR-based problems, counting bits) with the reasoning made explicit, not just the final trick.
- **Errichto / CP-Algorithms** — the standard competitive-programming references for number theory (modular inverse, sieve, fast exponentiation) and geometry primitives (cross product, orientation), at a level of rigor beyond typical interview prep but useful for the "why does this work" layer.
- **NeetCode** — the Bit Manipulation and Math & Geometry sections of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can implement Union-Find with **both** path compression and union by rank/size, and state why both together give the inverse-Ackermann bound
- [ ] Can explain when to reach for a plain prefix-sum array vs. Segment Tree vs. BIT, based on whether updates occur and whether the aggregate is invertible
- [ ] Can solve "find the single non-duplicate element" using XOR in O(1) space, and explain why it works from XOR's three core properties
- [ ] Can explain what a bitmask represents in subset-DP, and why it only works when n is small (≤ ~20)
- [ ] Can state why modular division requires a modular inverse, and how Fermat's Little Theorem provides one when the modulus is prime
- [ ] Can explain why trial division only needs to check up to √n for a single primality check, and when a Sieve is worth it instead
- [ ] Can rotate a matrix 90 degrees in place via transpose + reverse, and correctly reason about which direction each combination produces
- [ ] Can use the cross product (not slope/division) to check collinearity or turn direction, and explain why it avoids the vertical-line division-by-zero issue
- [ ] Has independently solved at least one problem from each pattern category in Parts F (across all three topics) before considering Phase 5 complete

Next: **End of core roadmap — proceed to mock interviews and mixed-topic review.**
