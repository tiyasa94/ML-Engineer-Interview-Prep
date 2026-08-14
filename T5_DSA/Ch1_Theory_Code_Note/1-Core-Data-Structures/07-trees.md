# Trees — Complete Theory, Patterns & Interview Mastery

**Phase 2 — Linked Structures & Trees, Topic 7** | Practice problems: NeetCode's Trees section, after this document, not before.

---

# Part A — What a Tree Actually Is

A tree is a node-based structure where each node holds a value and pointers to its **children** — a **binary tree** restricts this to at most two children (`left`, `right`). Terminology worth being precise about: **root** (top node, no parent), **leaf** (no children), **height** of a node (longest path down to a leaf, in edges), **depth** of a node (path up to the root), **subtree** (a node plus everything beneath it).

## Binary tree vs. binary search tree (BST) — a distinction worth never blurring

A **binary tree** has no ordering constraint at all. A **binary search tree** adds one specific property: for every node, everything in its **left** subtree is smaller, everything in its **right** subtree is larger. This single property is what makes O(log n) search possible (Part B) — it does *not* hold for a generic binary tree, and assuming BST-style shortcuts on a plain binary tree is a real, common error.

## Shape determines complexity — the single most important fact in this topic

A tree's height `h` governs the complexity of nearly every operation. A **balanced** tree has `h = O(log n)`. A **degenerate/skewed** tree (effectively a linked list — every node has only one child) has `h = O(n)`. The exact same algorithm (BST search, for instance) is O(log n) on a balanced tree and O(n) on a skewed one — this is *why* self-balancing trees (AVL trees, Red-Black trees — CLRS Chapter 13) exist: they guarantee `h = O(log n)` regardless of insertion order, at the cost of extra bookkeeping on every insert/delete. Not required to implement for interviews, but the *reason* they exist is worth being able to state.

---

# Part B — Key Theoretical Tools

## Traversals, and why recursive correctness follows by structural induction

- **Preorder**: node, then left subtree, then right subtree.
- **Inorder**: left subtree, then node, then right subtree.
- **Postorder**: left subtree, then right subtree, then node.
- **Level-order (BFS)**: all nodes at depth 0, then depth 1, then depth 2, ... — uses a queue (Phase 0's `deque`), not the call stack.

Each recursive DFS traversal's correctness follows by **structural induction on the tree**: assume the traversal is correct on any smaller subtree (the inductive hypothesis), and show that correctly traversing the two children and combining with the current node (in whichever order defines pre/in/post) produces a correct traversal of the whole tree. The base case is the empty tree (or a leaf), trivially correct. This is the same inductive reasoning as Phase 0's recursion theory, applied to a branching structure instead of a linear one.

## Why an inorder traversal of a BST yields sorted order — proved, not just observed

By the BST property (Part A), every value in the left subtree is smaller than the node, and every value in the right subtree is larger. By the inductive hypothesis, `inorder(left)` produces the left subtree's values in sorted order, and `inorder(right)` produces the right subtree's values in sorted order. Since every left value < node < every right value, concatenating `inorder(left) + [node] + inorder(right)` is sorted. This is not a coincidence to memorize — it is the direct, provable consequence of combining the BST property with structural induction, and it is exactly why "inorder traversal of a BST" is the standard technique for "kth smallest element" style problems.

## BST operation complexity: O(h), and what that means in practice

Search, insert, and delete on a BST all walk a single root-to-leaf path, comparing and branching left/right — O(h) time. Combined with Part A: O(log n) on a balanced tree, O(n) worst case on a skewed one. Stating "O(log n) average, O(n) worst case, and here is exactly why" (mirroring the honesty required for hash tables, Topic 1) is the complete, correct answer.

---

# Part C — The Core Patterns

### 1. DFS Traversal Templates

```
def preorder(node):
    if not node: return []
    return [node.val] + preorder(node.left) + preorder(node.right)

def inorder(node):
    if not node: return []
    return inorder(node.left) + [node.val] + inorder(node.right)

def postorder(node):
    if not node: return []
    return postorder(node.left) + postorder(node.right) + [node.val]
```
**Signals:** "traverse", "visit every node", or any problem where the visiting *order* is explicitly given.

### 2. BFS / Level-Order Traversal

```
from collections import deque
def level_order(root):
    if not root: return []
    result, queue = [], deque([root])
    while queue:
        level = []
        for _ in range(len(queue)):          # process exactly one full level per outer iteration
            node = queue.popleft()
            level.append(node.val)
            if node.left: queue.append(node.left)
            if node.right: queue.append(node.right)
        result.append(level)
    return result
```
**Signals:** "level order", "level by level", "right side view", "zigzag traversal", "minimum depth" (BFS finds the nearest leaf first, without exploring deeper branches unnecessarily).

### 3. Divide-and-Conquer / Bottom-Up Combination

The most common tree pattern by far: compute something about a node using results already computed for its children.
```
def solve(node):
    if not node: return base_case
    left_result = solve(node.left)
    right_result = solve(node.right)
    return combine(left_result, right_result, node.val)
```
**Signals:** "height", "depth", "balanced", "diameter", "sum", "maximum path" — anything where a node's answer depends on its children's answers.

### 4. Top-Down / Information Passed Downward

The mirror image of Pattern 3: information flows FROM the root DOWN to descendants (a running sum, a running depth, a set of ancestors) rather than being combined upward from children.
```
def solve(node, info_from_above):
    if not node: return
    updated_info = update(info_from_above, node.val)
    solve(node.left, updated_info)
    solve(node.right, updated_info)
```
**Signals:** "root-to-leaf path", "path sum" (accumulating downward), any problem needing to know something about a node's *ancestors*.

### 5. BST-Specific: Exploit the Ordering

Search/insert/delete by comparing against the current node and branching left or right — never both. Validating a BST needs a valid `(lower_bound, upper_bound)` range passed **top-down** (Pattern 4) — checking only `node.left.val < node.val < node.right.val` locally is a very common but **incorrect** shortcut, since it misses violations from deeper descendants against a distant ancestor.

### 6. Lowest Common Ancestor (LCA)

```
def lca(node, p, q):
    if not node or node == p or node == q:
        return node
    left = lca(node.left, p, q)
    right = lca(node.right, p, q)
    if left and right:                  # p and q found in DIFFERENT subtrees -> this node IS the LCA
        return node
    return left or right                 # both in the same subtree -> propagate that answer up
```
**Signals:** "lowest common ancestor", "common ancestor".

### 7. Tree Construction from Traversals

Given two traversal orders (commonly preorder + inorder), the tree can be uniquely reconstructed: preorder's first element is always the root; that root's position in the inorder sequence splits it into the left and right subtrees' inorder sequences, recursively.
**Signals:** "construct/build a tree from [traversal types]".

---

# Part D — Optimization Principles

- **O(n) time is the norm, not something to optimize away** — any traversal that must look at every node is O(n) by necessity; the actual optimization question in this topic is almost always about **space** (recursive call-stack depth, Part B) or about **exploiting BST ordering to avoid visiting the whole tree** (O(h) instead of O(n) — a real, meaningful win, not just a constant-factor one).
- **Bottom-up (Pattern 3) vs. top-down (Pattern 4) is a design decision, not a style preference**: choosing the wrong direction for a given problem either fails to compile the needed information at all, or requires much more convoluted code to route it correctly — worth explicitly identifying which direction a new problem needs *before* writing any recursion.
- **The "different return value than the final answer" trick**: for problems like Diameter of Binary Tree, the value returned UP the recursion (height, for use by the parent's calculation) is *not* the same as the final answer being computed (the diameter, tracked separately, often via a mutable closure variable or an instance attribute) — recognizing that these can be two different things is what unlocks several otherwise-confusing "combination" problems.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "binary tree", generic traversal | DFS Traversal (Pattern 1) |
| "level order", "level by level", "zigzag", "right side view" | BFS (Pattern 2) |
| "height", "depth", "balanced", "diameter", "sum of..." | Bottom-Up Combination (Pattern 3) |
| "root-to-leaf path", "path sum", needs ancestor info | Top-Down (Pattern 4) |
| "binary SEARCH tree" explicitly | Exploit ordering (Pattern 5) — O(h), not O(n) |
| "lowest common ancestor" | LCA (Pattern 6) |
| "construct/build tree from traversals" | Construction (Pattern 7) |
| "kth smallest/largest in a BST" | Inorder traversal (Part B's proof, directly applied) |

---

# Part F — Common Gotchas & Interview Traps

- **Treating a generic binary tree as if it were a BST** — O(log n) shortcuts only apply when the ordering property is actually guaranteed; confirm which is given before assuming it.
- **Missing the `None`/empty base case** — the single most common source of a crash in tree recursion; every recursive tree function needs an explicit "what happens at an empty subtree" case.
- **Validating a BST with only a local check** (Part C.5) — comparing a node only against its immediate children misses violations against a further ancestor; the correct approach passes a valid range top-down.
- **Height/depth off-by-one**: is a single node's height `0` or `1`? Is an empty tree's height `0` or `-1`? Pick a convention and apply it **consistently** — mixing conventions mid-solution is a very common source of an off-by-one wrong answer.
- **Conflating the recursive return value with the final answer** (Part D) — forgetting that a helper's return value (used by its caller) and the problem's actual answer (sometimes tracked separately) can be different things.
- **Duplicate values in a BST**: decide, explicitly, which side equal values go on (commonly, but not universally, the right subtree) — an unstated convention here silently breaks insert/search consistency.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Traversal:** Binary Tree Inorder/Preorder/Postorder Traversal → Binary Tree Level Order Traversal

**Bottom-Up Combination:** Maximum Depth of Binary Tree → Balanced Binary Tree → Diameter of Binary Tree → Invert Binary Tree → Same Tree → Subtree of Another Tree

**Top-Down / Path:** Path Sum → Binary Tree Maximum Path Sum (a harder variant combining Patterns 3 and 4)

**BST-Specific:** Validate Binary Search Tree (do this one specifically to implement Part C.5's range-passing correctly) → Kth Smallest Element in a BST (implement Part B's inorder proof directly) → Insert into a BST → Lowest Common Ancestor of a BST (a simpler special case of Pattern 6, exploiting ordering)

**LCA (general binary tree):** Lowest Common Ancestor of a Binary Tree

**Construction / Serialization:** Construct Binary Tree from Preorder and Inorder Traversal → Serialize and Deserialize Binary Tree

---

# Part H — References

**Primary text:** CLRS — **Chapter 10, §10.4, "Representing rooted trees"** for the structural basics, and **Chapter 12, "Binary Search Trees"** (direct, dedicated, thorough — formal `TREE-SEARCH`, `TREE-INSERT`, `TREE-DELETE` pseudocode and the height-dependent complexity analysis behind Part B). **Chapter 13, "Red-Black Trees"** for the self-balancing structure referenced in Part A — not required for interviews, but the canonical formal treatment of *why* balance guarantees exist.

**Video references** (search directly by name/series):
- **mycodeschool** — classic, clear tree traversal and BST visualizations.
- **Abdul Bari** — tree fundamentals and BST operation complexity.
- **MIT OpenCourseWare, 6.006** — the BST lecture, covering the same height-dependent complexity analysis as CLRS Chapter 12.
- **Back To Back SWE** — strong coverage of the harder combination problems (Binary Tree Maximum Path Sum, Diameter).
- **NeetCode** — the Trees section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can state the distinction between a binary tree and a BST precisely, and why it matters for complexity claims
- [ ] Can prove, not just state, why inorder traversal of a BST yields sorted order
- [ ] Can explain why tree operation complexity is O(h), and why that is O(log n) balanced vs O(n) skewed
- [ ] Can identify, before coding, whether a new problem needs bottom-up or top-down information flow
- [ ] Can correctly validate a BST using a passed-down range, not a local-only check
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 8

Next: **Phase 2, Topic 8 — Tries**.
