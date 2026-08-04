# Mathematical Foundations Roadmap

> Primary References: **Mathematics for Machine Learning** (Deisenroth, Faisal, Ong — free at [mml-book.com](https://mml-book.com)) and **Practical Statistics for Data Scientists** (Bruce & Bruce, O'Reilly), supplemented where those two do not reach — Information Theory (Cover & Thomas, *Elements of Information Theory*; MacKay, *Information Theory, Inference, and Learning Algorithms*) and parts of Advanced Optimization (Petersen & Pedersen, *The Matrix Cookbook*; Nocedal & Wright, *Numerical Optimization*; Kingma & Ba, the Adam paper).

> Goal: build the math that actually gets used and asked about in ML Engineer interviews — not math for its own sake — with every derivation, identity, and claim verified in code rather than taken on faith, and every topic connected explicitly to a real ML technique.

---

# Status

This chapter is **complete** — 5 subchapters, 15 topics, each a paired `.md` + `.ipynb`, every notebook executed with zero errors and every numerical claim checked, not asserted.

- [x] Linear Algebra
- [x] Calculus
- [x] Statistics
- [x] Information Theory
- [x] Advanced Optimization & Numerical Methods

This file documents what was built and doubles as the revision path back through it — treat it the way `DSA-Roadmap.md` is treated for DSA: the map to re-walk during revision, not just a record.

---

# Learning Strategy

For each topic:

- Read theory intuition-first — the picture before the formula.
- Derive it by hand before reading the notebook's derivation.
- Verify every identity/claim numerically (gradient checks, benchmarks, real measured numbers) rather than trusting it on sight.
- Build or re-run the visualization until the geometric intuition is automatic, not memorized.
- Connect it explicitly to at least one real ML technique — every notebook in this chapter does this by design.
- Revisit the self-check checklist a few days later, cold.
- Explain it out loud, unaided, as if to an interviewer.

---

# Study Framework (per notebook)

The per-topic strategy above governs a subchapter; this governs a single notebook, every time.

1. **Read the `.md` first** — intuition, the actual derivation, a worked visual, why it matters for ML. Do not open the `.ipynb` yet.
2. **Attempt the derivation on paper** before reading further — even a rough attempt beats reading a finished derivation cold.
3. **Run the `.ipynb` top to bottom** and compare the executed, verified output against your own derivation. Where they disagree, that disagreement is the actual learning moment.
4. **Break it on purpose** — change a matrix, a distribution's parameters, a learning rate; predict what happens before re-running; see if the prediction holds.
5. **Answer the self-check checklist** at the bottom of the `.md`, cold, no notes.
6. **Explain the concept out loud**, start to finish, in under two minutes.
7. **Log confusion points** and schedule a revisit (Revision System, below).

---

# Syllabus: Subchapters, in Order

The file tree sorts alphabetically; this is the intended reading order — each subchapter leans on the one before it.

## 1. Linear Algebra
| Notebook | Covers | Yield |
|---|---|---|
| 01 — Vectors, Spaces & Transformations | vector spaces, linear independence, basis, rank, linear maps | |
| 02 — Norms, Inner Products & Projections | L1/L2/L∞ norms, cosine similarity, orthogonal projection | |
| 03 — Eigenvalues, SVD & PCA | eigendecomposition, SVD, PCA derived from first principles | ⭐ |

## 2. Calculus
| Notebook | Covers | Yield |
|---|---|---|
| 01 — Differentiation, Gradients & Jacobians | derivatives, gradients, Jacobians, gradient checking | |
| 02 — Chain Rule, Backprop & Autodiff | backprop as chain rule, computational graphs, vanishing gradients | ⭐ |
| 03 — Optimization: GD & Convexity | gradient descent, learning rate dynamics, convexity, Lagrange multipliers | |

## 3. Statistics
| Notebook | Covers | Yield |
|---|---|---|
| 01 — Probability Foundations & Bayes | joint/marginal/conditional probability, Bayes' theorem, independence vs. correlation | |
| 02 — Distributions, MLE & MAP | CLT, MLE, MAP, the MAP-to-regularization connection | ⭐ |
| 03 — Hypothesis Testing & Bias-Variance | p-values, confidence intervals, bootstrap, bias-variance decomposition | |

## 4. Information Theory
*(No dedicated coverage in either primary book — follows the standard treatment; noted explicitly in notebook 01.)*
| Notebook | Covers | Yield |
|---|---|---|
| 01 — Entropy & Cross-Entropy | Shannon entropy, cross-entropy, the cross-entropy/MLE connection | |
| 02 — KL Divergence & Mutual Information | KL divergence, forward vs. reverse KL, mutual information vs. correlation | ⭐ |
| 03 — ML Applications of Entropy | decision tree splits, distillation/temperature scaling, perplexity | |

## 5. Advanced Optimization & Numerical Methods
*(Extends Calculus/Statistics beyond primary-book coverage — matrix calculus identities, Newton's method, and Adam follow standard external references, noted per-notebook.)*
| Notebook | Covers | Yield |
|---|---|---|
| 01 — Matrix Calculus, Hessian & Newton's Method | gradient/trace identities, the Hessian, Newton's method derived | |
| 02 — Adam & Modern Optimizers | momentum → RMSprop → Adam derived and benchmarked, AdamW | ⭐ |
| 03 — Numerical Stability & Bayesian Foundations | log-sum-exp, softmax overflow, LLN vs. CLT, Bayesian sequential updating | |

**Cross-references worth tracing deliberately during revision**, since they are what make this a chapter and not 15 unrelated notebooks:
- PCA (Linear Algebra 03) is re-derived via gradients in Calculus 02, and its Hessian/curvature reading gets a second pass in Advanced Optimization 01.
- The softmax + cross-entropy gradient (Calculus 02) is explained information-theoretically in Information Theory 01, and its numerical failure mode (overflow) is fixed in Advanced Optimization 03.
- MAP's connection to L2 regularization (Statistics 02) is the same shrinkage idea behind AdamW's decoupled weight decay (Advanced Optimization 02).
- The `Y=X²` "uncorrelated but dependent" example (Statistics 01) is revisited with mutual information (Information Theory 02).
- Bayes' theorem (Statistics 01) is extended to sequential/Bayesian updating (Advanced Optimization 03).

---

# Repository Structure

```
Mathematical Foundations/
├── Linear Algebra/
│   ├── 01-vectors-spaces-transformations.{md,ipynb}
│   ├── 02-norms-inner-products-projections.{md,ipynb}
│   ├── 03-eigenvalues-svd-pca.{md,ipynb}                              ⭐
│   └── figures/
├── Calculus/
│   ├── 01-differentiation-gradients-jacobians.{md,ipynb}
│   ├── 02-chain-rule-backprop-autodiff.{md,ipynb}                     ⭐
│   ├── 03-optimization-gradient-descent-convexity.{md,ipynb}
│   └── figures/
├── Statistics/
│   ├── 01-probability-foundations-bayes.{md,ipynb}
│   ├── 02-distributions-mle-map.{md,ipynb}                            ⭐
│   ├── 03-hypothesis-testing-bias-variance.{md,ipynb}
│   └── figures/
├── Information Theory/
│   ├── 01-entropy-cross-entropy.{md,ipynb}
│   ├── 02-kl-divergence-mutual-information.{md,ipynb}                 ⭐
│   ├── 03-ml-applications-entropy.{md,ipynb}
│   └── figures/
└── Advanced Optimization & Numerical Methods/
    ├── 01-matrix-calculus-hessian-newton.{md,ipynb}
    ├── 02-adam-and-modern-optimizers.{md,ipynb}                       ⭐
    ├── 03-numerical-stability-bayesian-foundations.{md,ipynb}
    └── figures/
```

Every topic is a paired `.md` + `.ipynb`:
- **`.md`** — theory, intuition, the actual derivation, a worked visual, and why it matters for ML. Ends with a self-check checklist.
- **`.ipynb`** — companion code, pre-executed, every claim in the `.md` verified numerically (gradient checks, measured benchmarks, real computed outputs) — plots render immediately, no need to re-run, but structured so the derivations can be cleared and redone by hand.
- **`figures/`** — the PNGs the `.md` files embed, regenerated by running the notebook.

---

# Master Formula Sheet

The one line per topic worth being able to write from memory, unaided.

**Linear Algebra**
- Eigenvector equation: `Av = λv`
- SVD: `A = UΣVᵀ`
- Orthogonal projection: `P = Bᵀ(BBᵀ)⁻¹B`
- PCA: eigendecomposition of the covariance matrix; top eigenvector = direction of maximum variance

**Calculus**
- Gradient descent: `x_{t+1} = x_t - lr·∇f(x_t)`
- Softmax + cross-entropy gradient: `∂L/∂logits = softmax(logits) - one_hot(y)`
- Lagrangian (constrained optimization): `L(x,λ) = f(x) - λg(x)`, optimum where `∇f = λ∇g`

**Statistics**
- Bayes' theorem: `P(H|E) = P(E|H)·P(H) / P(E)`
- MLE: `argmax_θ P(data|θ)`
- MAP: `argmax_θ P(data|θ)·P(θ)` — MAP with a Gaussian prior on weights = ridge regression
- Bias-variance: `Expected Error = Bias² + Variance + Irreducible Noise`

**Information Theory**
- Entropy: `H(p) = -Σ p(x)·log p(x)`
- Cross-entropy: `H(p,q) = -Σ p(x)·log q(x)`
- KL divergence: `KL(p‖q) = H(p,q) - H(p)`, always ≥ 0, not symmetric
- Mutual information: `I(X;Y) = KL(P(X,Y) ‖ P(X)P(Y))`

**Advanced Optimization & Numerical Methods**
- Quadratic form gradient: `d(xᵀAx)/dx = (A+Aᵀ)x`
- Newton's method: `x_new = x - H⁻¹∇f(x)`
- Adam: `m̂_t = m_t/(1-β₁ᵗ)`, `v̂_t = v_t/(1-β₂ᵗ)`, `θ_t = θ_{t-1} - lr·m̂_t/(√v̂_t + ε)`
- log-sum-exp trick: `log(Σexp(xᵢ)) = m + log(Σexp(xᵢ-m))`, where `m = max(x)`

---

# Resources

- **MML** — free at [mml-book.com](https://mml-book.com); cited by exact section/page throughout Linear Algebra, Calculus, and most of Statistics.
- **PSDS** — Bruce & Bruce, O'Reilly; cited for the applied-statistics half of Statistics (bootstrap, confidence intervals, hypothesis testing, bias-variance) and for decision-tree entropy in Information Theory.
- **Cover & Thomas**, *Elements of Information Theory*, and **MacKay**, *Information Theory, Inference, and Learning Algorithms* (free online) — the standard treatment used for Information Theory, since neither primary book covers it in depth.
- **Petersen & Pedersen**, *The Matrix Cookbook* (free online) — matrix calculus identity reference for Advanced Optimization 01.
- **Nocedal & Wright**, *Numerical Optimization* — Newton's method treatment for Advanced Optimization 01.
- **Kingma & Ba** (2015), the original Adam paper — Advanced Optimization 02.
- This repo's own `Python & DSA/01-fundamentals-syntax-control-flow.ipynb` and `02-data-structures-native.ipynb` for the NumPy/Python mechanics this chapter's code assumes as background.

---

# Revision System

Every notebook already ends with a self-check checklist — log the result the same way `DSA/revision-log.md` logs problems:

| Date | Subchapter / Notebook | Self-check result | Revisit 1 (+3d) | Revisit 2 (+14d) |
|---|---|---|---|---|
| | | all checked cold / needed the `.md` again / needed to re-derive on paper | | |

- **All checked cold** → revisit at +14 days.
- **Needed to reread the `.md`** → revisit at +3 days.
- **Needed to re-derive on paper before it clicked** → treat as not yet learned; revisit at +3 days and repeat the full Study Framework, not just the checklist.

A topic counts as *learned*, not just *read*, only after a cold, unaided pass through its self-check checklist.

---

# Mastery Criteria (per topic)

A topic is not "done" because the notebook ran once — it's done when, cold:

- [ ] The core derivation can be reproduced on paper from memory, not just recognized when read.
- [ ] Every self-check checklist item can be answered without opening the `.md`.
- [ ] At least one cross-reference to another subchapter (list above) can be stated without prompting.
- [ ] At least one concrete ML technique the concept underlies can be named and explained, not just cited.
- [ ] The concept can be explained out loud, start to finish, in under two minutes.
- [ ] For the ⭐ flagship notebooks specifically: the concept can be derived on a whiteboard under time pressure, not just explained conversationally — these are the ones interviewers actually ask to see derived.

---
