# 01 - Vectors, Spaces & Linear Transformations

**Companion code:** [`01-vectors-spaces-transformations.ipynb`](./01-vectors-spaces-transformations.ipynv)

**Book references:**
- *Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) (https://drive.google.com/file/d/1KsG9bntb0vD9QrUvHrlR2vqBQSr_V0J9/view) 
- §2.1–2.3, pp.19–34 (Systems of Linear Equations, Matrices, Solving Systems of Linear Equations)
- §2.4–2.8, pp.35–63 (Vector Spaces, Linear Independence, Basis and Rank, Linear Mappings, Affine Spaces)


## Introduction to Vectors
Linear algebra is the study of vectors and certain algebra rules to manipulate vectors. In general, vectors are special objects that can be added together and multiplied by scalars to produce another object of the same kind. From an abstract mathematical viewpoint, any object that satisfies these two properties can be considered a vector. Here are some examples of such vector objects:
- Geometric vectors
- Polynomials
- Audio signals
- Elements of Rn

![Different Vector Types](./book_figures/different_vector_types.png)

Linear algebra focuses on the similarities between these vector concepts. 


## Systems of linear equations

The general form of a system of m linear equations with n variables (x_1, x_2,..., x_n) is written algebraically as a set of linear equations where coefficients (a_{ij}) and constants (b_{i}) are real or complex numbers:

![General form of system of linear equations](./book_figures/system_of_le.png)

In general, for a real-valued system of linear equations we obtain either no, exactly one, or infinitely many solutions.

In a system of linear equations with two variables x1, x2, each linear equation
defines a line on the x1x2-plane. Since a solution to a system of linear
equations must satisfy all equations simultaneously, the solution set is the
intersection of these lines. This intersection set can be a line (if the linear
equations describe the same line), a point, or empty (when the lines are
parallel). 

![Solution space of two linear equalitions](./book_figures/solution_space-of_2linear_eq.png)

Similarly, for three variables, each linear equation determines a plane in three-dimensional
space. When we intersect these planes, i.e., satisfy all linear equations at
the same time, we can obtain a solution set that is a plane, a line, a point
or empty (when the planes have no common intersection).


## Matrices

Matrices play a central role in linear algebra. They can be used to com-
pactly represent systems of linear equations, but they also represent linear functions (linear mappings).

With m, n ∈ N a real-valued (m, n) matrix A is
an m·n-tuple of elements aij , i = 1, . . . , m, j = 1, . . . , n, which is ordered
according to a rectangular scheme consisting of m rows and n columns:

![Matrix](./book_figures/matrix.png)

By convention (1, n)-matrices are called rows and (m, 1)-matrices are called columns. These special matrices are also called row/column vectors.

**Note:** Read page 28-32 of Mathematics of Machine Learning book. for matrix addition and multiplication, determinant(A), trace(A), Identity(A), Inverse(A), Transpose(A), Symmetric matrix, and matrix associatvity and distributivity properties. 

A matrix is not a grid of numbers - it's a function. `Ax` means take the vector `x` and transform it (rotate it, stretch it, shear it, project it) into a different space. 

Fun fact: Every layer of a neural network does this (linear part first, nonlinearity second). Every embedding lookup, every attention projection - it's this operation, over and over, at scale. 

![A matrix transforms the whole space](./figures/linear_transformation.png)

The left panel is the input space - the grid and the two basis vectors `e1 = (1,0)`, `e2 = (0,1)`. The right panel is what happens after applying:

```
A = [[1.5, 0.5],
     [0.3, 1.2]]
```

Notice the grid lines are still straight and evenly spaced (that's what makes it *linear* - it preserves lines and the origin) but the whole space has been sheared and stretched. `A`'s columns are literally where `e1` and `e2` land - that's the fastest way to read off what a matrix "does" just by looking at it.

A compact way of formulating systems of linear equations is `Ax = b` for a 2×2 system which is exactly two lines on a plane; the solution is where they cross.

![Two equations as two lines, meeting at the solution](./figures/linear_system_2d.png)

- **Unique solution** → lines/planes intersect at exactly one point → `A` is invertible (`det(A) ≠ 0`).
- **No solution** → lines are parallel, never meet → equations are inconsistent.
- **Infinite solutions** → lines are the same line → equations are redundant (rank-deficient).

Fun fact: In ML terms, this is exactly why `XᵀX` needs to be invertible for the closed-form linear regression solution `w = (XᵀX)⁻¹Xᵀy` to exist - if your features are collinear (redundant), `XᵀX` becomes singular, same failure mode as the "infinite solutions" case above.

## Solving system of linear equations

**Particular and General Solution:**
1. Find a particular solution to Ax = b.
2. Find all solutions to Ax = 0.
3. Combine the solutions from steps 1. and 2. to the general solution.

**Elementary Transformations**
Key to solving a system of linear equations are elementary transformations that keep the solution set the same, but that transform the equation system into a simpler form:

![](./book_figures/rref.png)

![](./book_figures/rref1.png)

**Remark (Reduced Row Echelon Form).** An equation system is in reduced reduced
row-echelon form (also: row-reduced echelon form or row canonical form) if row-echelon form
- It is in row-echelon form.
- Every pivot is 1.
- The pivot is the only nonzero entry in its column.

**Remark (Gaussian Elimination).** Gaussian elimination is an algorithm that elimination
performs elementary transformations to bring a system of linear equations
into reduced row-echelon form.

**Note:** Also we have **The Minus-1 Trick**.

![](./book_figures/minus1trick.png)

and **Moore-Penrose pseudo-inverse** algorithms.

![](./book_figures/Mppi.png)

## Vector spaces, span, and linear independence

- A **vector space** is a set of vectors closed under addition and scalar multiplication - informally, "everywhere you can get to by combining these vectors."

![](./book_figures/vectorspaces.png)
![](./book_figures/vsubpaces.png)

- The **span** of a set of vectors is every point reachable via their linear combinations.

- Vectors are **linearly independent** if none of them is a combination of the others - equivalently, the only way to combine them to get the zero vector is with all-zero coefficients.

![](./book_figures/lc.png)
![](./book_figures/lc1.png)

```python
# from the companion .py file
def is_linearly_independent(vectors: np.ndarray) -> bool:
    return bool(np.linalg.matrix_rank(vectors) == vectors.shape[0])
```

## Basis and rank

- A **basis** is a minimal set of vectors that spans the whole space - every vector in the space has a unique representation as a combination of basis vectors.
![](./book_figures/basis.png)

- The **rank** of a matrix is the dimension of its column space - how much of the output space it can actually reach. A matrix with rank less than its number of columns is "losing information": multiple different inputs can map to the same output.

![](./book_figures/rank.png)

```
full rank (independent rows):        shape=(3, 3), rank=3, independent=True
rank-deficient (row3 = row1 + row2): shape=(3, 3), rank=2, independent=False
duplicate rows:                      shape=(2, 2), rank=1, independent=False
```

**Why this matters in interviews:** "full rank" comes up constantly - feature collinearity breaking `XᵀX` invertibility, why you drop one dummy variable in one-hot encoding (to avoid a redundant, linearly dependent column), why weight matrices in a low-rank adapter (LoRA) are deliberately constrained to low rank to reduce trainable parameters.

## Linear mappings

Every linear map between finite-dimensional vector spaces can be represented as a matrix, once you fix a basis. This is *why* neural network layers are matrices: a fully-connected layer literally is the matrix representation of a linear map from the input space to the output space (before the nonlinearity is applied).

![](./book_figures/linearmapping.png)
![](./book_figures/homomorphism.png)

## Matrix representation of linear mappings

![](./book_figures/mrlm.png)
![](./book_figures/ltv.png)
