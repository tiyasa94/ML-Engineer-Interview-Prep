# Tries — Complete Theory, Patterns & Interview Mastery

**Phase 2 — Linked Structures & Trees, Topic 8** | Practice problems: NeetCode's Tries section, after this document, not before.

---

# Part A — What a Trie Actually Is

A trie (prefix tree) is a tree specialized for storing **strings**, where each edge (or indexed child) represents one character, and the path from the root to any node represents the **prefix** spelled out along that path. The root represents the empty string. Crucially, each node needs an explicit **end-of-word marker** (a boolean flag) — without it, there is no way to distinguish "this prefix happens to exist because a longer word passes through it" from "this exact string was itself inserted as a complete word."

```
class TrieNode:
    def __init__(self):
        self.children = {}          # char -> TrieNode
        self.is_end_of_word = False
```

**Complexity**: insert and exact-word search are both **O(L)**, where `L` is the length of the string being inserted/searched — critically, **independent of how many other words are stored** (`n`). This is the entire selling point over alternatives.

---

# Part B — Why a Trie Beats a Hash Set for This Job

A hash set (Topic 1) also gives O(L) exact-match search on average. The difference is **prefix queries**: "does any stored word start with this prefix?" A hash set cannot answer this without scanning every stored word — O(n·L) in the worst case. A trie answers it in O(L): simply walk the prefix's characters from the root; if that walk completes without falling off the trie, the prefix exists, structurally, by construction — no scanning of other words required at all. **The existence of a path IS the existence of the prefix** — this structural fact is the entire reason a trie exists as a distinct data structure rather than "just use a hash set."

**Space**: worst case O(ALPHABET_SIZE × N × L) with fixed-size array children (one slot per possible character at every node, regardless of how many are actually used); a dict-based children structure (as above) is typically far more space-efficient in practice for sparse alphabets, at a small constant-factor cost versus direct array indexing.

---

# Part C — The Core Patterns

### 1. Basic Trie: Insert, Search, StartsWith

```
def insert(root, word):
    node = root
    for c in word:
        if c not in node.children:
            node.children[c] = TrieNode()
        node = node.children[c]
    node.is_end_of_word = True

def search(root, word):
    node = root
    for c in word:
        if c not in node.children:
            return False
        node = node.children[c]
    return node.is_end_of_word            # must be a COMPLETE word, not just a valid prefix

def starts_with(root, prefix):
    node = root
    for c in prefix:
        if c not in node.children:
            return False
        node = node.children[c]
    return True                            # reaching here at all means the prefix exists
```
**Signals:** "implement a trie/prefix tree", "starts with", "prefix".

### 2. Wildcard Search (DFS Through the Trie)

When a query can contain a wildcard character (matches any single character), search becomes a DFS/backtracking traversal instead of a straight-line walk: at a wildcard position, branch into *every* child rather than following one specific path.
```
def search_with_wildcard(node, word, i):
    if i == len(word):
        return node.is_end_of_word
    c = word[i]
    if c == '.':
        return any(search_with_wildcard(child, word, i + 1) for child in node.children.values())
    if c not in node.children:
        return False
    return search_with_wildcard(node.children[c], word, i + 1)
```
**Signals:** "wildcard", "'.' matches any character", "pattern matching".

### 3. Trie + DFS on a Grid (Multi-Word Search)

The trie's real payoff shows up when searching a grid (Phase 2/3 territory) for **many** words simultaneously: instead of running a separate DFS per word (O(words × cells × 4^L)), build one trie from all target words, then run a **single** DFS over the grid, using the trie to prune — if the current path through the grid doesn't correspond to any node in the trie, that entire branch can be abandoned immediately, since no word in the dictionary could possibly continue from there.
**Signals:** "list of words", "search in a grid/board" — the co-occurrence of "many words" and "grid search" is the strongest signal for this specific combination.

### 4. Prefix Enumeration / Autocomplete

Walk to the node representing a given prefix (Pattern 1's `starts_with` logic), then DFS from that node, collecting every complete word (`is_end_of_word == True`) found in the subtree beneath it.
**Signals:** "autocomplete", "all words with a given prefix", "suggest".

### 5. Binary Trie (Advanced, Bitwise)

A trie can be built over the **binary representations** of numbers instead of characters — each node has at most 2 children (bit 0 or bit 1). This enables O(bit-length) queries like "find the number in this set that XORs with a given value to produce the maximum result" — greedily choosing, at each bit position, the child that *disagrees* with the query's current bit (since XOR is maximized by mismatched bits). A distinct, less common but genuinely elegant pattern worth recognizing when it appears.
**Signals:** "maximum XOR", "XOR of two numbers in an array".

---

# Part D — Optimization Principles

- **The headline win**: O(L) per operation, independent of `n` — versus a naive "check against every stored word" approach at O(n·L). Worth stating this contrast explicitly with the variables named, not just "it's faster."
- **Trie-guided pruning for multi-target search** (Pattern 3) is the more subtle, higher-value optimization: it's not just about single-word lookup speed, it's about avoiding **redundant, repeated search work** across many words by sharing their common prefixes in one structure — the same "avoid redundant recomputation" theme recurring since Phase 0, applied at the level of entire search branches rather than individual values.

---

# Part E — Recognizing This Pattern in an Unfamiliar Problem

| Phrase in the problem | Likely pattern |
|---|---|
| "implement a trie/prefix tree" | Basic Trie (Pattern 1) |
| "wildcard", "'.' matches any character" | Wildcard DFS (Pattern 2) |
| "list/dictionary of words" + "grid/board search" | Trie + Grid DFS (Pattern 3) |
| "autocomplete", "all words with prefix" | Prefix Enumeration (Pattern 4) |
| "maximum XOR" | Binary Trie (Pattern 5) |

---

# Part F — Common Gotchas & Interview Traps

- **Forgetting the end-of-word marker** — without it, `search("app")` cannot be distinguished from `starts_with("app")` when "apple" (but not "app" itself) has been inserted; this is the single most common trie bug.
- **Array-based vs. dict-based children**: a fixed 26-slot array is faster and simpler when the alphabet is known to be lowercase English letters only; a dict is required (or at least much more convenient) for a general/unknown character set — choosing the array approach for a problem that doesn't actually guarantee a restricted alphabet is a real correctness risk, not just a style choice.
- **Wildcard search without proper branching** — treating a wildcard as "skip this character" instead of "try every possible child" gives an incorrect, overly permissive or overly restrictive match.
- **Rebuilding the trie unnecessarily**: for multi-word grid search (Pattern 3), the trie should be built **once**, up front, from all target words — rebuilding it per grid cell or per search attempt defeats its entire purpose.
- **Marking visited cells during grid DFS and forgetting to un-mark on backtrack** — a general backtracking gotcha (Phase 3 preview) that shows up immediately in Pattern 3's grid search.

---

# Part G — Curated Practice List (NeetCode / LeetCode, by pattern — no solutions here by design)

**Basic Trie:** Implement Trie (Prefix Tree) — build this one completely from scratch, it is the foundation every other trie problem sits on

**Wildcard:** Design Add and Search Words Data Structure

**Grid + Multi-Word:** Word Search II (the canonical trie-pruned grid DFS — Pattern 3, in full)

**Advanced/Optional:** Maximum XOR of Two Numbers in an Array (binary trie, Pattern 5)

---

# Part H — References

**Primary text:** Cormen, Leiserson, Rivest, Stein — *Introduction to Algorithms* (CLRS) does **not** cover tries directly — it is a string-algorithms-specific structure outside CLRS's core chapters. The standard, honest reference instead: Sedgewick & Wayne, *Algorithms* (4th edition), **§5.2, "Tries"** — a full, dedicated, rigorous treatment including array-based (R-way) tries and the space/time tradeoffs versus other string search structures. Knuth's *The Art of Computer Programming, Volume 3* also touches tries in the context of sorting and searching, for anyone wanting the deepest possible treatment.

**Video references** (search directly by name/series):
- **Abdul Bari** — trie fundamentals and construction.
- **William Fiset** — a specifically strong, implementation-focused trie walkthrough (also has broader data structure playlist coverage worth returning to).
- **NeetCode** — the Tries section of the roadmap; use after an independent attempt.

---

# Self-Check / Mastery Criteria

- [ ] Can implement a trie's insert/search/startsWith from scratch, including the end-of-word marker, without referencing a template
- [ ] Can explain, precisely, why a trie answers prefix queries in O(L) when a hash set cannot without scanning
- [ ] Can implement wildcard search via branching DFS through the trie
- [ ] Can explain why building one trie for Word Search II beats running a separate DFS per word
- [ ] Has independently solved at least one problem from each pattern category in Part G before moving to Topic 9

Next: **Phase 2, Topic 9 — Heap / Priority Queue**.
