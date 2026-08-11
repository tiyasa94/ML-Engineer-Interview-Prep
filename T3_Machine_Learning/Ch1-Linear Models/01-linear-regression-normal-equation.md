# 01 — Linear Regression & the Normal Equation

**Companion notebook:** [`01-linear-regression-normal-equation.ipynb`](./01-linear-regression-normal-equation.ipynb)
**Book references:** MML §9.1–9.2 (Linear Regression, pp.303–313); ESL §3.1–3.2 for the classical statistical treatment.

---

## The objective function and its closed-form solution

Model: `y = Xβ + ε`. Objective: minimize the residual sum of squares `RSS(β) = ‖y - Xβ‖²`.

**Deriving the normal equation** — this is Advanced Optimization notebook 01's quadratic-form gradient identity, applied directly: expand `RSS(β) = (y-Xβ)ᵀ(y-Xβ)`, take the gradient with respect to `β`, set it to zero:

```
∇β RSS(β) = -2Xᵀ(y - Xβ) = 0
       XᵀXβ = Xᵀy
          β = (XᵀX)⁻¹ Xᵀy       (the normal equation)
```

Verified in the notebook against `sklearn.linear_model.LinearRegression` — exact match to `1e-8`.

## Gradient descent: the iterative alternative

The normal equation costs `O(p³)` to invert a `p×p` matrix — fine for a handful of features, prohibitive for very large `p`. Gradient descent (Calculus notebook 03) reaches the same optimum iteratively, at `O(np)` per step, no matrix inversion required. Verified in the notebook: gradient descent's converged solution matches the closed-form normal equation to `1e-3` on standardized data.

## The geometry: OLS is a projection

`ŷ = Xβ̂` is literally the **orthogonal projection** of `y` onto the column space of `X` (Linear Algebra notebook 02's projection formula, `P = X(XᵀX)⁻¹Xᵀ`, applied to regression). The defining property of any orthogonal projection: the residual `y - ŷ` is orthogonal to everything in the space being projected onto — `Xᵀ(y - ŷ) = 0`. This isn't a separate fact to memorize; it's the normal equation rearranged, and it holds to numerical precision (verified: `X²ᵀ residual ≈ [-1.4e-15, -2.1e-15]`) for any least-squares fit.

## When the normal equation breaks: multicollinearity

`(XᵀX)⁻¹` requires `XᵀX` to be invertible, which fails — or nearly fails — when predictor columns are linearly dependent, or close to it. The **condition number** of `XᵀX` measures exactly how close to singular it is; a large condition number means small changes in the data produce huge changes in the estimated coefficients, and matrix inversion becomes numerically unstable even when technically still invertible.

Measured in the notebook: adding a single near-duplicate predictor column pushed the condition number from **1.72** to **3,217,130** — a roughly 1.9-million-fold increase from one redundant feature.

This instability — not just "the math technically still works" but "the *numerics* stop being trustworthy" — is exactly what motivates the next topic. Adding a small penalty `λI` to `XᵀX` before inverting (`(XᵀX + λI)⁻¹`) fixes the singularity directly: this is Ridge Regression, and it is not a heuristic patch — it is a principled solution derived from a Bayesian prior on the coefficients.

---

## Self-check before moving on

- [ ] I can derive the normal equation from the RSS objective via its gradient, from memory
- [ ] I can explain why OLS is a projection, and why residuals are orthogonal to the fitted values as a direct consequence
- [ ] I can state the complexity tradeoff between the normal equation (O(p³)) and gradient descent (O(np) per step)
- [ ] I can explain, precisely, what multicollinearity does to `XᵀX`'s condition number and why that matters numerically, not just theoretically
- [ ] I can state linear regression's core assumptions (linearity, homoscedasticity, no multicollinearity, independence) and recognize when each is violated

Next: [`02-ridge-lasso-elastic-net.md`](./02-ridge-lasso-elastic-net.md)
