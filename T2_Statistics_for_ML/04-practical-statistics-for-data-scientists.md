# Practical Statistics for Data Scientists

**Companion notebook:** [`practical-statistics-for-data-scientists.ipynb`](./practical-statistics-for-data-scientists.ipynb) - bootstrap, permutation test, confidence interval, power analysis, and confusion matrix/ROC/AUC, all implemented from scratch.


**Book reference:** **Practical Statistics for Data Scientists** (Bruce & Bruce, O'Reilly) [https://drive.google.com/file/d/1LGisFRLBa8YojQ7EOd4rPUWeicjsN8ys/view]

- Ch.1 EDA (pp.11–62)
- Ch.2 Data & Sampling Distributions (pp.63–105)
- Ch.3 Statistical Experiments (pp.106–161)
- Ch.4 Regression (pp.162–219)
- Ch.5 Classification (pp.220–264)
- Ch.6 Statistical Machine Learning (pp.265–309) 
- Ch.7 Unsupervised Learning (pp.310–)

# Part 1 - Exploratory Data Analysis (PSDS Ch.1, pp.11–62)

## 1.1 Estimates of Location (pp.19–25)

The mean is not always the right "center." PSDS's framing, worth having exactly:

- **Mean**: `x̄ = (1/n)Σxᵢ` - sensitive to outliers; one extreme value can move it arbitrarily far.
- **Weighted mean**: `Σwᵢxᵢ / Σwᵢ` - used when some observations are inherently more reliable or more representative than others (e.g. weighting by inverse variance, or by how representative a demographic group is of the population).
- **Median**: the middle value of sorted data - a **robust** estimate, meaning it isn't sensitive to extreme values at all: moving the largest value in a dataset to infinity does not move the median.
- **Trimmed mean**: drop the smallest `p%` and largest `p%` of values, then take the mean of what remains - a tunable middle ground between the mean's sensitivity to outliers and the median's complete disregard of magnitude. *(p.21, 23–25)*
- **Weighted median**: the median analog of the weighted mean.

**Interview framing worth having ready**: median (and trimmed mean) for skewed, outlier-prone data - income, home prices, latency/response-time distributions - where a few extreme values would otherwise dominate the mean; plain mean for symmetric, well-behaved data where every observation should count equally.

## 1.2 Estimates of Variability (pp.26–31)

- **Variance / standard deviation**: `s² = Σ(xᵢ-x̄)²/(n-1)` - note the `n-1`, not `n` (**Bessel's correction**): dividing by `n` would systematically *underestimate* the true population variance, because the sample mean is itself fit to the data and is slightly closer to the sample points than the true population mean would be, on average - `n-1` corrects this bias. Standard deviation (`√variance`) is preferred for interpretation specifically because it's in the **same units as the original data**, unlike variance.
- **Mean absolute deviation**: `(1/n)Σ|xᵢ-x̄|` - a more robust, less outlier-sensitive alternative to variance, but still uses the mean as its center.
- **Median absolute deviation (MAD)**: `median(|xᵢ - median(x)|)`, often scaled by a constant (**1.4826**) so that, under normality, it's directly comparable to the standard deviation - this is the **most robust** common variability measure, since it uses the median at both steps and is essentially unaffected by outliers.
- **Range and percentiles**; **IQR** (interquartile range, P75-P25) - the boxplot's outlier convention (a point beyond `1.5×IQR` from the nearest quartile is flagged) is drawn from here.

## 1.3 Exploring the Data Distribution (pp.32–40)

Boxplots, frequency tables, histograms, density estimates - the standard visual-EDA toolkit, all downstream of the location/variability estimates above.

---

# Part 2 - Data & Sampling Distributions (PSDS Ch.2, pp.63–105)

## 2.1 Random Sampling and Sample Bias (pp.64–74)

A sample must be **representative** of the population being studied - random sampling is the mechanism that gives this guarantee in expectation. **Selection bias** is the general failure mode: any systematic (non-random) mechanism that influences which observations end up in the sample distorts every downstream statistic. Named forms worth having ready in an interview: **self-selection bias** (respondents opt in, e.g. survey responders differ systematically from non-responders), **survivorship bias** (only observing units that "survived" some filtering process - the classic WWII returning-bomber armor example), and **regression to the mean** (an extreme observation is likely to be followed by a less extreme one, purely from randomness, easily mistaken for a real effect).

## 2.2 Sampling Distribution of a Statistic (pp.75–78)

**The distinction PSDS emphasizes most heavily, and the one most worth being crisp about**: the **data distribution** is the distribution of individual values in the raw data; the **sampling distribution** is the distribution of a *statistic* (e.g. the mean) computed across *many different samples* drawn from that data. These are not the same object, and conflating them is a common source of confused reasoning about confidence intervals.

**Standard error**: `SE = s/√n` - the sampling distribution of the mean narrows proportionally to `1/√n` as sample size grows; this square-root relationship is exactly why *quadrupling* a sample size only *halves* the standard error, not quarters it - a genuinely useful, concrete fact for "how many more samples would meaningfully tighten this" reasoning.

## 2.3 The Bootstrap (pp.79–85)

**The core practical technique of this chapter.** Resample the observed data **with replacement**, many times (typically 1,000–10,000 resamples), recomputing the statistic of interest on each resample - the resulting distribution of that statistic *is* an empirical estimate of its sampling distribution, without needing to assume any particular parametric form (normal, etc.) for the underlying data.

```
Bootstrap algorithm:
1. Draw a resample of size n, WITH replacement, from the original n observations.
2. Compute the statistic of interest on that resample.
3. Repeat steps 1-2 many times (e.g. 10,000).
4. The resulting collection of statistic values IS the empirical sampling distribution.
```

**Limitation worth stating honestly**: the bootstrap cannot fix a small or biased *original* sample - it estimates the sampling distribution of whatever the original sample actually contains; if that original sample is itself unrepresentative, every bootstrap resample inherits the same bias.

## 2.4 Confidence Intervals (pp.85–88)

**The correct frequentist interpretation, precisely** - a very common interview trip-up: a 95% confidence interval means that if this *sampling-and-interval-construction procedure* were repeated many times, **95% of the resulting intervals** would contain the true population parameter. It does **not** mean "there is a 95% probability the true value lies in *this specific* interval" - the true value either is or isn't in any given realized interval; the 95% describes the reliability of the *procedure*, not a probability statement about this one outcome. (This is the exact same distinction the Advanced Optimization chapter draws between confidence intervals and Bayesian credible intervals - worth connecting explicitly.)

## 2.5 Key Distributions (pp.88–105)

- **Normal distribution** (p.88) - PSDS's recurring caution, worth internalizing: the normal distribution is a convenient, tractable *approximation*, not a law of nature; many real datasets (financial returns, rare events, latency) are **not** well-approximated by it.
- **Long-tailed distributions** (pp.70, 89, 92) - real-world data frequently has heavier tails than a normal distribution would predict, meaning extreme events are *more common* than a naive normal-based model would estimate - directly relevant to risk modeling and anomaly detection.
- **Student's t-distribution** (pp.86, 95) - fatter tails than normal, parameterized by degrees of freedom; the correct choice for small-sample inference where the true population standard deviation is unknown and must itself be estimated from the sample.
- **Binomial distribution** (p.97) - the distribution of a count of successes across independent trials with fixed success probability; the natural distribution underlying A/B test conversion-rate comparisons.
- **Chi-square distribution** (p.103) - arises as the sampling distribution of a sum of squared standardized normal variables; underlies the chi-square test (Part 3.6).
- **Poisson and exponential distributions** (pp.100–103) - Poisson models the **count** of events in a fixed interval given a constant average rate; exponential models the **waiting time** between events under that same constant-rate assumption - the two are directly linked (exponential is the inter-arrival-time distribution for a Poisson process).
- **Weibull distribution** (pp.101–104) - generalizes the exponential distribution with a shape parameter, allowing the failure/event rate to increase or decrease over time (unlike the exponential's constant rate) - the standard distribution for reliability/failure-time (survival) analysis.

---

# Part 3 - Statistical Experiments & Significance Testing (PSDS Ch.3, pp.106–161)

**The single highest-yield chapter in this document for MLE interviews** - A/B testing and significance testing come up constantly in practice (model comparison, feature rollout decisions, experiment design) and are frequently under-prepared relative to core ML algorithms.

## 3.1 A/B Testing (pp.106–113)

The standard framework: a **control** group (existing behavior) and a **treatment** group (the change being tested), with **randomized** assignment between them. Randomization is what allows a later observed difference to be attributed to the treatment itself, rather than to some other systematic difference between the groups (Part 2.1's selection bias, avoided by construction). A test with no control group cannot distinguish "the treatment worked" from "things would have improved anyway" (seasonality, external trends, regression to the mean).

## 3.2 Hypothesis Tests and the Resampling/Permutation Framework (pp.112–123)

**PSDS's preferred, computationally-driven approach - genuinely worth knowing cold, not just the classical formulas.** A **permutation test**: pool the two groups' data together, then repeatedly *reshuffle* the group labels at random, recomputing the test statistic (e.g. difference in means) on each reshuffle. Because the reshuffled labels are random and uninformative by construction, the resulting distribution of the statistic *is* an empirical estimate of what the statistic would look like **under the null hypothesis of no real difference**. Comparing the *actually observed* statistic to this empirical null distribution gives a p-value directly, with essentially no distributional assumptions required - the same "simulate instead of assume a formula" spirit as the bootstrap.

```
Permutation test algorithm:
1. Pool both groups' data together.
2. Repeatedly (e.g. 10,000 times): randomly reshuffle group labels, recompute the statistic.
3. The resulting distribution of statistics IS the empirical null distribution.
4. p-value = fraction of shuffled statistics at least as extreme as the ACTUAL observed statistic.
```

**The p-value, defined precisely**: the probability, **under the null hypothesis**, of observing a result **at least as extreme** as what was actually observed. The single most common misinterpretation, worth explicitly guarding against in an interview answer: a p-value is **not** "the probability the null hypothesis is true" - it is a statement about how surprising the data would be *if* the null were true, not a probability over hypotheses themselves.

## 3.3 Statistical Significance, Type 1 and Type 2 Errors (pp.118–129)

- **α (significance level)**: the threshold p-value below which the null is rejected - conventionally 0.05, but this is a convention, not a law.
- **Type 1 error** (false positive): rejecting the null when it's actually true - concluding there's an effect when there isn't one.
- **Type 2 error** (false negative): failing to reject the null when it's actually false - missing a real effect.
- **Statistical vs. practical significance**: a real, important distinction PSDS emphasizes - with a large enough sample size, even a genuinely trivial, practically meaningless effect can become "statistically significant" (p < 0.05); the size of the effect and its real-world importance must be judged separately from whether it cleared a significance threshold.

## 3.4 t-Tests (pp.130–132)

The classical, formula-based alternative to a permutation test - appropriate when the sample size is small and a normality assumption (or the t-distribution's fatter-tailed correction for that, Part 2.5) is reasonable. Worth being able to state clearly: the permutation test and the t-test are usually answering the *same question* and tend to agree closely when their assumptions hold; the permutation test's advantage is not needing to assume a specific distributional form at all.

## 3.5 Multiple Testing (pp.132–136)

**The multiple comparisons problem**, precisely: testing many hypotheses simultaneously inflates the overall false-positive rate. Running 20 independent tests at `α=0.05`, **even with zero real effects present**, yields roughly **one false positive by chance alone**, on average - `1 - (1-0.05)^20 ≈ 0.64` probability of at least one false positive across the full set of 20 tests. Directly relevant whenever many metrics are compared in one A/B test, or many model configurations are compared across many datasets - a genuinely common, genuinely under-guarded-against failure mode in applied ML work. Correction methods (Bonferroni and others) adjust the significance threshold to account for the number of comparisons being made.

## 3.6 Degrees of Freedom, ANOVA, and the Chi-Square Test (pp.136–147)

- **ANOVA** (Analysis of Variance) - generalizes the t-test's "do two group means differ" to **more than two groups simultaneously**, testing whether *any* of several group means differ, without needing a separate pairwise test (and its associated multiple-testing inflation, Part 3.5) for every pair.
- **Chi-square test** - tests independence between two categorical variables, or goodness-of-fit between an observed categorical distribution and an expected one; the test statistic follows the chi-square distribution (Part 2.5) under the null of independence/fit.

## 3.7 Multi-Arm Bandits (pp.151–156)

An alternative to fixed 50/50 A/B allocation: **adaptively** shift traffic toward whichever arm is currently performing better *during* the experiment itself, rather than waiting for a fixed sample size before making any decision. This directly reduces the "cost" of testing - fewer users are exposed to a genuinely worse option - at the cost of more complex statistical analysis than a simple fixed-allocation test. Worth being able to contrast directly against classical A/B testing: bandits optimize for cumulative outcome *during* the experiment; classical A/B testing optimizes for a clean, unbiased *estimate* of the effect size, obtained *after* the experiment.

## 3.8 Power and Sample Size (pp.156–161)

**Statistical power** = `1 - P(Type 2 error)` = the probability of correctly detecting a real effect, given that one actually exists, at a specified effect size and sample size. Power, effect size, sample size, and `α` are all interlinked - fixing three determines the fourth. "How many samples do I need to reliably detect an effect of this size" is a genuinely common, very practical interview question, and is precisely a power/sample-size calculation, not a vague judgment call.

---

# Part 4 - Regression: Practical Notes (PSDS Ch.4, pp.162–219)

Full derivation-first coverage (normal equation, Ridge/Lasso, geometric intuition) lives in `Classical ML/01-Linear Models`. PSDS's own treatment is complementary and more applied: practical diagnostics (residual plots, checking homoscedasticity visually), factor/categorical variable encoding in a regression context, and polynomial/spline extensions for capturing nonlinearity without leaving the linear-regression framework. Worth a direct read alongside `Classical ML` notebooks 01–02 rather than a substitute for them.

---

# Part 5 - Classification: Evaluation (PSDS Ch.5, pp.220–264)

The algorithmic content (logistic regression derivation, decision trees, etc.) is covered in depth elsewhere in this repo; this section focuses specifically on PSDS's **evaluation** material, which is comprehensive and highly interview-relevant.

## 5.1 Confusion Matrix, Precision, Recall (pp.245–249)

|  | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

- **Precision** = `TP / (TP + FP)` - of everything predicted positive, what fraction actually was.
- **Recall** (a.k.a. sensitivity, true positive rate) = `TP / (TP + FN)` - of everything actually positive, what fraction was caught.

**The precision-recall tradeoff, with the concrete framing worth having ready in an interview**: spam detection generally prioritizes **precision** - a false positive (a real email marked spam) is costly and visible to the user, so the threshold is tuned conservatively. Cancer screening generally prioritizes **recall** - a false negative (a real case missed) is far more costly than a false positive (an unnecessary follow-up test), so the threshold is tuned aggressively toward catching more positives even at the cost of more false alarms. Which metric to prioritize is a business/domain decision, not a purely statistical one - worth explicitly stating this framing rather than defaulting to "just optimize F1."

## 5.2 ROC Curve and AUC (pp.250–256)

The **ROC curve** plots True Positive Rate (recall) against False Positive Rate (`FP/(FP+TN)`) across every possible classification threshold - a single curve summarizing a model's *entire* precision/recall tradeoff space, rather than one number at one threshold. **AUC** (area under the ROC curve) condenses this into one number, with a clean probabilistic interpretation: **AUC is the probability that a randomly chosen actual positive example is ranked (by the model's predicted score) above a randomly chosen actual negative example.** AUC = 0.5 is exactly equivalent to random guessing; AUC = 1.0 is a perfect ranking.

## 5.3 Strategies for Imbalanced Data (pp.257–263)

**Why plain accuracy is actively misleading on imbalanced data - a classic interview gotcha**: on a dataset that's 99% negative, a trivial classifier that *always* predicts negative achieves 99% accuracy while catching **zero** positive cases - accuracy alone completely fails to expose this. Precision/recall (Part 5.1) or AUC (Part 5.2) are the correct metrics to reach for instead.

Standard mitigation strategies:
- **Undersampling** the majority class - discard some majority-class examples to rebalance the training set; simple, but throws away data.
- **Oversampling** the minority class - duplicate minority-class examples; simple, but can encourage overfitting to those exact duplicated points.
- **SMOTE** (Synthetic Minority Oversampling Technique) - generates **synthetic** minority-class examples by interpolating between existing minority examples and their nearest neighbors, rather than duplicating exact points - a more sophisticated middle ground between plain oversampling and doing nothing.
- **Class weighting** - instead of resampling the data, weight the loss function itself so misclassifying a minority-class example costs more during training; achieves a similar effect without altering the dataset.

---

# Part 6–7 - Statistical Machine Learning & Unsupervised Learning (PSDS Ch.6–7, pp.265–)

Covered in full derivation-first depth in `Classical ML/02-Tree-Based Methods` onward (trees, bagging, boosting) and the planned `Classical ML/04-Unsupervised Learning` subchapter (k-means, hierarchical clustering, GMM/EM, dimensionality reduction). PSDS's treatment here is a good applied-practice companion to read alongside those notebooks, not a substitute for their derivations.

---

# Interview Rapid-Fire

- **Mean vs. median**: median is robust to outliers, mean is not - use median for skewed/outlier-heavy data.
- **Why `n-1` in sample variance**: Bessel's correction - dividing by `n` underestimates the true population variance because the sample mean fits the sample too closely.
- **Data distribution vs. sampling distribution**: individual values vs. the distribution of a *statistic* across many samples - the distinction confidence intervals are built on.
- **Bootstrap**: resample with replacement, many times, to empirically estimate a sampling distribution without assuming a parametric form.
- **Confidence interval, correctly stated**: describes the reliability of the *procedure* across repeated sampling, not a probability statement about one specific realized interval.
- **Permutation test**: reshuffle labels to build an empirical null distribution - the assumption-light alternative to a classical t-test.
- **p-value, correctly stated**: `P(data this extreme or more | null is true)` - never `P(null is true | data)`.
- **Type 1 vs. Type 2 error**: false positive vs. false negative; `α` controls Type 1 directly, power = `1 - P(Type 2)`.
- **Multiple testing problem**: many simultaneous tests inflate the false-positive rate - ~64% chance of at least one false positive across 20 independent tests at `α=0.05`, even with zero real effects.
- **Precision vs. recall, when to prioritize which**: spam (precision) vs. cancer screening (recall) - a business decision, not a purely statistical one.
- **AUC's precise meaning**: probability a random positive ranks above a random negative.
- **Why accuracy fails on imbalanced data**: a trivial always-majority classifier can score arbitrarily high while catching zero positives.

---

# Self-Check / Mastery Criteria

- [ ] Can state precisely why sample variance divides by `n-1`, not `n`
- [ ] Can explain the data-distribution vs. sampling-distribution distinction without conflating them
- [ ] Can implement a bootstrap confidence interval and a permutation test from scratch, and explain what each is doing conceptually
- [ ] Can state the correct frequentist interpretation of a confidence interval, and why the common misinterpretation is wrong
- [ ] Can state the correct definition of a p-value, and identify the common misinterpretation explicitly
- [ ] Can explain the multiple testing problem with an actual number, not just "it's a problem"
- [ ] Can state when to prioritize precision vs. recall with a concrete business example for each
- [ ] Can explain why accuracy is a misleading metric on imbalanced data, with a concrete numeric example
- [ ] Has looked at the actual PSDS pages cited above for at least the boxed formulas and worked examples in Chapters 1–3 and 5

