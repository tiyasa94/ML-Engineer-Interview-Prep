# 03 - Eigenvalues, Eigenvectors, SVD & PCA ⭐

**The single highest-interview-yield file in this chapter.** If you only have time to deeply master one linear algebra topic, make it this one.

**Companion code:** [`03-eigenvalues-svd-pca.ipynb`](./03-eigenvalues-svd-pca.ipynb)
**Book reference:** *Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) (https://drive.google.com/file/d/1KsG9bntb0vD9QrUvHrlR2vqBQSr_V0J9/view) - §4.1–4.6, pp.99–134 (Determinant and Trace, Eigenvalues and Eigenvectors, Cholesky Decomposition, Eigendecomposition and Diagonalization, Singular Value Decomposition, Matrix Approximation)

---

## Eigenvectors: the directions a matrix doesn't rotate

For a matrix `A`, an eigenvector `v` and its eigenvalue `λ` satisfy `Av = λv` - applying `A` to `v` doesn't change its direction, only scales it by `λ`.

![Eigenvectors as the axes of the transformed ellipse](./figures/eigenvectors.png)

The unit circle (gray) gets transformed by `A` into an ellipse (blue). The eigenvectors (red, green) are exactly the **axes of that ellipse** - every other direction on the circle gets rotated *and* stretched, but these two directions only stretch, by a factor of their eigenvalue.

**Symmetric matrices** (like covariance matrices - this is why they matter so much in ML) have a special guarantee: their eigenvalues are always real, and their eigenvectors are always orthogonal to each other. That guarantee is what makes PCA well-defined.

## PCA: this picture *is* the derivation

PCA answers: "what's the direction along which my data varies the most?" The answer is: **the eigenvector of the covariance matrix with the largest eigenvalue.**

![PCA principal axes on 2D data](./figures/pca_2d.png)

Walking through why:
1. You want to find a direction `w` (with `‖w‖=1`) that maximizes the variance of the data projected onto it: `Var(Xw) = wᵀΣw`, where `Σ` is the covariance matrix.
2. Maximizing `wᵀΣw` subject to `‖w‖=1` is a constrained optimization problem. Using a Lagrange multiplier: maximize `wᵀΣw - λ(wᵀw - 1)`.
3. Taking the derivative w.r.t. `w` and setting it to zero gives `Σw = λw` - **that's exactly the eigenvector equation.**
4. So the variance-maximizing direction is the top eigenvector of `Σ`, and the variance captured along it is that eigenvector's eigenvalue.

```python
def pca_from_scratch(X, n_components):
    X_centered = X - X.mean(axis=0, keepdims=True)
    cov = (X_centered.T @ X_centered) / (X_centered.shape[0] - 1)
    eigvals, eigvecs = np.linalg.eigh(cov)           # symmetric -> eigh, not eig
    order = np.argsort(eigvals)[::-1]                # eigh returns ascending order
    eigvals, eigvecs = eigvals[order], eigvecs[:, order]
    components = eigvecs[:, :n_components].T
    X_projected = X_centered @ components.T
    return X_projected, components, eigvals[:n_components]
```

In the figure, PC1 captures the direction of maximum spread; PC2 (orthogonal to PC1, guaranteed by the symmetric-matrix property above) captures what's left.

## SVD: the generalization that always exists

Eigendecomposition requires a square, diagonalizable matrix. **SVD works for any matrix** - including non-square ones - because it's built from the eigendecompositions of `AᵀA` and `AAᵀ`, which are always symmetric and positive semi-definite (hence always diagonalizable, by the guarantee above).

`A = UΣVᵀ`, where `U` and `V` are orthogonal matrices (their columns are the eigenvectors of `AAᵀ` and `AᵀA` respectively) and `Σ` is diagonal, holding the **singular values** (square roots of the eigenvalues of `AᵀA`).

**Relationship to eigendecomposition:** for a symmetric positive semi-definite matrix, SVD and eigendecomposition coincide exactly (`U = V`, singular values = eigenvalues). This is why, for the covariance matrix in PCA, you'll see both `eigh(cov)` and `svd(X_centered)` used interchangeably in practice - they give the same answer, and the SVD route is often more numerically stable since it avoids explicitly forming `Σ = XᵀX`.

## Low-rank approximation - why this matters practically

The Eckart–Young theorem says the best rank-k approximation of a matrix (in a least-squares sense) is obtained by keeping only the top k singular values/vectors and zeroing out the rest. This is the mathematical basis for compression, denoising, and dimensionality reduction across ML.

![SVD low-rank image approximation](./figures/svd_compression.png)

Even a synthetic image with real structure is captured almost entirely by the first ~20 singular values out of 120 - the rest is mostly the added noise. This is *exactly* the intuition behind why PCA-based dimensionality reduction works on real data: most of the "signal" concentrates in a small number of directions, and you can discard the rest with minimal information loss.

![Singular value spectrum](./figures/svd_spectrum.png)

That steep drop-off in the spectrum is the visual answer to "how many components/singular values do I actually need?" - a question that comes up constantly in both PCA and low-rank model compression (e.g., LoRA fine-tuning deliberately exploits this: full weight updates during fine-tuning are empirically low-rank, so constraining the update to rank `r` loses little while cutting trainable parameters enormously).

---

## Self-check before moving on

- [ ] I can state `Av = λv` and explain what it means geometrically
- [ ] I can derive PCA from "maximize projected variance" to "eigenvectors of the covariance matrix," writing out each step
- [ ] I can explain why SVD always exists but eigendecomposition doesn't, and when the two coincide
- [ ] I can explain what a low-rank approximation is and why the singular value spectrum tells you how many components you need
- [ ] I can connect this to at least one real ML technique beyond PCA (e.g., LoRA, recommender systems via matrix factorization)

This closes out the Linear Algebra subchapter. Next up: **Calculus** (differentiation, gradients, backprop math) - building on the vector/matrix machinery from this chapter.
