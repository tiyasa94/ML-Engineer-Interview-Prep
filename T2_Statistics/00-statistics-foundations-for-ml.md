# 04 - Statistics Foundations for ML Engineers

**Companion notebooks:** none (theory + interview reference; pairs with 01-03)
**Covers:** population vs. sample, estimators & their properties, sampling distributions & standard error, law of large numbers, descriptive statistics (mean/variance/skew/kurtosis), covariance & correlation (Pearson vs. Spearman), experimental design & A/B testing, statistical power & multiple-testing correction, common distributions cheat sheet, outliers & robust statistics
**Yield:** ⭐ (fills the gap between "probability theory" in 01-03 and the applied statistics that shows up constantly in ML interviews and day-to-day model evaluation)

---

## Why this notebook exists

Notebooks 01-03 cover probability *theory* (Bayes, distributions, MLE/MAP, hypothesis testing, bias-variance). But there's a layer of foundational statistics that sits underneath all of that - the vocabulary and tools you use to go from raw data to the numbers those theories operate on. This is the stuff that gets casually assumed in interviews ("what's the difference between population and sample variance?") and quietly misused in practice (using Pearson correlation on a monotonic-but-nonlinear relationship, running an A/B test with no power analysis, treating a sample statistic as if it had zero uncertainty).

---

## 1. Population vs. sample

- **Population** - the entire set of items/events you care about (all users, all possible dice rolls, every transaction that will ever occur). Usually unobservable in full.
- **Sample** - a finite subset of the population that you actually observe and compute statistics from.
- **Parameter** - a numerical property of the *population* (population mean `μ`, population variance `σ²`). Fixed but usually unknown.
- **Statistic** - a numerical property of a *sample* (sample mean `x̄`, sample variance `s²`). Computed from data, therefore itself a random variable - it changes if you draw a different sample.

This distinction is the reason confidence intervals and standard errors exist at all: a statistic is an estimate of a parameter, and estimates have variability that the raw number alone doesn't show.

**Sample mean:**
```
x̄ = (1/n) Σ xᵢ
```

**Sample variance (unbiased, "Bessel's correction"):**
```
s² = (1/(n-1)) Σ (xᵢ - x̄)²
```

Why `n-1` and not `n`? Because `x̄` is itself computed from the data, it's slightly closer to the sample points than the true `μ` would be, so squared deviations from `x̄` systematically *understate* the true variance. Dividing by `n-1` instead of `n` corrects this bias exactly (this is a direct instance of the estimator-bias idea in §2). With `n` in the denominator you get the *maximum likelihood* estimator of variance, which is biased downward for finite `n` but converges to the right answer as `n → ∞`.

---

## 2. Estimators: bias, consistency, efficiency

An **estimator** is a rule/formula for producing a statistic from a sample (e.g., "average the sample" is the estimator for the population mean). Three properties matter:

- **Unbiased:** `E[θ̂] = θ` - on average, across infinitely many samples, the estimator hits the true parameter. (Sample mean is unbiased for `μ`; sample variance with `n-1` is unbiased for `σ²`; sample variance with `n` is biased.)
- **Consistent:** `θ̂ → θ` in probability as `n → ∞` - the estimator converges to the truth with enough data, even if it's biased for small `n`. (The `n`-denominator variance estimator is biased but still consistent.)
- **Efficient:** among unbiased estimators, the one with the *lowest variance*. Efficiency is a comparison, not a yes/no property of a single estimator - "the sample mean is the most efficient unbiased estimator of `μ` for normally distributed data" is a meaningful statement.

**Bias-variance of an estimator vs. bias-variance of a model (notebook 03):** same underlying math (`MSE = Bias² + Variance`), applied to two different things - here it's error in a *parameter estimate*, in 03 it's error in a *model's predictions*. Worth explicitly connecting these when asked in an interview, since people often think they're unrelated concepts.

**Standard error (SE)** - the standard deviation *of a statistic's sampling distribution* (not of the data itself). For the sample mean:
```
SE(x̄) = σ / √n        (population σ known)
SE(x̄) ≈ s / √n        (population σ unknown, estimated from sample)
```
This is the number that shrinks as you collect more data, and it's what confidence intervals (notebook 03) are built from: `x̄ ± z·SE`.

---

## 3. Law of Large Numbers vs. Central Limit Theorem

Easy to conflate - they answer different questions about the same setup (averaging i.i.d. random variables):

| | Law of Large Numbers (LLN) | Central Limit Theorem (CLT, notebook 02) |
|---|---|---|
| Claims | `x̄ → μ` as `n → ∞` | The *distribution* of `x̄` (properly scaled) `→ Normal`, regardless of the original distribution's shape |
| Answers | "Does the sample mean converge to the true mean?" | "What does the sampling distribution of the mean look like, and how wide is it?" |
| Gives you | Convergence (a point) | A distribution shape + spread (`N(μ, σ²/n)`) - which is what lets you build confidence intervals and p-values |

LLN tells you *that* you're converging; CLT tells you *how fast and in what shape*, which is the practically useful part.

---

## 4. Descriptive statistics beyond mean/variance

- **Skewness** - measures asymmetry. Positive skew = long right tail (e.g., income, most transaction-amount distributions); negative skew = long left tail. Relevant in ML because skewed target/feature distributions often benefit from a log or Box-Cox transform before feeding into models that assume roughly symmetric errors (e.g., linear regression).
- **Kurtosis** - measures tail heaviness relative to a normal distribution. High kurtosis ("heavy tails") means outliers are more probable than a Gaussian would predict - relevant for anomaly detection and for knowing when Gaussian-based assumptions (e.g., in some anomaly-scoring or control-chart methods) will underestimate the frequency of extreme events.
- **Median vs. mean** - median is robust to outliers and skew; mean is not. Report both when a distribution looks skewed (e.g., "median session length" is often more representative than "mean session length" because a few very long sessions can drag the mean up).
- **IQR (interquartile range)** - `Q3 - Q1`, robust spread measure, basis of the standard "1.5×IQR" outlier rule.

---

## 5. Covariance and correlation

**Covariance** - direction of joint variation between two variables:
```
Cov(X, Y) = E[(X - E[X])(Y - E[Y])]
```
Sign tells you direction; magnitude is not interpretable on its own because it depends on the scales of `X` and `Y`.

**Pearson correlation** - covariance normalized to `[-1, 1]`:
```
ρ = Cov(X, Y) / (σ_X · σ_Y)
```
Measures strength of *linear* association only (this is exactly the gap exploited in notebook 01's independence-vs-correlation example).

**Spearman correlation** - Pearson correlation computed on the *ranks* of the data instead of raw values. Captures any *monotonic* relationship (linear or not), not just linear ones, and is robust to outliers since ranking compresses extreme values. Use Spearman when you suspect a nonlinear-but-monotonic relationship, or when the data has heavy outliers that would distort Pearson.

**Covariance matrix** - for a vector of `p` variables, the `p×p` matrix `Σ` where `Σᵢⱼ = Cov(Xᵢ, Xⱼ)`. Diagonal entries are variances. Foundational for PCA (eigendecomposition of the covariance matrix), Gaussian distributions in more than one dimension, and Mahalanobis distance.

**Simpson's paradox** (worth knowing alongside correlation): a trend that appears in several groups of data can reverse or disappear when the groups are combined, usually because of a lurking/confounding variable. Classic caution against reading causal or even purely correlational conclusions off aggregated data without checking subgroups.

---

## 6. Experimental design & A/B testing

This is where hypothesis testing (notebook 03) meets practical constraints.

- **Randomization** - the reason A/B tests can support causal claims: randomly assigning units to control/treatment makes the two groups statistically equivalent *in expectation* on every variable (measured or not), so any systematic difference in outcome is attributable to the treatment.
- **Confounding variable** - a variable that influences both the "treatment" and the outcome, creating a spurious association if not controlled for (classic example: ice cream sales and drowning deaths are correlated because both are driven by hot weather, not by each other).
- **Statistical power** - `P(reject H₀ | H₀ is false)`, i.e., the probability of correctly detecting a real effect. Complements the significance level `α` (notebook 03's Type I error rate); the analogous **Type II error** `β` is failing to detect a real effect, and `power = 1 - β`. Power depends on effect size, sample size, variance, and `α` - this is why you run a **power analysis** *before* an A/B test to determine the minimum sample size needed to reliably detect an effect of a given size, rather than checking significance after collecting an arbitrary amount of data.
- **Minimum detectable effect (MDE)** - the smallest true effect size a test is powered to reliably detect given a fixed sample size; there's a direct tradeoff between sample size, MDE, and power.
- **Multiple testing / p-hacking** - running many hypothesis tests (e.g., checking a dozen metrics in one A/B test, or repeatedly peeking at results as data comes in) inflates the overall false-positive rate above the nominal `α`, since each test carries its own chance of a false positive purely by luck. Standard corrections:
  - **Bonferroni correction:** use `α/m` per test instead of `α`, for `m` tests - simple and conservative (increases Type II error, i.e. lowers power).
  - **Benjamini-Hochberg (FDR control):** controls the *expected proportion* of false positives among rejected hypotheses rather than the family-wise error rate - less conservative than Bonferroni, more common in large-scale testing (e.g., many metrics, many genes, many features).
- **Novelty/primacy effects & network effects** - practical A/B testing pitfalls: users may react to *any* change initially (novelty) or take time to adjust (primacy), and treatment can leak into control when users interact with each other (network effects) - both violate the clean "independent, stable units" assumption behind standard A/B test statistics.

---

## 7. Common distributions - quick reference

| Distribution | Type | Typical ML/stats use |
|---|---|---|
| Bernoulli | discrete | single binary outcome (one coin flip, one click) |
| Binomial | discrete | count of successes in `n` independent Bernoulli trials |
| Poisson | discrete | count of rare events in a fixed interval (requests/sec, defects/batch) |
| Geometric | discrete | number of trials until first success |
| Uniform | continuous | "no information" prior, random initialization |
| Normal (Gaussian) | continuous | CLT limit, noise models, weight initialization, many statistical tests' assumptions |
| Exponential | continuous | time between independent events (memoryless), survival analysis |
| Beta | continuous | models a probability itself; conjugate prior for Bernoulli/Binomial (Bayesian A/B testing) |
| Gamma | continuous | generalizes Exponential; waiting times, conjugate prior for Poisson rate |
| Student's t | continuous | like Normal but heavier tails; used for small-sample confidence intervals/tests (notebook 03) when population variance is unknown |
| Chi-squared | continuous | sum of squared standard normals; goodness-of-fit tests, variance-related tests |

Rule of thumb for interviews: know *what generates* each distribution (not just its formula) - e.g., Poisson is the limit of Binomial as `n → ∞`, `p → 0`, with `np` held constant, which is exactly why it models rare-event counts.

---

## 8. Outliers & robust statistics

- **Z-score method** - flag points beyond `±3` standard deviations from the mean; assumes roughly-normal data and is itself distorted by extreme outliers (since they inflate the mean/std used to compute it).
- **IQR method** - flag points beyond `Q1 - 1.5·IQR` or `Q3 + 1.5·IQR`; robust because quartiles aren't pulled around by extreme values the way mean/std are.
- **Robust statistics** - median, IQR, median absolute deviation (MAD), and Spearman correlation are all preferred over mean/std/Pearson when outliers or skew are present, because they don't let a handful of extreme points dominate the estimate.
- Practical ML note: whether to remove, cap (winsorize), transform (log), or keep outliers is a modeling decision, not just a statistical one - a "outlier" transaction might be exactly the fraud case you're trying to detect.

---

## Self-check before moving on

- [ ] I can explain why sample variance divides by `n-1`, not `n`, and connect this to estimator bias
- [ ] I can state the difference between bias, consistency, and efficiency of an estimator
- [ ] I can explain what LLN guarantees vs. what CLT guarantees, in one sentence each
- [ ] I can explain when to use Spearman over Pearson correlation, with an example
- [ ] I can explain statistical power, Type II error, and why power analysis should happen *before* an A/B test
- [ ] I can explain the difference between Bonferroni and Benjamini-Hochberg corrections and when each is used
- [ ] I can match at least 5 distributions in the table above to a real generative scenario, not just their formulas

---

## Interview Q&A

**Q1: What's the difference between a population parameter and a sample statistic?**
A parameter (e.g., `μ`, `σ²`) describes the full population and is fixed but typically unknown. A statistic (e.g., `x̄`, `s²`) is computed from an observed sample and is itself a random variable - it would come out differently if you drew a different sample. Statistics are used to *estimate* parameters, and that estimation carries uncertainty (captured by the standard error).

**Q2: Why does sample variance use `n-1` in the denominator instead of `n`?**
Because the sample mean `x̄` used in the variance formula is itself estimated from the same data, the squared deviations `(xᵢ - x̄)²` are on average slightly smaller than the true squared deviations from the unknown `μ` would be - `x̄` is, by construction, the point that minimizes the sum of squared deviations for *this* sample. Dividing by `n-1` (Bessel's correction) exactly corrects for this so that `E[s²] = σ²`. Dividing by `n` gives the MLE of variance, which is biased low for finite samples but still consistent.

**Q3: Is an unbiased estimator always better than a biased one?**
No. What matters practically is mean squared error, `MSE = Bias² + Variance`. A biased estimator with much lower variance can have lower overall MSE than an unbiased one - this is the exact same tradeoff as the model bias-variance tradeoff in notebook 03, just applied to a parameter estimate instead of a prediction. Regularized estimators (ridge regression, MAP estimates) are classic examples: they're deliberately biased in exchange for lower variance.

**Q4: What does the Central Limit Theorem actually say, and why does it matter for A/B testing?**
It says that the sampling distribution of the mean of i.i.d. random variables approaches a Normal distribution as `n` grows, *regardless of the shape of the original distribution*, with mean `μ` and variance `σ²/n`. This is why A/B test statistics on things like conversion rate or average order value can be treated as approximately normal for large enough samples even when the underlying per-user data isn't normal at all - it justifies using z-tests/t-tests and normal-based confidence intervals.

**Q5: Explain the difference between Pearson and Spearman correlation, and give an example where they'd disagree.**
Pearson measures linear association between raw values; Spearman measures monotonic association by correlating the *ranks*. They disagree whenever a relationship is monotonic but non-linear - e.g., `Y = X³`: as `X` increases, `Y` always increases (perfect monotonic relationship, Spearman ≈ 1), but the relationship curves, so Pearson will be less than 1 (or even close to 0 if the curve is symmetric around a point). Spearman is also more robust to outliers since ranking compresses extreme values.

**Q6: What is statistical power, and how does it relate to sample size?**
Power is `P(reject H₀ | H₀ is false)` - the probability of correctly detecting a real effect. Power increases with larger sample size, larger true effect size, lower variance in the data, and a looser (higher) significance threshold `α`. Practically, before running an A/B test you'd do a power analysis: fix the effect size you care about detecting, `α`, and desired power (commonly 80%), and solve for the required sample size - so you don't run an underpowered test that fails to detect a real effect, or an unnecessarily long test that wastes traffic.

**Q7: Why is running many simultaneous significance tests a problem, and how do you fix it?**
Each individual test carries its own false-positive probability `α` (e.g., 5%), purely by chance. Run 20 independent tests at `α = 0.05` and you'd expect roughly one false positive by chance alone even if nothing is actually true, so the *overall* chance of at least one false positive is much higher than 5%. Fixes: Bonferroni correction (use `α/m` per test - controls family-wise error rate, conservative) or Benjamini-Hochberg procedure (controls the expected false discovery rate among rejected tests - less conservative, more common when testing many metrics/features).

**Q8: What's a confounding variable, and how does randomization address it?**
A confounder influences both the "treatment" assignment and the outcome, creating an association between them that isn't causal (e.g., a marketing campaign that happens to run during a holiday season - sales lift could be from the campaign, the holiday, or both, and they're now entangled). Randomizing which units get the treatment breaks this link: since assignment no longer depends on any other variable (measured or not), the treatment and control groups become statistically equivalent in expectation on every dimension, so any observed outcome difference can be attributed to the treatment itself.

**Q9: What's Simpson's paradox, and why should it make you cautious with aggregated data?**
It's when a trend present in each of several subgroups reverses or vanishes when the subgroups are pooled together, typically because of an unaccounted-for confounding variable that differs in size across the subgroups. It's a caution against trusting an aggregate correlation or A/B test result without checking whether it holds consistently across meaningful segments (e.g., by device type, by user cohort) - an overall "win" can hide a loss in an important segment, or vice versa.

**Q10: Give a real generative story for the Poisson distribution and explain why it's not just "a discrete distribution."**
Poisson models the count of independent, rare events occurring in a fixed interval of time or space, given a constant average rate `λ` (e.g., number of server errors per minute, number of customer arrivals per hour). It arises as the limiting case of a Binomial distribution as the number of trials `n → ∞` and success probability `p → 0` while `np = λ` stays fixed - i.e., "many chances, each individually very unlikely." Knowing this generative link (rather than just the formula `P(k) = λᵏe⁻λ/k!`) is what lets you recognize when Poisson is the *right* model for a given count-data problem versus, say, Binomial or Negative Binomial (which handles overdispersion Poisson can't).

---

Next: return to [`01-probability-foundations-bayes.md`](./01-probability-foundations-bayes.md) for the Bayes' theorem track, or continue to `02-distributions-mle-map.md` ⭐ and `03-hypothesis-testing-bias-variance.md` if not already covered.
