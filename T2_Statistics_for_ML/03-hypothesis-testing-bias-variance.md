# 03 - Hypothesis Testing, Confidence Intervals & Bias-Variance

**Companion notebook:** [`03-hypothesis-testing-bias-variance.ipynb`](./03-hypothesis-testing-bias-variance.ipynb)
**Book references:** *Practical Statistics for Data Scientists* (Bruce & Bruce) (https://drive.google.com/file/d/1LGisFRLBa8YojQ7EOd4rPUWeicjsN8ys/view) - Ch. 2 "Data and Sampling Distributions" (The Bootstrap, Confidence Intervals), Ch. 3 "Statistical Experiments and Significance Testing" (Hypothesis Tests, Statistical Significance and P-Values), Ch. 6 "Statistical Machine Learning" (Bias-Variance Tradeoff, in the K-Nearest Neighbors section)

---

## What a p-value actually means

A **p-value** is the probability, *assuming the null hypothesis is true*, of observing a test statistic at least as extreme as the one actually observed. That's it. It is emphatically **not** "the probability the null hypothesis is true" - that's the single most common misinterpretation, and it's worth internalizing why: the p-value is computed *under the assumption* the null is true, so by construction it can't also tell you how likely that assumption is.

![p-value visualization](./Companion_Notebooks/figures/pvalue.png)

The shaded red area (both tails, for a two-sided test) *is* the p-value - literally the probability mass at least as far from the center as what we observed, computed entirely under the null distribution. A small p-value means "this would be a surprising thing to see if there were really no effect," which is evidence *against* the null - but it's not a direct statement about how likely the null is to be true, and it says nothing about the size or practical importance of the effect (statistical significance ≠ practical significance, especially with large samples where even tiny, meaningless effects become "significant").

## Confidence intervals via the bootstrap

A common misreading of a 95% confidence interval is "there's a 95% probability the true value is in this specific interval." The more careful statement: **if you repeated this sampling-and-interval-construction procedure many times, about 95% of the resulting intervals would contain the true value.** The interval itself either does or doesn't contain the truth - the 95% describes the reliability of the *procedure*, not a probability about this one interval.

The **bootstrap** builds such an interval without assuming any particular distribution: resample the data with replacement many times, compute the statistic on each resample, and take percentiles of that empirical distribution.

![Bootstrap confidence interval](./Companion_Notebooks/figures/bootstrap_ci.png)

Note the data itself was drawn from a skewed Gamma distribution, not a Gaussian - the bootstrap doesn't care. That's its main practical advantage over classical formula-based confidence intervals, which often assume normality: it works directly off the empirical resampling distribution, whatever shape the underlying data has.

## The bias-variance tradeoff

Expected prediction error decomposes into three pieces:

```
Expected Error = Bias² + Variance + Irreducible Noise
```

- **Bias** - error from an overly simplistic model that can't capture the true pattern (underfitting).
- **Variance** - error from a model that's too sensitive to the specific training sample, capturing noise instead of signal (overfitting).
- **Irreducible noise** - error that no model can remove; inherent randomness in the data-generating process.

![Bias-variance tradeoff curve](./Companion_Notebooks/figures/bias_variance_curve.png)

Two intuitive examples worth having ready:
- **K in KNN:** small K (e.g., K=1) means low bias but high variance - the prediction chases every quirk of the nearest single point. Large K oversmooths - high bias, low variance.
- **Tree depth:** a shallow tree underfits (high bias); an unconstrained deep tree memorizes training data (high variance) - which is exactly why techniques like pruning, minimum leaf size, and ensembling (random forests average away variance across many high-variance trees) exist.

This is also the same curve underlying the classic **train/validation error vs. model complexity** diagnostic plot: training error keeps falling as complexity increases (the model can always fit the training data better), while validation error is U-shaped - falling as bias drops, then rising again as variance takes over. Reading that gap between train and validation error is the standard practical diagnostic for "is my model underfitting or overfitting."

---

## Self-check before moving on

- [ ] I can state precisely what a p-value means, and identify the most common misinterpretation
- [ ] I can explain what "95% confidence" actually refers to (reliability of the procedure, not probability of this one interval)
- [ ] I can explain how the bootstrap constructs a confidence interval without a normality assumption
- [ ] I can state the bias-variance decomposition of expected error and give two concrete examples
- [ ] I can connect the bias-variance curve to the practical train/validation error diagnostic

This closes out the Statistics subchapter - and the full Math Foundations chapter (Linear Algebra → Calculus → Statistics). Next up in the 6-month plan: **Month 2, Classical ML** (feature engineering, tree ensembles, SVMs, clustering).
