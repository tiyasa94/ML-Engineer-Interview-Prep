# Classical ML Roadmap

> Goal: build classical ML — from linear models through scientific ML — the way it's actually asked about in ML Engineer interviews at NVIDIA-class companies: derivations, not just `sklearn.fit()`, plus the numerical/physics-adjacent ML that generic MLE prep skips entirely.

---

# Status

- [ ] Linear Models
- [ ] Tree-Based Methods
- [ ] SVMs & Kernel Methods
- [ ] Unsupervised Learning
- [ ] Evaluation, Features & Practical ML
- [ ] Scientific ML

---

# Learning Strategy

For each topic:
- Derive the model/algorithm from its objective function before looking at how a library implements it.
- Verify every claim numerically — fit both the from-scratch version and `sklearn`'s version on the same data, confirm they agree.
- Connect back to Mathematical Foundations explicitly wherever the derivation uses it (gradients, MLE/MAP, eigendecomposition, Lagrange multipliers, Newton's method) — this chapter is where that math gets used, not re-taught.
- Know the failure modes and assumptions as precisely as the success cases (when does this model break, what does it assume about the data).
- Revisit the self-check checklist a few days later, cold.

---

# Study Framework (per topic)

1. **State the objective function** the model is actually optimizing, before any code.
2. **Derive the solution** (closed-form, gradient-based, or iterative) on paper.
3. **Implement from scratch** in NumPy — no `sklearn` — and verify against `sklearn`'s output on the same toy dataset.
4. **Identify the assumptions** the model makes about the data, and construct one example where a violated assumption visibly breaks it.
5. **State the complexity** (training and inference, time and space).
6. **Connect to Math Foundations** explicitly — which prior derivation is being reused here.
7. **Explain out loud**, under two minutes, before checking notes.

---

# Syllabus

## 1. Linear Models
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| Linear Regression & the Normal Equation | OLS derivation, closed-form vs. gradient descent, assumptions (linearity, homoscedasticity, no multicollinearity) | Calculus (gradient descent), Linear Algebra (normal equation via `(XᵀX)⁻¹Xᵀy`) |
| Ridge / Lasso / Elastic Net | L2 vs. L1 penalties, geometric intuition (diamond vs. circle constraint), why Lasso induces sparsity, bias-variance effect | Statistics (MAP = regularization), Advanced Optimization (coordinate descent for Lasso) |
| Logistic Regression & Softmax | Sigmoid from log-odds, cross-entropy loss derivation, softmax as multi-class generalization, decision boundary geometry | Information Theory (cross-entropy), Advanced Optimization (Newton's method / IRLS) |

## 2. Tree-Based Methods
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| Decision Trees (CART) | Splitting criteria (Gini, entropy, MSE), greedy recursive partitioning, overfitting via depth/leaf size | Information Theory (entropy/information gain, direct reuse) |
| Bagging & Random Forests | Bootstrap aggregation, variance reduction via averaging, feature subsampling, OOB error | Statistics (bootstrap, bias-variance decomposition) |
| Gradient Boosting (AdaBoost → GBM → XGBoost) | Boosting as functional gradient descent, AdaBoost's exponential loss, XGBoost's second-order (Newton) approximation | Advanced Optimization (Newton's method, directly reused) |

## 3. SVMs & Kernel Methods
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| Maximum Margin Classifiers | Geometric margin, why maximizing margin generalizes, the hard-margin optimization problem | Linear Algebra (norms, projections) |
| Soft Margin & Lagrangian Duality | Slack variables, primal vs. dual formulation, KKT conditions, support vectors as the sparse dual solution | Calculus (Lagrange multipliers, direct reuse) |
| The Kernel Trick | Feature-space mapping without computing it explicitly, Mercer's theorem, common kernels (RBF, polynomial) | Linear Algebra (inner products) |

## 4. Unsupervised Learning
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| K-Means & Hierarchical Clustering | Lloyd's algorithm, convergence, k-means++, linkage criteria, dendrograms | — |
| GMM & the EM Algorithm | Latent variable models, E-step/M-step derivation, relationship to soft k-means | Statistics (MLE, direct reuse), Advanced Optimization (Bayesian framing) |
| Dimensionality Reduction Beyond PCA | t-SNE (perplexity, crowding problem), UMAP (topological intuition), when neither PCA nor these suffice | Linear Algebra (PCA/SVD, direct extension) |

## 5. Evaluation, Features & Practical ML
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| Metrics & Cross-Validation | Precision/recall/F1/ROC-AUC, k-fold and its variants, why accuracy misleads | Statistics (hypothesis testing, confidence intervals) |
| Feature Engineering & Selection | Encoding, scaling, interaction terms, filter/wrapper/embedded selection methods | — |
| Imbalanced Data & Leakage | Resampling, class weighting, common leakage patterns (temporal, target leakage) | — |

## 6. Scientific ML (NVIDIA-Specific)
| Topic | Covers | Math Foundations tie-in |
|---|---|---|
| Physics-Informed Neural Networks (PINNs) | Embedding PDE residuals directly into the loss function via autodiff, boundary/initial condition handling | Calculus (autodiff/backprop, direct reuse) |
| Numerical PDE Methods Crossover | Finite difference/element basics, where classical numerical methods meet learned surrogates | Advanced Optimization (numerical stability) |
| Neural ODEs & Operator Learning | Neural ODEs as continuous-depth networks, DeepONet/Fourier Neural Operator (FNO) — learning operators between function spaces, where NVIDIA Modulus/PhysicsNeMo fits | Calculus, Linear Algebra (FNO's spectral/Fourier-space operations) |

---

# Repository Structure

```
Classical ML/
├── 01-Linear Models/
│   ├── 01-linear-regression-normal-equation.{md,ipynb}
│   ├── 02-ridge-lasso-elastic-net.{md,ipynb}
│   ├── 03-logistic-regression-softmax.{md,ipynb}
│   └── figures/
├── 02-Tree-Based Methods/
│   ├── 01-decision-trees-cart.{md,ipynb}
│   ├── 02-bagging-random-forests.{md,ipynb}
│   ├── 03-gradient-boosting.{md,ipynb}
│   └── figures/
├── 03-SVMs & Kernel Methods/
│   ├── 01-maximum-margin-classifiers.{md,ipynb}
│   ├── 02-soft-margin-lagrangian-duality.{md,ipynb}
│   ├── 03-kernel-trick.{md,ipynb}
│   └── figures/
├── 04-Unsupervised Learning/
│   ├── 01-kmeans-hierarchical-clustering.{md,ipynb}
│   ├── 02-gmm-em-algorithm.{md,ipynb}
│   ├── 03-dimensionality-reduction-beyond-pca.{md,ipynb}
│   └── figures/
├── 05-Evaluation, Features & Practical ML/
│   ├── 01-metrics-cross-validation.{md,ipynb}
│   ├── 02-feature-engineering-selection.{md,ipynb}
│   ├── 03-imbalanced-data-leakage.{md,ipynb}
│   └── figures/
└── 06-Scientific ML/
    ├── 01-physics-informed-neural-networks.{md,ipynb}
    ├── 02-numerical-pde-methods-crossover.{md,ipynb}
    ├── 03-neural-odes-operator-learning.{md,ipynb}
    └── figures/
```

Same convention as Mathematical Foundations: `.md` for theory/derivation/intuition, `.ipynb` pre-executed with every claim verified against a from-scratch implementation, `figures/` for embedded visuals.

---

# References

## Already in the repo
- **MML** — *Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) — Ch.9 Linear Regression, Ch.10 PCA (Dim. Reduction, already covered), Ch.11 GMM/EM, Ch.12 Classification with SVMs. The most directly reusable of the four for this chapter — its Part II is essentially a compressed version of this chapter's Subchapters 1, 3, and 4.
- **PSDS** — *Practical Statistics for Data Scientists* (Bruce & Bruce) — bootstrap, resampling, and evaluation-adjacent statistical content feeding Subchapter 5.
- **MacKay** — *Information Theory, Inference, and Learning Algorithms* — entropy/information gain (Subchapter 2's decision trees), and a genuinely excellent, under-used treatment of clustering/EM from an information-theoretic angle (Subchapter 4).
- **Murphy** — *Probabilistic Machine Learning* (2 volumes: *An Introduction*, *Advanced Topics*) — the most comprehensive single modern reference across nearly this entire chapter; has dedicated chapters on linear/logistic regression, trees/ensembles, SVMs, clustering, and EM, all in a consistent probabilistic framing. Treat as the default "go deeper here" reference when MML doesn't cover a topic in enough depth.

## Worth adding — classical ML's actual home textbooks
- **ESL** — *The Elements of Statistical Learning* (Hastie, Tibshirani, Friedman) — free at [statlearning.com](https://hastie.su.domains/ElemStatLearn/) — **the** canonical reference for Subchapter 2 specifically (Ch.9–10, 15: trees, boosting, random forests) and strong throughout Subchapters 1 and 3. More mathematically dense than MML/Murphy; the standard "derive it properly" text for classical ML.
- **ISL** — *An Introduction to Statistical Learning* (James, Witten, Hastie, Tibshirani) — free at the same site — ESL's gentler, more applied sibling; excellent for Subchapter 5 (Ch.5, resampling/cross-validation) and as a first pass before ESL's denser treatment of the same topics.
- **Bishop** — *Pattern Recognition and Machine Learning* — classic, thorough treatment of SVMs (Ch.7) and GMM/EM (Ch.9) with a Bayesian throughline; a strong alternative angle to Murphy on the same material.
- **Cristianini & Shawe-Taylor** — *An Introduction to Support Vector Machines* — if Subchapter 3's duality/kernel derivations need a source dedicated entirely to SVMs rather than a chapter of a broader book.

## Scientific ML (Subchapter 6) — no textbook covers this yet as canonically as ESL covers trees; the primary sources are the founding papers and NVIDIA's own documentation
- Raissi, Perdikaris, Karniadakis (2019), **"Physics-Informed Neural Networks"** (*Journal of Computational Physics*) — the foundational PINNs paper.
- Lu, Jin, Karniadakis (2021), **"DeepONet"** — the foundational operator-learning paper.
- Li et al. (2020), **"Fourier Neural Operator for Parametric PDEs"** — the FNO paper, directly relevant to NVIDIA's Modulus/PhysicsNeMo stack.
- **NVIDIA Modulus / PhysicsNeMo documentation** — the direct, practical reference for how these ideas are implemented in NVIDIA's own framework; worth reading alongside the papers, not instead of them.
- Chen et al. (2018), **"Neural Ordinary Differential Equations"** — the foundational Neural ODE paper.

## Video references (search directly by name/series)
- **StatQuest (Josh Starmer)** — the single best starting point for nearly every topic in Subchapters 1–4: linear/logistic regression, decision trees, random forests, AdaBoost/gradient boosting/XGBoost, SVMs, PCA, clustering, EM — consistently clear, visual, intuition-first, and a strong complement to this chapter's from-scratch-derivation approach.
- **Andrew Ng, Stanford CS229 / Machine Learning course** — more mathematically thorough than StatQuest, closer to this chapter's own derivation-first style; the lecture notes (freely available) are worth reading alongside the videos.
- **Steve Brunton (University of Washington)** — specifically excellent for Subchapter 6; his channel covers data-driven dynamical systems, physics-informed ML, and the numerical-methods-meets-ML crossover directly, and is the best video complement to the scientific ML papers above.
- **Yannic Kilcher** — paper-walkthrough style videos; useful specifically for the PINNs/DeepONet/FNO papers once the foundational theory (from this chapter's own topics) is solid.

---

# Next Up

Start with **Subchapter 1, Topic 1 — Linear Regression & the Normal Equation**.
