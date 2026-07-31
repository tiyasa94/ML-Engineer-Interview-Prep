# ML Engineer Interview Prep

A 6-month, project-based prep repo for ML Engineer interviews in the current job market - math foundations through classical ML, deep learning, NLP/LLMs, MLOps, and system design, built to be genuinely interview-ready, not just "read about it once."

## Progress

- [x] **Mathematical Foundations** - Linear Algebra, Calculus, Statistics, Information Theory ✅
- [x] **Python (Software/ML Engineering)** - fundamentals through advanced: OOP, concurrency, async, APIs/FastAPI, packaging, Git, interview gotchas, DSA LeetCode Guide ✅
- [ ] Classical ML - regression, tree ensembles, SVMs, clustering, feature engineering
- [ ] Deep Learning - neural nets from scratch, CNNs, RNNs, training dynamics
- [ ] NLP & LLMs / GenAI - transformers, fine-tuning, RAG, agents, evaluation
- [ ] MLOps & Deployment - serving, monitoring, CI/CD, distributed training
- [ ] System Design - ML system design case studies, estimation
- [ ] Interview Intensive - mock interviews, behavioral prep, portfolio polish, applications

## Repository structure

```
ML-Engineer-Interview-Prep/
├── Mathematical Foundations/
│   ├── Linear Algebra/
│   │   ├── 01-vectors-spaces-transformations.{md,ipynb}
│   │   ├── 02-norms-inner-products-projections.{md,ipynb}
│   │   ├── 03-eigenvalues-svd-pca.{md,ipynb}                    ⭐
│   │   └── figures/
│   ├── Calculus/
│   │   ├── 01-differentiation-gradients-jacobians.{md,ipynb}
│   │   ├── 02-chain-rule-backprop-autodiff.{md,ipynb}           ⭐
│   │   ├── 03-optimization-gradient-descent-convexity.{md,ipynb}
│   │   └── figures/
│   ├── Statistics/
│   │   ├── 01-probability-foundations-bayes.{md,ipynb}
│   │   ├── 02-distributions-mle-map.{md,ipynb}                  ⭐
│   │   ├── 03-hypothesis-testing-bias-variance.{md,ipynb}
│   │   └── figures/
│   └── Information Theory/
│       ├── 01-entropy-cross-entropy.{md,ipynb}
│       ├── 02-kl-divergence-mutual-information.{md,ipynb}       ⭐
│       ├── 03-ml-applications-entropy.{md,ipynb}
│       └── figures/
├── Python & DSA/
│   ├── 01-fundamentals-syntax-control-flow.ipynb
│   ├── 02-data-structures-native.ipynb
│   ├── 03-functions-oop-for-ml.ipynb
│   ├── 04-iterators-generators-context-managers.ipynb
│   ├── 05-threading-gil-multiprocessing.ipynb
│   ├── 06-async-await-asyncio.ipynb
│   ├── 07-api-fundamentals.ipynb
│   ├── 08-pydantic-and-api-concepts.ipynb
│   ├── 09-ml-serving.ipynb
│   ├── 10-exceptions-memory-performance.ipynb
│   ├── 11-packaging-project-structure.ipynb
│   ├── 12-git-version-control.ipynb
│   ├── 13-python-interview-gotchas.ipynb                        
│   └── 14-DSA-LeetCode-Guide.pdf                                  
├── requirements.txt
├── .gitignore
└── README.md
```

## How each topic is organized

**Mathematical Foundations** - paired `.md` + `.ipynb` per topic:
- `.md` - theory, intuition, derivation, a worked visual, and why it matters for ML. Ends with a self-check checklist.
- `.ipynb` - companion code, pre-executed, plots render immediately.
- `figures/` - the PNGs the `.md` files embed.

**Python & DSA** - `.ipynb` only, concise and interview/revision-focused, no companion `.md`. Every notebook is fully executed, and wherever practical the demos are genuinely live rather than simulated: real local HTTP servers hit with real `requests` calls, a real `pip install -e .` of a scratch package, a real git repo with an actual merge conflict created and resolved live. Several real bugs only surfaced this way (e.g. a bare git remote's `HEAD` pointing at a nonexistent ref, silently producing empty clones, or a benchmark claim that turned out false on modern CPython 3.11+) - direct execution catches gotchas that descriptions alone would miss.

## Setup

```bash
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook
```

## Books referenced

- **MML** - *Mathematics for Machine Learning* (Deisenroth, Faisal, Ong) - free at [mml-book.com](https://mml-book.com) - cited by exact section/page throughout Linear Algebra, Calculus, and most of Statistics.
- **PSDS** - *Practical Statistics for Data Scientists* (Bruce & Bruce, O'Reilly) - cited for the applied-statistics half of Statistics (bootstrap, confidence intervals, hypothesis testing, bias-variance) and for decision-tree entropy in Information Theory.
- Information Theory's core definitions (entropy, cross-entropy, KL divergence, mutual information) aren't covered in depth by either book above, so that subchapter instead follows the standard treatment (Cover & Thomas, *Elements of Information Theory*; MacKay, *Information Theory, Inference, and Learning Algorithms*, free online) - noted explicitly at the top of its first notebook.
- Python & DSA is original content (no textbook citations); FastAPI/Pydantic/SQLAlchemy sections follow each library's official documentation conventions.

## Next up

**Classical ML** - feature engineering, decision trees/random forests/gradient boosting, SVMs, clustering, evaluation metrics - same `.md` + `.ipynb` + `figures/` format as Mathematical Foundations. DSA notebooks remain parked until picked back up.