# Advanced Trees, Bit Manipulation & Math/Geometry — Complete Theory, Patterns & Interview Mastery

**Phase 5 — Advanced Trees, Bit Manipulation & Math/Geometry, Topic 18** | Practice problems: NeetCode's Advanced Graphs / Bit Manipulation / Math & Geometry sections, after this document, not before.

---
# Topic 18 — Bit Manipulation

## Part A — What This Topic Actually Is

Bit manipulation problems exploit the fact that integers are stored as fixed-width binary, and that bitwise operators (`&`, `|`, `^`, `~`, `<<`, `>>`) let you inspect and transform that representation directly, often turning an `O(n)`-space or `O(n log n)`-time solution into `O(1)`-space, `O(n)`-time, or better. This topic is less about a single unifying data structure (like Topic 17) and more a **toolbox of specific, memorable tricks**, each one worth recognizing on sight.

## Part B — The Core Tricks

### XOR properties

XOR (`^`) has three properties that generate most of this pattern's problems:
- `x ^ x = 0` (a value XORed with itself cancels out)
- `x ^ 0 = x` (identity)
- XOR is commutative and associative (order doesn't matter)

Combined, this means: **XOR every element in a list together, and every value that appears an even number of times cancels out, leaving only values that appear an odd number of times.** This single fact solves "find the single number that doesn't repeat" in `O(n)` time, `O(1)` space — no hash set needed.

### Bitmasking

Using an integer as a compact set representation: bit `i` of the mask is `1` if element `i` is "in" the set, `0` otherwise. Enables:
- Representing subsets of a small set (commonly `n ≤ ~20`, since `2^20 ≈ 1M` is the usual practical ceiling for brute-force enumeration) as a single integer, letting DP problems index by "which elements have been used so far" — `dp[mask]` — instead of a more expensive explicit set/array (this is exactly Topic 13's DP framework, with the state defined as a bitmask; a direct cross-topic link worth naming explicitly).
- Checking/setting/clearing a specific bit: `x & (1 << i)` to check, `x | (1 << i)` to set, `x & ~(1 << i)` to clear.

**Signals:** "n is small (≤ ~20)", "subset", "traveling salesman-style DP over subsets", "state can be represented as included/excluded".

### Counting set bits

- Brian Kernighan's trick: `x & (x - 1)` clears the **lowest set bit** of `x` (this is the exact same `i & (-i)` family of trick used in BIT, Topic 17 — worth naming the connection). Repeating this and counting iterations until `x` reaches `0` counts set bits in `O(popcount)` time rather than `O(bit-width)`.
- DP formulation: `bits[i] = bits[i >> 1] + (i & 1)` — the count for `i` is the count for `i` with its last bit removed, plus that last bit — builds a full table of counts from `0` to `n` in `O(n)`.

### Other recurring tricks worth knowing on sight

- `x & (x - 1) == 0` (and `x != 0`) checks whether `x` is a power of two — clearing the lowest set bit of a power of two always yields `0`, since a power of two has exactly one set bit.
- `x & 1` checks even/odd, cheaper than `x % 2`.
- `x << 1` / `x >> 1` as multiply/divide by 2 — rarely the point of a problem on its own, but useful to recognize inside a larger bit-manipulation solution.
- Sign bit and two's complement — worth understanding conceptually (why `~x = -x - 1`) even though it rarely drives a whole problem by itself at this level.

## Part C — Optimization Principles

- **The headline win, XOR family**: hash-set-based "find the unique/missing/duplicate element" (`O(n)` time, `O(n)` space) -> XOR-based (`O(n)` time, `O(1)` space) — a space-complexity-class win, not a speed one; worth stating precisely, since interviewers often specifically ask "can you do this without extra space" as the follow-up that bitmask/XOR tricks are the answer to.
- **The headline win, bitmasking DP**: brute-force enumeration of subsets as explicit lists/sets -> subsets represented as integers, enabling `dp[mask]` state — turns an otherwise-unwieldy state representation into an array index, which is what makes subset-DP problems (`n ≤ ~20`) tractable at all within typical time limits.

## Part D — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely trick |
|---|---|
| "every element appears twice except one" | XOR everything together |
| "find the missing number" (in a known range) | XOR range with array, or sum-formula subtraction |
| "n is small (≤ ~20)", "subset", "each item included or excluded" | Bitmasking (often combined with DP) |
| "count the number of 1 bits", "Hamming weight/distance" | Brian Kernighan's trick or the DP bit-count table |
| "power of two" | `x & (x-1) == 0` check |
| "without using extra space", "in O(1) space", involving array/duplicates | Strong signal to look for a bit trick before reaching for a hash set |

## Part E — Common Gotchas & Interview Traps

- **Sign/overflow issues in languages with fixed-width integers** — bit tricks that work cleanly in Python (arbitrary-precision integers) can behave differently in Java/C++ (fixed 32/64-bit, two's complement negative numbers); worth flagging language-specific edge cases explicitly if asked in a language other than Python.
- **Confusing "appears once, others appear twice" (plain XOR) with "appears once, others appear three times"** — the three-times variant needs a different technique (counting bits at each position mod 3), not plain XOR; recognizing which exact variant is being asked matters.
- **Off-by-one in bit indexing** — deciding whether bit 0 is the least-significant or most-significant bit, and being consistent, is a common source of subtle bugs.
- **Reaching for bitmask DP when n is not actually small** — bitmasking is only tractable because `2^n` states must fit in the time limit; if `n` is large (say, `> 20-25`), this pattern doesn't apply and a different approach is needed.

## Part F — Curated Practice List

**XOR family:** Single Number → Single Number II → Missing Number

**Counting bits:** Number of 1 Bits → Counting Bits

**Bitmasking / other:** Sum of Two Integers (using bit operations, no `+`/`-`) → Reverse Bits

---