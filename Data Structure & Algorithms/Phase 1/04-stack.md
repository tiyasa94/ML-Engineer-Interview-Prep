# Stack — Complete Theory, Patterns & Interview Mastery

**Phase 1 — Foundational Patterns, Topic 4** | Practice problems: NeetCode's Stack section, after this document, not before.

---

# Part A — What a Stack Actually Is

A stack is a **LIFO** (Last In, First Out) structure supporting three core operations, all O(1): `push` (add to the top), `pop` (remove and return the top), `peek`/`top` (look at the top without removing it). There is no operation to access anything other than the top element directly — that restriction is the entire point, not a limitation to work around.

## Two implementations, one clear winner in practice

- **Array-based** — a dynamic array (Python `list`), pushing/popping from the **end**. `append()`/`pop()` (no argument) are O(1) amortized, exactly the dynamic-array analysis from Phase 0 and Topic 1's Part A. This is the standard, default choice.
- **Linked-list-based** — push/pop at the head of a singly linked list, O(1) always, no amortization needed since there is no resizing. Rarely the practical choice despite the "always O(1), not just amortized" property, because array-based stacks have better cache locality (contiguous memory, Topic 1's Part A) and lower per-element overhead (no pointer storage) — worth knowing as a genuine tradeoff, not just "arrays are always better."

**A critical, very common implementation mistake**: a Python `list` used as a stack must push/pop from the **end** (`append()`/`pop()`), never the front (`insert(0, x)`/`pop(0)`) — the front operations are O(n) (Phase 0's DSA toolkit document), silently turning an intended O(1) stack operation into an O(n) one.

## The call stack IS a stack

Phase 0's Recursion notebook covered this operationally; worth restating as the conceptual anchor for this entire topic: every recursive function call pushes a frame (return address, local variables) onto the call stack, and returning pops it. **Every recursion-to-iteration conversion works by making that implicit call stack explicit** — replacing the language runtime's stack with a manually managed one. This is not a coincidence or an analogy; it is the same data structure, used two different ways.

---

# Part B — Why Stacks Are the Right Structure: The Theory

## The "most recent unmatched thing" principle

Any problem centered on **properly nested or matched structure** — balanced parentheses, matching HTML/XML tags, undo history, a browser's back button, function call/return — has one property in common: whatever needs to be "closed" or "resolved" next is always the most *recently* opened, still-unresolved thing. That is exactly LIFO order. A stack isn't merely *convenient* for these problems; it is the structure whose access pattern matches the problem's actual structure.

## A genuine theoretical fact worth knowing: this is why finite automata aren't enough

The language of balanced parentheses (some number of `(` followed by exactly that many `)`, at every nesting level) **cannot** be recognized by a finite automaton — it requires unbounded memory to track "how many are currently open," and a finite automaton has, by definition, a *fixed*, finite number of states. Adding a stack to a finite automaton produces a **pushdown automaton**, which *can* recognize exactly this class of languages (the context-free languages, one level up from regular languages in the Chomsky hierarchy). This is why a stack is the textbook-correct structure for parsing nested/matching structure — it is literally the minimal extra memory needed to move from what finite automata can recognize to what nested structures require. (This specific theoretical framing is covered in computation-theory texts, not CLRS — see References below — but is worth having as genuine understanding of *why* stacks and nested structure are so tightly linked, beyond "it happens to work.")

## The Monotonic Stack correctness argument

A **monotonic stack** maintains its elements in strictly increasing or decreasing order at all times, by popping violations before pushing a new element. The canonical use: "next greater element" — for each element, find the nearest element to its right that is larger.

```
stack = []                          # holds INDICES, values strictly decreasing bottom-to-top
answer = [-1] * len(arr)             # default: no next-greater element exists
for i, x in enumerate(arr):
    while stack and arr[stack[-1]] < x:
        j = stack.pop()
        answer[j] = x                # x IS j's next greater element -- found, final, correct
    stack.append(i)
```

**Correctness**: when `arr[stack[-1]] < x`, the element at `stack[-1]` has just found its next-greater element — `x` itself, since `x` is both later in the array and larger. This answer can never change or need revisiting, so popping it and recording the answer immediately is not an approximation, it is final and correct, by exactly the same reasoning as Sliding Window's monotonic deque (Topic 3, Part C.4) — a smaller, earlier value becomes permanently irrelevant once a larger, later value exists, for as long as both remain candidates. **This is the same amortized argument as the monotonic deque, applied to a stack instead of a deque**: every index is pushed once and popped at most once, so total operations are bounded by `2n` — O(n), not the O(n²) a naive "for each element, scan right for the next greater" approach would cost.

---

# Part C — The Core Patterns

### 1. Matching / Validation

Push opening symbols; on a closing symbol, check the stack's top for a match, pop if it matches, otherwise the structure is invalid. Generalizes to any "must close what was most recently opened" problem.
```
stack = []
for c in s:
    if c in OPENERS:
        stack.append(c)
    else:
        if not stack or not matches(stack.pop(), c):
            return False
return not stack                     # nothing left un-closed
```
**Signals:** "valid parentheses", "balanced brackets", "matching tags".

### 2. Expression Evaluation

Postfix (RPN) evaluation: push operands; on an operator, pop two operands, apply the operator, push the result back — O(n), no parsing of precedence needed at all, which is *why* calculators and compilers often convert infix expressions to postfix first (the classical Shunting-Yard algorithm, itself stack-based) before evaluating.
```
stack = []
for token in tokens:
    if is_operator(token):
        b, a = stack.pop(), stack.pop()   # note the order -- b was pushed most recently
        stack.append(apply(token, a, b))
    else:
        stack.append(int(token))
return stack[0]
```
**Signals:** "evaluate expression", "calculator", "Reverse Polish Notation / postfix".

### 3. Monotonic Stack

Part B's pattern, generalized: maintain elements in monotonic order, popping (and finalizing an answer for) violations as new elements arrive.
**Signals:** "next greater/smaller element", "next warmer temperature", "largest rectangle" (a trickier variant: an increasing-height monotonic stack, where popping a bar computes its largest possible rectangle using the *current* index and the new stack top as the left/right boundaries).

### 4. Auxiliary Stack for O(1) Extra Tracking (Min Stack)

Maintain a second stack in lockstep with the main one, each position holding "the minimum seen so far, including this element" — turns `getMin()` into O(1) at the cost of O(n) extra space, the same space-time tradeoff theme from every topic so far.
```
stack, min_stack = [], []
def push(x):
    stack.append(x)
    min_stack.append(min(x, min_stack[-1] if min_stack else x))
def pop():
    min_stack.pop()
    return stack.pop()                 # BOTH stacks pop together -- keeping them in sync is essential
def get_min():
    return min_stack[-1]
```

### 5. Two Stacks for a Different Structure (Queue via Stacks)

A queue (FIFO) can be built from two stacks: push always goes to `stack_in`; pop/peek dequeues from `stack_out`, and only when `stack_out` is empty, everything is transferred from `stack_in` (reversing the order in the process). Each element is moved from `stack_in` to `stack_out` **at most once** across its entire lifetime — the same amortized-analysis shape as everything else in this document: individual operations can occasionally cost O(n) (the transfer), but the *total*, summed across all operations, is O(n), giving O(1) amortized per operation.

---

# Part D — Optimization Principles

- **The core trade this topic makes**: tracking only what is still "relevant" (unmatched opens, or monotonic candidates) instead of repeatedly re-scanning the input — the stack-based instance of the "avoid redundant recomputation" theme recurring since Phase 0.
- **The concrete, quotable win**: a monotonic stack turns "for each element, scan the rest of the array for its next greater element" (O(n²)) into a single O(n) pass — worth stating with the actual before/after complexity when explaining the optimization, not just "it's faster."
- **Space-time tradeoff, explicit version**: Min Stack trades O(n) extra space (the auxiliary stack) for O(1) time on an operation (`getMin`) that would otherwise cost O(n) (scanning the whole stack) or O(log n) (a balanced-BST-backed alternative) — worth naming which resource is being spent to buy which.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "valid", "balanced", "matching brackets/tags" | Matching/Validation (Pattern 1) |
| "evaluate", "calculator", "postfix/prefix/RPN" | Expression Evaluation (Pattern 2) |
| "next greater/smaller", "next warmer", "daily temperatures" | Monotonic Stack (Pattern 3) |
| "largest rectangle", "histogram" | Monotonic Stack, the trickier width-calculation variant |
| "min/max in O(1)" alongside push/pop | Auxiliary Stack (Pattern 4) |
| "implement a queue/stack using..." | Two-Structure simulation (Pattern 5) |
| "undo", "nested", recursive structure needing an iterative version | Explicit stack replacing the call stack (Part A) |

**A specific, valuable cross-reference**: Trapping Rain Water (Topic 2's Two Pointers list) also has a clean **monotonic stack** solution, distinct from the two-pointer one — genuinely worth solving both ways once comfortable with each, since being able to say "here are two different valid approaches, with this tradeoff between them" is a strong interview answer in its own right.

---

# Part F — Common Gotchas & Interview Traps

- **Popping from an empty stack** — always check `if not stack:` before popping, especially in matching problems (a closing bracket with nothing left to match against is exactly the invalid case, not a crash).
- **Front vs. end of a Python list** (Part A) — `pop()`/`append()` only; using `pop(0)`/`insert(0, x)` silently degrades an intended O(1) stack operation to O(n).
- **Monotonic stack direction**: "next greater to the right" (scan left-to-right, pop smaller elements) is a different traversal from "next greater to the left" (scan right-to-left, or reverse the array first) — copying a template without checking which direction the problem actually asks for is a common, easy-to-make error.
- **Sentinel handling in width-based monotonic stack problems** (Largest Rectangle in Histogram): when no smaller element exists to the left or right, the boundary must be treated as index `-1` or `n` respectively — omitting this sentinel handling is the most common source of an off-by-one or out-of-bounds bug in this specific problem.
- **Min Stack desynchronization**: both the main stack and the auxiliary min-tracking stack must push and pop together, every time — a `pop()` that only pops the main stack silently corrupts all future `getMin()` calls.
- **Order of children when converting recursion to iteration** (Part A): since a stack is LIFO, pushing children left-to-right processes them right-to-left when popped — if a specific processing order matters (e.g. matching a recursive DFS's exact visit order), children must be pushed in **reverse** order.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Matching/Validation:** Valid Parentheses → Generate Parentheses (a backtracking problem that reuses this topic's "track unmatched opens" intuition — a good bridge into Phase 3)

**Expression Evaluation:** Evaluate Reverse Polish Notation → Basic Calculator → Decode String (a nested-structure problem — stack of pending multipliers/strings)

**Monotonic Stack:** Daily Temperatures → Next Greater Element I → Next Greater Element II (circular array variant) → Largest Rectangle in Histogram → Car Fleet

**Auxiliary/Two-Structure:** Min Stack → Implement Queue using Stacks → Implement Stack using Queues

**Cross-Reference Practice:** Trapping Rain Water — solve with a monotonic stack, then compare against the Two-Pointer solution from Topic 2

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS). Unlike Two Pointers and Sliding Window, Stack **is** a directly named, dedicated CLRS topic: **§10.1, "Stacks and Queues"** — covers the array-based implementation with `PUSH`/`POP`/`STACK-EMPTY` pseudocode directly, worth reading as the canonical formal treatment. Monotonic stacks specifically are a modern interview-pattern refinement not named in CLRS by that term, but their amortized O(n) guarantee is backed by the same **Chapter 17, Amortized Analysis** referenced in the Sliding Window document.

The pushdown-automaton connection in Part B is **not** CLRS material (CLRS is an algorithms text, not a computation-theory text) — the accurate reference for that specific fact is Sipser, *Introduction to the Theory of Computation* (the chapter on context-free languages and pushdown automata), included here for genuine conceptual depth, not because it will be interview-tested directly.

**Video references** (search these directly by name/series — exact URLs not guessed at):
- **Abdul Bari** — clear coverage of stack/queue fundamentals and classical expression-evaluation algorithms (infix-to-postfix, postfix evaluation).
- **Back To Back SWE** — a specifically well-regarded walkthrough of monotonic stack problems, including Largest Rectangle in Histogram's trickier width/sentinel logic.
- **MIT OpenCourseWare, 6.006** — for the amortized analysis lecture underpinning the monotonic stack and two-stacks-as-a-queue arguments in Part B.
- **NeetCode** — the Stack section of the roadmap; use after an independent attempt, per the Problem-Solving Framework.

---

# Self-Check / Mastery Criteria

- [ ] Can explain why LIFO order is the natural fit for matching/nested-structure problems, not just that it "works"
- [ ] Can state, and reproduce, the correctness argument for a monotonic stack's O(n) amortized guarantee
- [ ] Can implement postfix expression evaluation and explain why postfix needs no precedence handling at evaluation time
- [ ] Knows to always use the end of a Python list for stack operations, and why the front would silently break the complexity
- [ ] Can solve Trapping Rain Water with a monotonic stack, having already solved it with two pointers, and explain the tradeoff between the two approaches
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 5

Next: **Phase 1, Topic 5 — Binary Search**.
