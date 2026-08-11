# 02 - Chain Rule, Backpropagation & Automatic Differentiation ⭐

**The single highest-interview-yield notebook in the Calculus chapter.** "Derive backprop" or "explain how backprop works" is one of the most commonly asked ML questions, at every level of seniority.

**Companion notebook:** [`02-chain-rule-backprop-autodiff.ipynb`](./02-chain-rule-backprop-autodiff.ipynb) 
**Book reference:** 
*Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) [https://drive.google.com/file/d/1KsG9bntb0vD9QrUvHrlR2vqBQSr_V0J9/view]

- §5.5–5.8, pp.158–169 (Useful Identities for Computing Gradients, Backpropagation and Automatic Differentiation, Higher-Order Derivatives, Linearization and Multivariate Taylor Series)

## Backpropagation and Automatic Differentiation

In many machine learning applications, we find good model parameters
by performing gradient descent, which relies on the fact
that we can compute the gradient of a learning objective with respect
to the parameters of the model. For a given objective function, we can
obtain the gradient with respect to the model parameters using calculus
and applying the chain rule;

![](./book_figures/image10.png)
![](./book_figures/image11.png)
![](./book_figures/image12.png)
![](./book_figures/image13.png)
![](./book_figures/image14.png)

If `f(x) = h(g(x))`, the chain rule says `f'(x) = h'(g(x)) · g'(x)`. That's it. Backpropagation is exactly this rule, applied repeatedly through a **computational graph** - a graph where each node is an intermediate value, computed from its inputs.

Take a tiny example: `f = (x * y) + z`. Break it into a forward pass through two operations, `q = x*y` and `f = q+z`.

![Computational graph forward and backward pass](./figures/computational_graph.png)

Two passes:
- **Forward pass** (blue values): compute `q = x*y = -12`, then `f = q+z = -7`, left to right.
- **Backward pass** (red gradients): start from `df/df = 1` at the output, and multiply local derivatives backward through each node - `df/dq = 1` (addition just passes the gradient through unchanged), then `df/dx = df/dq · dq/dx = 1 · y = -4` and `df/dy = df/dq · dq/dy = 1 · x = 3`.

That's the whole algorithm. A neural network with millions of parameters runs exactly this process - forward pass computes the output and caches intermediate values, backward pass multiplies local gradients back through the graph, reusing those cached values so nothing gets recomputed. That reuse is the entire reason backprop is efficient (`O(n)` in the number of operations, instead of exponential if you tried to symbolically expand the whole chain rule at once).

## Automatic Differentiation

It turns out that backpropagation is a special case of a general technique
in numerical analysis called automatic differentiation. We can think of automatic
differentiation tomatic differentation as a set of techniques to numerically (in contrast to
symbolically) evaluate the exact (up to machine precision) gradient of a
function by working with intermediate variables and applying the chain
rule. Automatic differentiation applies a series of elementary arithmetic 
operations, e.g., addition and multiplication and elementary functions,
e.g., sin, cos, exp, log. By applying the chain rule to these operations, the
gradient of quite complicated functions can be computed automatically.
Automatic differentiation applies to general computer programs and has
forward and reverse modes. Baydin et al. (2018) give a great overview of
automatic differentiation in machine learning.

**This is also precisely what "automatic differentiation" (autodiff) in PyTorch/TensorFlow does** - it's not symbolic differentiation (manipulating equations) and it's not numerical differentiation (finite differences, as used for gradient-checking). It's this exact graph-based chain-rule bookkeeping, done automatically so you never have to hand-derive it.





## The gradient of softmax + cross-entropy - the other classic derivation

This is the second-most-common "derive this by hand" question after backprop itself. The clean result:

```
∂L/∂logits = softmax(logits) - one_hot(y)
```

```python
def softmax(logits):
    z = logits - logits.max(axis=1, keepdims=True)
    exp = np.exp(z)
    return exp / exp.sum(axis=1, keepdims=True)

def softmax_xent_loss_and_grad(logits, y_true):
    n, c = logits.shape
    probs = softmax(logits)
    loss = np.mean(-np.log(probs[np.arange(n), y_true] + 1e-12))
    one_hot = np.zeros_like(probs)
    one_hot[np.arange(n), y_true] = 1.0
    grad = (probs - one_hot) / n
    return loss, grad
```

The companion notebook gradient-checks this against finite differences - max difference `~1e-12`, confirming the closed form.

## Vanishing gradients - where calculus explains a real deep learning failure mode

The chain rule multiplies local derivatives together across many layers. If those local derivatives are small, the product shrinks exponentially with depth - this is the **vanishing gradient problem**, and it's a direct, mechanical consequence of the chain rule, not some mysterious training instability.

![Sigmoid and its derivative, showing vanishing gradients](./figures/vanishing_gradient.png)

Sigmoid's derivative maxes out at **0.25**, and gets close to **0** whenever the input is far from zero (the function "saturates"). In a deep network, backprop multiplies one of these small numbers per layer - 10 layers of sigmoid activations could multiply the gradient by `0.25^10 ≈ 1e-6`, effectively killing the learning signal by the time it reaches early layers.

This is precisely *why* ReLU (`max(0, x)`, derivative is either 0 or exactly 1 - no shrinking in the active region) displaced sigmoid/tanh as the default hidden-layer activation, and why architectural tricks like residual connections (skip connections) and batch/layer normalization exist - they're explicit interventions to keep gradients from vanishing (or exploding) as they backpropagate through depth.

## Beyond first derivatives: Taylor series (brief)

Higher-order derivatives and multivariate Taylor expansion (MML §5.7–5.8) let you locally approximate any smooth function with a polynomial - the second-order (quadratic) term involves the **Hessian**, which is what second-order optimizers like Newton's method use, and what underlies the "curvature" intuition behind why some loss landscapes are easy or hard to optimize (flat regions, sharp valleys, saddle points). We'll come back to this directly in the next file, on optimization.


