# 01 - Differentiation, Gradients & Jacobians

**Companion notebook:** [`01-differentiation-gradients-jacobians.ipynb`](./01-differentiation-gradients-jacobians.ipynb)

**Book reference:** 
*Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) [https://drive.google.com/file/d/1KsG9bntb0vD9QrUvHrlR2vqBQSr_V0J9/view] 
- §5.1–5.4, pp.141–157 (Differentiation of Univariate Functions, Partial Differentiation and Gradients, Gradients of Vector-Valued Functions, Gradients of Matrices)

## Differentiation of a univariate function

The derivative of a function describes the function's instantaneous rate of change at a certain point - it gives us the slope of the line tangent to the function's graph at that point.

`f'(x)` is the slope of the tangent line to `f` at `x` - the limit of the slope of a secant line as the two points it connects get infinitely close:

```
f'(x) = lim_{h->0} [f(x+h) - f(x)] / h
```

![](./book_figures/image.png)

Everything else in this chapter - gradients, Jacobians, backprop - is this same idea, generalized to more inputs and outputs.

![](./book_figures/image1.png)

![Derivative as the slope of the tangent line](./figures/tangent_line.png)

## Derivative Rules

![](./book_figures/image3.png)

## Taylor Series

A Taylor series expansion writes a smooth function as an infinite sum of terms. Each term uses the function's derivatives at a single point. The general formula expands a function f(x) around a point a:

![](./book_figures/image2.png)


Maclaurin Series: A special case where the expansion point is a = 0.
Derivatives: Each n-th term involves the n-th derivative f^{(n)}(a) divided by the factorial n!.
Approximation: Stopping the series early creates a Taylor polynomial that closely estimates the function near a.

## Partial derivatives and the gradient

![](./book_figures/image4.png)

For a function of several variables, `f(x, y)`, the **partial derivative** `∂f/∂x` treats `y` as fixed and differentiates with respect to `x` alone. Stack all the partial derivatives into a vector and you get the **gradient**:

```
∇f = [∂f/∂x1, ∂f/∂x2, ..., ∂f/∂xn]
```

![](./book_figures/image5.png)
![](./book_figures/image6.png)

The gradient has a crucial geometric property: **it always points in the direction of steepest ascent, and it's always perpendicular to the contour lines of `f`.** That single fact is the entire justification for gradient *descent* - step in the direction opposite the gradient to decrease the function fastest.

![Gradient field, perpendicular to contour lines](./figures/gradient_field.png)

Every red arrow points "uphill," away from the center (the minimum), and every arrow crosses its local contour line at a right angle. Gradient descent just walks backward along these arrows.

## Gradient of Vector-Valued Functions
![](./book_figures/image7.png)
![](./book_figures/image8.png)
![](./book_figures/image9.png)

## The gradient of MSE loss, worked by hand

For linear regression, `L(w) = (1/n)‖Xw - y‖²`. Using the chain rule (more on this in the next notebook):

```
∂L/∂w = (2/n) · Xᵀ(Xw - y)
```

The companion notebook verifies this analytic gradient against a **numerical gradient** (finite differences) - this "gradient check" technique is exactly what you'd do to sanity-check a backprop implementation, and it's a legitimate thing to mention in an interview if asked how you'd debug a suspicious gradient:

```python
def numerical_gradient(f, w, eps=1e-5):
    grad = np.zeros_like(w)
    for i in range(len(w)):
        w_plus, w_minus = w.copy(), w.copy()
        w_plus[i] += eps
        w_minus[i] -= eps
        grad[i] = (f(w_plus) - f(w_minus)) / (2 * eps)
    return grad
```

Running it: the analytic and numerical gradients agree to about `1e-11` - confirming the hand-derived formula is correct.

## Jacobians: the gradient's generalization to vector-valued functions

A **gradient** is defined for a scalar-valued function (one output). A **Jacobian** generalizes this to a *vector*-valued function `f: Rⁿ → Rᵐ` - it's the `m × n` matrix of all partial derivatives, one row per output, one column per input:

```
J[i, j] = ∂f_i / ∂x_j
```

A gradient is just a Jacobian with a single row (`m = 1`). This distinction matters because every layer of a neural network is a vector-valued function (many inputs, many outputs), so backpropagation is really chaining Jacobians together, not just chaining scalar derivatives - that's the subject of the next notebook.
