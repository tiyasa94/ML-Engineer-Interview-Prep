# Decision Trees (CART) — Complete Theory, Derivations & Interview Mastery

**Classical ML — Tree-Based Methods, Topic 1** | Companion notebook: [`01-decision-trees-cart.ipynb`](./01-decision-trees-cart.ipynb)

This document goes deeper than a typical "how decision trees work" overview — full derivations, an honest treatment of where the greedy algorithm provably fails, and every claim below verified numerically in the companion notebook against `sklearn`, not just asserted.

---

# Part A — What a Decision Tree Actually Is

A decision tree recursively partitions the feature space into axis-aligned rectangular regions, and predicts a constant value (majority class, or mean, depending on the task) within each region. Structurally: **internal nodes** hold a decision rule (`feature ≤ threshold`), **leaves** hold a prediction. A prediction is made by walking from the root, following the branch dictated by each node's rule, until a leaf is reached.

**CART** (Classification And Regression Trees — Breiman, Friedman, Olshen, Stone, 1984) is the specific algorithm this document covers, and the one `sklearn.tree` implements. It is worth being precise that "decision tree" is a family, not one algorithm: CART restricts every split to **binary** (two-way), and defaults to Gini impurity for classification; older algorithms like **ID3** and its successor **C4.5** (Quinlan) allow multi-way splits and use information gain (or gain ratio, C4.5's bias-corrected version) instead. This document, and the notebook, implement CART specifically.

---

# Part B — The Greedy Recursive Partitioning Algorithm

## The algorithm, in full

```
function BUILD_TREE(X, y, depth):
    if stopping_criterion_met(X, y, depth):
        return Leaf(value = majority_class(y) or mean(y))
    best_feature, best_threshold = argmax over all (feature, threshold) pairs
                                     of impurity_decrease(X, y, feature, threshold)
    if best_gain <= 0:
        return Leaf(value = majority_class(y) or mean(y))
    X_left, y_left, X_right, y_right = split(X, y, best_feature, best_threshold)
    return Node(feature = best_feature, threshold = best_threshold,
                left = BUILD_TREE(X_left, y_left, depth+1),
                right = BUILD_TREE(X_right, y_right, depth+1))
```

This is a **greedy** algorithm: at every node, it picks the single best split available right now, and never reconsiders that choice later, exactly the "no do-overs" character of Greedy algorithms from the DSA syllabus — and, as Part D shows, this greediness has the same real, provable failure mode that any unproven greedy algorithm risks.

## Why an exhaustive threshold search is actually tractable

For a continuous feature, it looks like there are infinitely many candidate thresholds to search over. There aren't: **the optimal split point must lie between two consecutive distinct sorted values of that feature** — any threshold strictly between two data points produces the exact same partition of the data as any other threshold in that same gap, so only the midpoints between consecutive sorted values need to be tested. For `n` examples, this means exactly `n-1` candidate thresholds per feature, not infinitely many.

## Complexity of building the tree

For each of `d` features: sort once, `O(n log n)`. With that sorted order maintained through partitioning (achievable without re-sorting at every node), evaluating all `n-1` thresholds at a given node costs `O(n)` per feature. Across one full level of the tree, the total number of examples processed (summed across all nodes at that level) is still `n`, so each level costs `O(nd)`. For a balanced tree with `O(log n)` levels, total construction cost is:

```
O(n * d * log n)      -- efficient implementation, sorting maintained across levels
O(n * d * log^2 n)    -- naive implementation, re-sorting from scratch at every node
```

Worth being able to state both, and why the efficient version avoids the extra `log n` factor.

---

# Part C — Splitting Criteria: the Theoretical Core

## Classification: three impurity measures, precisely defined

For a node with class proportions `p1, ..., pk`:

- **Gini impurity**: `Gini = 1 - sum(pi^2)` — the probability that two randomly drawn examples from this node have different labels.
- **Entropy**: `H = -sum(pi * log2(pi))` — Information Theory notebook 01's Shannon entropy, applied directly; **information gain** is `H(parent) - sum((nj/n) * H(childj))`, the exact same weighted-reduction structure used for every impurity measure below.
- **Misclassification error**: `1 - max(pi)` — the fraction of examples that would be misclassified if this node predicted its majority class.

## The concavity distinction — a real, precise, under-taught fact

Gini and Entropy are **strictly concave** functions of the class-proportion vector; misclassification error is only **piecewise linear** (weakly concave). This matters concretely: a strictly concave impurity measure always prefers a split that pushes a child toward purity, *even when the overall misclassification rate doesn't change at all* — misclassification error can be completely indifferent between a "useless" split and a genuinely informative one.

**The classic worked example (ESL §9.2)**, verified numerically in the notebook: a balanced 400/400 parent. **Split A** produces children `(300,100)` and `(100,300)`. **Split B** produces children `(200,400)` and `(200,0)` — one perfectly pure child.

| Criterion | Split A gain | Split B gain |
|---|---|---|
| Gini | 0.1250 | **0.1667** |
| Entropy | 0.1887 | **0.3113** |
| Misclassification Error | 0.2500 | 0.2500 |

Misclassification error is **exactly tied** between the two splits (0.25 = 0.25) — it cannot distinguish them at all. Gini and Entropy both correctly, strongly prefer Split B for producing a pure node. This is *why* CART defaults to Gini (or entropy) rather than misclassification error as its splitting criterion, even though misclassification error is what's ultimately being minimized at prediction time — the strictly concave measures are simply better *search signals* during tree construction.

## Regression trees: variance/MSE reduction

For regression, impurity is defined as within-node variance: `Impurity(node) = (1/n) * sum((yi - ybar)^2)` — mean squared error against the node's own mean prediction. The split criterion is the exact same "weighted-reduction" structure as classification, just with this different impurity function: minimize the weighted sum of the two children's MSE, equivalently maximize variance reduction. This is not a separate algorithm — it is Part B's exact same greedy search, with a different impurity function plugged in, which is *why* CART unifies classification and regression under one framework.

---

# Part D — Why Greedy Splitting Is *Not* Globally Optimal

Finding the tree that globally minimizes total impurity (or error) is **NP-hard** (Hyafil & Rivest, 1976) — CART's greedy, one-split-at-a-time search is a heuristic that makes tree construction tractable, at the real cost of no optimality guarantee. This is the same "prove it, don't assume it" territory as the DSA Greedy topic: CART's greediness is not provably safe in the exchange-argument sense that, say, activity selection's greedy choice is — it is a pragmatic compromise.

## A concrete demonstration: the XOR problem

Construct data where the label is the XOR of two binary features — diagonal quadrants share a class. **Verified in the notebook**: splitting on *either* feature alone, at any threshold, produces **exactly zero impurity reduction** — a greedy search sees no gain anywhere and would stop immediately (or split arbitrarily on noise). Yet the two features *together*, via one split on each, perfectly separate all four classes. This is a genuine, structural blind spot: greedy single-feature-at-a-time search cannot "see" an interaction effect that only pays off two levels deep. It is the direct decision-tree analog of a greedy algorithm failing to find a globally optimal answer that requires two coordinated choices at once.

---

# Part E — Overfitting and Pruning

## The bias-variance knob

An unconstrained tree can grow until every leaf is pure (or holds a single example) — zero training error, essentially memorization. This is the deep end of the bias-variance tradeoff (Statistics notebook 03, directly reused): shallow trees are high-bias/low-variance (underfit, too simple to capture real structure); deep trees are low-bias/high-variance (overfit, sensitive to noise in the specific training sample). **Verified in the notebook**: training accuracy climbs monotonically toward 1.0 as depth increases, while test accuracy peaks at a moderate depth and then plateaus or degrades — the bias-variance curve, made concrete with real numbers.

## Pre-pruning: stopping early

`max_depth`, `min_samples_split`, `min_samples_leaf`, `min_impurity_decrease` — all stop growth *before* it happens, based on a threshold chosen ahead of time (typically via cross-validation, Subchapter 5).

## Post-pruning: cost-complexity ("weakest link") pruning — CART's own method, derived

Grow the tree fully, then prune back. Define a cost-complexity objective:

```
R_alpha(T) = R(T) + alpha * |T|
```

where `R(T)` is the tree's total impurity/error, `|T|` is its number of leaves, and `alpha >= 0` penalizes complexity — directly parallel to Linear Models notebook 02's `lambda` regularization strength, applied here to tree size instead of coefficient magnitude.

**The weakest-link algorithm**: for each internal node, define `g(node)` as the *increase* in `R(T)` if that node's entire subtree were collapsed to a single leaf, **divided by** the number of leaves removed in doing so — this ratio is the effective `alpha` at which pruning that specific subtree first becomes worthwhile. Repeatedly find and prune the node with the smallest `g(node)` (the "weakest link" — the subtree buying the least error-reduction per leaf it costs), producing a nested sequence of progressively smaller trees, each optimal for some range of `alpha`. Cross-validation then selects the best `alpha` from this sequence — exactly what `sklearn`'s `cost_complexity_pruning_path` computes, and what the notebook uses directly.

**Verified in the notebook**: on a 500-sample synthetic dataset, the fully-grown tree had 47 leaves and test accuracy 0.8467; the best-pruned tree (selected via this exact `alpha`-path) had 41 leaves and test accuracy 0.8533 — pruning improved *both* generalization and simplicity simultaneously, not a tradeoff between them in this case.

---

# Part F — Handling Real Data

## Categorical features

For **binary classification**, CART can find the *provably optimal* split among all `2^(k-1)-1` possible subsets of a `k`-category feature efficiently: sort the categories by their proportion of the positive class, then only the `k-1` splits along that sorted order need to be checked — a real, non-obvious algorithmic result from Breiman's original work, not an approximation. For **multi-class** problems, the analogous optimal-subset search is NP-hard in general, and practical implementations approximate it.

## Missing values: surrogate splits

CART's specific technique: for each primary split, identify a **surrogate** — a different feature (and threshold) whose split agrees with the primary split as closely as possible on the training data — and fall back to the surrogate whenever the primary feature's value is missing at prediction time. `sklearn`'s implementation does not include this specific mechanism; it's worth knowing as CART's original, more principled answer to missing data versus simple imputation.

## No feature scaling needed — proved, not just claimed

Splits are decided purely by an **ordering** comparison (`feature ≤ threshold`), never by a feature's magnitude directly. Any strictly monotonic transformation of a feature (scaling, shifting, or in fact any order-preserving function) leaves every possible split's data partition **completely unchanged**. **Verified in the notebook**: fitting on raw features versus standardized features versus an arbitrary affine transform (`X*1000 + 50`) produces **identical** trees — same depth, same leaf count, same predictions, exactly. This is a genuine, provable structural property, and a sharp, useful contrast with Linear Models (Ridge/Lasso's coefficient paths *do* depend on feature scale) and, later, SVMs (which also require scaling).

---

# Part G — Complexity, Strengths, Weaknesses

## Complexity summary

- **Training**: `O(n*d*log n)` (efficient implementation, Part B).
- **Prediction**: `O(depth)` per example — `O(log n)` for a balanced tree, `O(n)` worst case for a maximally unbalanced one. This is the exact same height-dependent complexity theme from the DSA Trees topic, reapplied: tree *shape*, not just size, governs prediction cost.

## Strengths
- Directly interpretable (the decision path is a human-readable sequence of rules).
- Handles mixed numeric/categorical data natively, no scaling required (Part F).
- Captures nonlinear relationships and feature interactions automatically, with no manual feature engineering.

## Weaknesses — and exactly what they motivate next
- **High variance / instability**: a small change in the training data can change an early split, cascading into a completely different tree structure below it — this specific weakness is what **Bagging and Random Forests** (Topic 2) directly attack, by averaging many trees to cancel out this instability.
- **Axis-aligned splits only**: a genuinely diagonal decision boundary (Part F's moons-dataset figure) can only ever be approximated by a staircase of axis-aligned steps, however fine.
- **Greedy suboptimality** (Part D): structural blind spots to interaction effects that require coordinated multi-feature splits.
- **Poor extrapolation**: a regression tree's leaves are constants; it can never predict a value outside the range seen during training, unlike a linear model, which extrapolates (for better or worse) by construction.

---

# Part H — References

**Primary text:** ESL (Hastie, Tibshirani, Friedman), **§9.2, "Tree-Based Methods"** — the canonical, thorough treatment; the exact concavity-comparison worked example in Part C is drawn from here. **Murphy, *Probabilistic Machine Learning: An Introduction*, Ch.18** ("Trees, Forests, Bagging, and Boosting") — a strong modern companion treatment, directly bridging into Topics 2-3 of this subchapter.

**Foundational source:** Breiman, Friedman, Olshen, Stone (1984), *Classification and Regression Trees* — the original CART book; the source of the categorical-splitting result and surrogate-splits technique in Part F.

**NP-hardness result:** Hyafil, Rivest (1976), *"Constructing Optimal Binary Decision Trees is NP-Complete"* — the formal backing for Part D's honesty about greedy suboptimality.

**Video references** (search directly by name/series):
- **StatQuest (Josh Starmer)** — a dedicated, exceptionally clear multi-part series on decision trees covering Gini, entropy, regression trees, and pruning individually; the best starting point for building intuition before this document's derivations.
- For scientific ML specifically (Subchapter 6), Steve Brunton remains the standout reference, noted in the chapter roadmap.

---

# Self-Check / Mastery Criteria

- [ ] Can state the CART algorithm's recursive structure from memory, including the stopping condition
- [ ] Can explain, precisely, why only `n-1` threshold candidates per feature need to be checked, not infinitely many
- [ ] Can derive Gini, Entropy, and misclassification error, and reproduce the concavity-comparison worked example showing why misclassification error is a worse splitting signal
- [ ] Can state that regression trees use the same greedy framework with MSE as the impurity function, not a separate algorithm
- [ ] Knows optimal decision tree construction is NP-hard, and can produce a concrete example (XOR) where greedy splitting sees zero gain despite a perfect two-split solution existing
- [ ] Can derive cost-complexity pruning's objective `R_alpha(T) = R(T) + alpha*|T|` and explain the weakest-link algorithm
- [ ] Can prove, not just claim, that decision trees are invariant to monotonic feature transformations
- [ ] Can list decision trees' core weaknesses and name exactly which upcoming topic (Random Forests, Boosting) each one motivates

Next: **Topic 2 — Bagging & Random Forests**.
