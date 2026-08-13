# 03 — Numerical Stability & Bayesian Foundations

**Companion notebook:** [`03-numerical-stability-bayesian-foundations.ipynb`](./03-numerical-stability-bayesian-foundations.ipynb)
**References:** the numerical-stability content is standard software/numerical-computing knowledge, not textbook material. The Bayesian material extends *Mathematics for Machine Learning* §6.1–6.4 (Statistics subchapter, notebook 01) and §8.3 (notebook 02).

---

## Numerical stability

Mathematically equivalent formulas can behave completely differently in floating point. Every failure below is real, reproduced exactly as shown, not just described.

**Softmax overflow.** `exp()` of a large number overflows a float64 long before the softmax's actual output (always between 0 and 1) would suggest any problem:

```python
logits = [1000., 1001., 1002.]
naive softmax:  [nan, nan, nan]      # silently NaN — no error, no obvious warning
stable softmax: [0.090, 0.245, 0.665]
```

The fix: subtract the max before exponentiating. `softmax(x)ᵢ = exp(xᵢ)/Σexp(xⱼ) = exp(xᵢ-c)/Σexp(xⱼ-c)` for *any* constant `c` — the `exp(c)` factor cancels top and bottom exactly. Choosing `c = max(x)` guarantees the largest exponent computed is `exp(0)=1`, so nothing overflows, and the math is provably identical.

**log-sum-exp** — the same trick, one level up. `log(Σexp(xᵢ))` shows up constantly (the normalizing constant of a softmax in log-space, the denominator of a mixture-model log-likelihood):

```python
naive:  inf
stable: 1002.4076059644444    # matches scipy.special.logsumexp exactly
```

**Catastrophic cancellation** — subtracting two nearly-equal large floats can destroy almost all meaningful precision:

```python
a, b = 1e16, 1.0
(a + b) - a   # mathematically 1.0, actually 0.0 — b was completely absorbed
```

float64 has ~16 significant decimal digits; `a+b` needed all of them just to represent `a`, leaving none for `b`.

**Comparing floats for equality** — the single most common float bug:

```python
0.1 + 0.2 == 0.3        # False (actual difference: 5.55e-17)
math.isclose(0.1 + 0.2, 0.3)   # True — the correct approach
```

**Where this shows up in ML code specifically:**
- Cross-entropy loss is `-log(softmax(logits))` — nearly every deep learning framework fuses this into a single `log_softmax`/`cross_entropy` op internally, specifically to apply the log-sum-exp trick once, rather than computing softmax then taking a log of a possibly-tiny or possibly-NaN number.
- Variance computed as `E[X²]-E[X]²` (the "naive" formula) can go slightly negative due to cancellation for low-variance data, then crash `sqrt()` — Welford's algorithm accumulates variance incrementally without ever computing that difference of two large, close numbers.
- Mixed-precision training (fp16/bf16) makes every issue above dramatically more likely, since those formats have far less precision than float64 — exactly why loss scaling exists in mixed-precision training recipes.

## Law of Large Numbers vs. Central Limit Theorem

Both describe repeated sampling and are constantly confused for each other — they answer different questions.

- **LLN**: the sample mean converges to the true mean as `n→∞`. Says nothing about the shape of the fluctuations along the way, only that they vanish eventually.
- **CLT** (Statistics subchapter, notebook 02): the *distribution* of the sample mean, appropriately rescaled, approaches a Gaussian as `n→∞` — a statement about shape, not just convergence.

![LLN convergence](./figures/lln_convergence.png)

LLN is *why* estimating a probability by observed frequency (e.g. MLE) is justified at all; CLT is *why* the resulting estimate's own uncertainty is well-approximated by a Gaussian, which is what confidence intervals rely on.

## Bayesian probability, as a sequential process

The Statistics subchapter covered Bayes' theorem and, separately, MAP as "MLE plus a prior." The genuinely Bayesian framing goes further: probability represents a *degree of belief* that gets updated as evidence arrives, one observation at a time — today's posterior becomes tomorrow's prior.

**Frequentist vs. Bayesian, in one line each.** Frequentist: a parameter has one true, fixed, unknown value; probability describes long-run frequency over repeated experiments — a confidence interval's "95%" describes the reliability of the *procedure* (Statistics notebook 03), not a probability statement about the specific parameter. Bayesian: a parameter is treated as a random variable with a distribution reflecting current belief — a **credible interval** ("there's a 95% probability the parameter lies in this range, given the data and the prior") is a directly different, and for many people more intuitive, statement — but it depends on the prior chosen, which a confidence interval does not.

**Sequential updating: Beta-Bernoulli, one flip at a time.** Estimating a coin's bias `p`. The Beta distribution is the conjugate prior for a Bernoulli/Binomial likelihood (mirrors the Gaussian-Gaussian conjugacy in the Statistics MLE/MAP notebook): starting from `Beta(α,β)`, observing `k` heads in `n` flips gives posterior `Beta(α+k, β+n-k)` — closed-form, no numerical integration, and each posterior becomes the prior for the next batch.

![Bayesian sequential updating](./figures/bayesian_updating.png)

Starting from a uniform `Beta(1,1)` prior against a true bias of `p=0.7`, the posterior mean after 50 flips lands at `0.692`. Two things worth having ready: the posterior after `n=0` observations *is* the prior — Bayesian updating isn't a separate machine bolted onto a prior, the prior is just the `n=0` case of the same update rule. And as `n` grows, the posterior narrows and the *choice* of prior matters less and less — with enough data, two different reasonable priors converge to nearly the same posterior. This is the practical resolution to "isn't the prior just subjective" objections: it matters most exactly when data is scarce, which is also exactly when MAP's shrinkage toward the prior (Statistics notebook 02) is most useful.

---

## Self-check before moving on

- [ ] I can explain why subtracting the max before exponentiating leaves softmax's value unchanged
- [ ] I can implement the log-sum-exp trick and explain what problem it solves
- [ ] I can give an example of catastrophic cancellation and why it's a "small answer, large intermediate values" problem
- [ ] I can state the difference between the Law of Large Numbers and the Central Limit Theorem in one sentence each
- [ ] I can state the Frequentist vs. Bayesian distinction, and the confidence-interval vs. credible-interval distinction
- [ ] I can explain Bayesian updating as sequential, with today's posterior becoming tomorrow's prior

This closes out Advanced Optimization & Numerical Methods, and with it, the full Mathematical Foundations chapter.
