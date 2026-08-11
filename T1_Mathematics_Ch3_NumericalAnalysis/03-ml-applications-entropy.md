# 03 -  ML Applications: Decision Trees, Distillation & Perplexity

**Companion notebook:** [`03-ml-applications-entropy.ipynb`](./03-ml-applications-entropy.ipynb)
**Book reference:** *Practical Statistics for Data Scientists* (Bruce & Bruce) (https://drive.google.com/file/d/1LGisFRLBa8YojQ7EOd4rPUWeicjsN8ys/view) -  Ch. 6 "Statistical Machine Learning," section "Measuring Homogeneity or Impurity" (entropy as a tree-splitting criterion, alongside Gini impurity).

This notebook is where entropy, cross-entropy, and KL divergence stop being abstract and turn into three concrete things you'll actually touch in practice.

---

## Decision trees: entropy as a splitting criterion

A decision tree grows by repeatedly picking the split that reduces entropy the most. **Information gain** for a candidate split is:

```
Gain = H(parent) - Σ (|child| / |parent|) · H(child)
```

The tree greedily picks whichever available split maximizes this gain at each node -  i.e., whichever split makes the resulting groups as *pure* (low-entropy) as possible.

![Information gain from candidate splits](./figures/information_gain.png)

Split A leaves both children with the *exact same* class mixture as the parent -  the feature told you nothing, and information gain correctly reports zero. Split B produces one pure child and one nearly-pure child -  a large entropy reduction, and the split a real tree-growing algorithm (ID3, C4.5, and CART's entropy criterion -  the alternative to Gini impurity from the Statistics chapter) would pick. This is the exact mechanism, not just an analogy: entropy from notebook 1 *is* the criterion.

## Knowledge distillation: temperature-scaled softmax and KL divergence

**Knowledge distillation** trains a smaller "student" model to mimic a larger "teacher" model's output distribution, rather than only the hard labels. The training objective is (a version of) KL divergence between the teacher's and student's predicted distributions -  literally minimizing `KL(teacher ‖ student)`.

The trick that makes this work well: **temperature scaling**. Instead of the normal softmax, divide the logits by a temperature `T > 1` before applying softmax, which softens the distribution and reveals the teacher's *full ranking* of alternatives (its "dark knowledge"), not just its single top prediction.

![Temperature-scaled softmax](./figures/temperature_scaling.png)

At `T=0.5` the model looks almost certain it's a cat and nothing else -  useful for final predictions, but it hides the fact the teacher also thought "dog" was plausible and "bear" was essentially ruled out. At `T=5`, that entire ranking becomes visible in the probabilities -  which is exactly the extra signal distillation wants the student to learn from. (You'll also recognize temperature scaling if you've adjusted "temperature" when sampling from an LLM -  same formula, same effect: lower temperature makes generation more deterministic/repetitive, higher temperature makes it more diverse/random.)

## Perplexity: cross-entropy, exponentiated

**Perplexity** is the standard evaluation metric for language models, and it's directly built from cross-entropy -  not a separate concept:

```
Perplexity = 2^(cross-entropy in bits) = exp(cross-entropy in nats)
```

```
cross-entropy = 0.5 bits/token  ->  perplexity =    1.41
cross-entropy = 1.0 bits/token  ->  perplexity =    2.00
cross-entropy = 2.0 bits/token  ->  perplexity =    4.00
cross-entropy = 4.0 bits/token  ->  perplexity =   16.00
cross-entropy = 8.0 bits/token  ->  perplexity =  256.00
```

Perplexity has an intuitive reading: **it's the effective number of equally-likely choices the model is "confused" between, on average, at each token.** A perplexity of 1 means the model is perfectly certain every time (cross-entropy = 0). A perplexity of, say, 20 means the model's uncertainty at each step is *as if* it were choosing uniformly among about 20 options -  even though the actual vocabulary might have 50,000+ tokens. Lower perplexity means a better (more confident, on the held-out true continuation) language model -  and because it's just cross-entropy re-expressed, minimizing an LLM's training loss (cross-entropy) and minimizing its perplexity are the same optimization.

This closes the loop for this entire subchapter: **entropy → cross-entropy → KL divergence → decision tree splits, distillation, and LLM evaluation** are not four separate topics to memorize -  they're one idea (how many bits does it take to describe outcomes from a distribution, and what happens when your model's beliefs don't match reality) showing up in four different corners of the field.

---

## Self-check before moving on

- [ ] I can explain how a decision tree uses entropy/information gain to pick splits
- [ ] I can explain what temperature scaling does to a softmax distribution, and why distillation uses a high temperature
- [ ] I can state the relationship between perplexity and cross-entropy, and give an intuitive reading of what perplexity "means"
- [ ] I can trace the throughline: entropy defines cross-entropy, cross-entropy's gap from entropy is KL divergence, and all three show up directly in tools you'll use (tree splits, distillation, LM evaluation)

This closes out the Information Theory subchapter, and with it, the full Math Foundations chapter: Linear Algebra → Calculus → Statistics → Information Theory. Next up in the 6-month plan: **Month 2, Classical ML** (feature engineering, tree ensembles, SVMs, clustering).
