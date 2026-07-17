# ML Engineer Interview Prep — 6-Month Roadmap

**e-book:** ```https://drive.google.com/drive/folders/1WHZhPslFVO9n-icyYU84IsZCKyqm2e1N?usp=drive_link```

---

## Suggested Repo Structure

```
ml-engineer-prep/
├── README.md                     ← this file (the plan)
├── 00-math-foundations/
├── 01-python-and-dsa/
├── 02-classical-ml/
├── 03-deep-learning/
├── 04-nlp-llm-genai/
├── 05-mlops-deployment/
├── 06-system-design/
├── 07-behavioral-prep/
├── 08-projects/
│   ├── project-1-classical-ml/
│   ├── project-2-deep-learning/
│   ├── project-3-nlp-llm/
│   ├── project-4-mlops-deployed-service/
│   └── project-5-capstone/
├── 09-mock-interviews/
└── progress-tracker.md           ← weekly checklist, update every Sunday
```

Each topic folder gets a `notes.md` (your own words, not copy-paste), a `problems.md` (things you got wrong + why), and code/notebooks. Writing your own notes is the single highest-leverage habit in this whole plan.

---

## Month 1 — Math, Python, DSA Basics, Classical ML Intro

**Goal:** rebuild the foundations so nothing later feels shaky.

- **Week 1 (Math for ML):** Linear algebra (vectors, matrices, eigenvalues/eigenvectors, SVD, matrix calculus), probability (distributions, Bayes' theorem, expectation/variance, MLE/MAP), basic statistics (hypothesis testing, confidence intervals, bias-variance). Resources: *Mathematics for Machine Learning* (Deisenroth et al.), CS229 math notes, 3Blue1Brown "Essence of Linear Algebra."
- **Week 2 (Python for ML + DSA warm-up):** NumPy/Pandas fluency, writing vectorized code, OOP in Python, complexity analysis (Big-O). Start DSA: arrays, strings, hashmaps, two pointers, sliding window (~2 problems/day on LeetCode/NeetCode).
- **Week 3 (DSA continued + SQL):** Stacks/queues, linked lists, trees, binary search, recursion basics. Add SQL fundamentals (joins, window functions, aggregations) — still commonly tested for MLE roles.
- **Week 4 (Classical ML intro):** Linear/logistic regression from scratch (no sklearn), gradient descent variants, regularization (L1/L2), evaluation metrics (accuracy, precision/recall, F1, ROC-AUC, log loss).

**Deliverable:** Notebook implementing linear regression + logistic regression + gradient descent from scratch with NumPy only.

---

## Month 2 — Classical ML Deep Dive + Feature Engineering

- **Week 1:** Decision trees, random forests, gradient boosting (XGBoost/LightGBM/CatBoost internals, not just API calls), bagging vs. boosting.
- **Week 2:** SVMs, k-NN, Naive Bayes, clustering (k-means, hierarchical, DBSCAN), dimensionality reduction (PCA, t-SNE, UMAP).
- **Week 3:** Feature engineering (encoding, scaling, handling missing data, outliers), imbalanced data (SMOTE, class weights, threshold tuning), cross-validation strategies, hyperparameter tuning (grid/random/Bayesian search).
- **Week 4:** Time series basics (ARIMA, feature-based forecasting), recommender systems intro (collaborative filtering, matrix factorization).
- **DSA (ongoing):** Binary trees/BSTs, graphs (BFS/DFS, topological sort), dynamic programming intro (1D DP).

**Deliverable:** End-to-end Kaggle-style tabular project (EDA → feature engineering → model comparison → tuned final model) with a written report.

---

## Month 3 — Deep Learning + Computer Vision + System Design Starts

- **Week 1:** Neural network fundamentals — forward/backprop by hand, activation functions, weight initialization, loss functions. Build a neural net from scratch (no framework), then redo it in PyTorch.
- **Week 2:** Training deep nets — optimizers (SGD, Adam, AdamW), batch norm/layer norm, dropout, learning rate schedules, overfitting/underfitting diagnostics, mixed precision training.
- **Week 3:** CNNs — convolution/pooling math, classic architectures (ResNet, VGG, EfficientNet), transfer learning, data augmentation. Reference: CS231n.
- **Week 4:** RNNs/LSTMs/GRUs (know them even though attention dominates now — still interview-relevant), sequence modeling basics.
- **System Design (start):** Learn the ML system design *framework* (clarify requirements → data → features → model choice → training → serving → monitoring → trade-offs). Read *Designing Machine Learning Systems* (Chip Huyen).
- **DSA (ongoing):** 2D DP, backtracking, heaps/priority queues.

**Deliverable:** Image classifier with transfer learning, trained and evaluated, with a short write-up of the architecture trade-offs you considered.

---

## Month 4 — NLP, Transformers, LLMs & GenAI

This is the section the **2026 market** weighs heavily — don't shortcut it.

- **Week 1:** NLP fundamentals — tokenization (BPE, WordPiece, SentencePiece), embeddings (word2vec, GloVe, contextual embeddings), classic NLP tasks. Then: the Transformer architecture in full detail — self-attention, multi-head attention, positional encodings, encoder/decoder variants. Implement attention from scratch. Reference: "Attention Is All You Need," Karpathy's "Let's build GPT."
- **Week 2:** LLM pretraining vs. fine-tuning, instruction tuning, RLHF/DPO at a conceptual level, parameter-efficient fine-tuning (LoRA, QLoRA, adapters), quantization basics.
- **Week 3:** Applied GenAI — RAG architecture (chunking, embeddings, vector databases like FAISS/Pinecone/Weaviate, retrieval strategies, re-ranking), prompt engineering patterns, structured output/function calling, agentic patterns (tool use, planning, multi-agent basics), evaluation of LLM outputs (rubrics, LLM-as-judge, hallucination detection).
- **Week 4:** LLM system concerns — context window management, cost/latency trade-offs, safety/guardrails basics, common failure modes. Reference: Hugging Face NLP course, CS224n.
- **System Design:** 2 case studies this month — a search/ranking system and a RAG-based product feature.
- **DSA (ongoing):** Mixed review, harder graph/DP problems, start timing yourself (45 min limit).

**Deliverable:** A working RAG application (your own document set → chunking/embedding pipeline → retrieval → LLM answer generation) with an evaluation script, not just a demo.

---

## Month 5 — MLOps, Deployment, Distributed Systems, System Design Intensive

- **Week 1:** Experiment tracking (MLflow/W&B), data/model versioning (DVC), reproducibility.
- **Week 2:** Model serving (REST/gRPC endpoints, batch vs. real-time inference, model formats like ONNX/TorchScript), containerization (Docker), basic Kubernetes concepts, CI/CD for ML.
- **Week 3:** Monitoring & observability (data drift, model drift, performance monitoring, alerting), feature stores, A/B testing for ML, shadow deployments, canary releases.
- **Week 4:** Distributed training (data parallel vs. model parallel, gradient accumulation, mixed precision at scale), cloud ML platforms (SageMaker/Vertex AI at a conceptual level — know what they automate). Reference: MLOps Zoomcamp, Full Stack Deep Learning, *Designing Machine Learning Systems*.
- **System Design (heavy):** Do one full case study per week — e.g., feed ranking, fraud detection, ad click-through prediction, video recommendation, search relevance, an LLM-powered feature at scale. Write up each one as if presenting to an interviewer, including trade-offs and back-of-envelope numbers.
- **DSA:** Start full mock coding interviews (45 min, 1–2 problems, verbalize your thinking).

**Deliverable:** Deploy one of your earlier models (or the RAG app) as a served API with monitoring/logging, containerized, with a basic CI pipeline. This becomes your strongest portfolio piece.

---

## Month 6 — Interview Intensive: Mocks, Behavioral, Applications

- **Week 1:** Full mock interview loop (coding + ML breadth + ML system design + behavioral) with a peer, mentor, or structured self-recording. Identify weak spots and drill them.
- **Week 2:** Behavioral prep — STAR-format stories for: conflict with a teammate, a project that failed, dealing with ambiguity, influencing without authority, a time you disagreed with a decision. ML-specific behavioral: "walk me through a model you built," "how did you handle a data quality issue," "how did you decide to ship or not ship a model." Also prep questions *you'll* ask them.
- **Week 3:** Portfolio & resume polish — clean up your repo, write clear READMEs per project, quantify impact in your resume bullets, tailor resume per role type (generalist vs. NLP vs. MLOps).
- **Week 4:** Applications, networking, and negotiation prep — apply broadly, do informational interviews, learn how ML offer negotiation typically works (base/equity/bonus, leveling).
- **Ongoing:** Keep doing 1–2 LeetCode problems and 1 system design case study per week so nothing goes stale while you're interviewing.

**Deliverable:** A polished, public GitHub repo with 4–5 strong projects, a tailored resume, and a bank of 10+ rehearsed behavioral stories.

---

## Full Topic Checklist (nothing here should be skipped)

### Math
- [ ] Linear algebra: vectors, matrix ops, eigen-decomposition, SVD
- [ ] Probability: distributions, Bayes' theorem, expectation/variance
- [ ] Statistics: hypothesis testing, confidence intervals, A/B testing
- [ ] Calculus/optimization: gradients, chain rule, convexity, gradient descent variants

### Programming / CS Fundamentals
- [ ] Python fluency (OOP, decorators, generators, vectorization)
- [ ] Data structures: arrays, hashmaps, linked lists, stacks/queues, trees, graphs, heaps
- [ ] Algorithms: sorting/searching, BFS/DFS, DP, backtracking, greedy
- [ ] SQL: joins, window functions, aggregation, query optimization basics

### Classical ML
- [ ] Linear/logistic regression, regularization
- [ ] Decision trees, random forest, gradient boosting (XGBoost/LightGBM)
- [ ] SVM, k-NN, Naive Bayes
- [ ] Clustering: k-means, hierarchical, DBSCAN
- [ ] Dimensionality reduction: PCA, t-SNE, UMAP
- [ ] Feature engineering, handling missing/imbalanced data
- [ ] Model evaluation metrics (classification + regression)
- [ ] Cross-validation, hyperparameter tuning

### Deep Learning
- [ ] Backpropagation from first principles
- [ ] Optimizers, initialization, normalization, regularization (dropout, weight decay)
- [ ] CNNs and classic architectures
- [ ] RNNs/LSTMs/GRUs
- [ ] Transformers: self-attention, positional encoding, encoder/decoder
- [ ] Training diagnostics: over/underfitting, vanishing/exploding gradients

### NLP / LLMs / GenAI
- [ ] Tokenization, embeddings
- [ ] Transformer architecture deep dive
- [ ] Pretraining vs. fine-tuning, instruction tuning, RLHF/DPO (conceptual)
- [ ] PEFT: LoRA, QLoRA, adapters
- [ ] Quantization basics
- [ ] RAG: chunking, embeddings, vector DBs, retrieval, re-ranking
- [ ] Prompt engineering, function calling/structured outputs
- [ ] Agentic patterns: tool use, planning, multi-agent
- [ ] LLM evaluation: rubrics, LLM-as-judge, hallucination detection
- [ ] Safety/guardrails basics

### MLOps / Deployment
- [ ] Experiment tracking, data/model versioning
- [ ] Model serving (batch vs. real-time), model formats (ONNX/TorchScript)
- [ ] Docker, basic Kubernetes concepts
- [ ] CI/CD for ML
- [ ] Monitoring: data drift, model drift, alerting
- [ ] Feature stores, A/B testing, canary/shadow deployments
- [ ] Distributed training basics
- [ ] Cloud ML platforms (conceptual understanding)

### System Design (ML-specific)
- [ ] The standard ML system design framework
- [ ] Recommendation systems
- [ ] Search/ranking systems
- [ ] Fraud/anomaly detection
- [ ] Ad click-through prediction
- [ ] Feed ranking
- [ ] LLM-powered product features at scale
- [ ] Back-of-envelope estimation (QPS, storage, latency budgets)

### Coding Interview
- [ ] Core DSA patterns (two pointers, sliding window, DFS/BFS, DP, greedy)
- [ ] ML-from-scratch implementations (k-means, backprop, attention, gradient descent)
- [ ] Verbalizing your thought process under time pressure

### Behavioral / Communication
- [ ] STAR-format story bank (8–12 stories)
- [ ] ML-specific behavioral answers (failed models, data issues, trade-off decisions)
- [ ] Questions to ask interviewers
- [ ] Negotiation basics (comp structure, leveling)

---

## Weekly Rhythm (template)

| Day | Track A (theme) | Track B (coding) | Track C/D |
|---|---|---|---|
| Mon–Fri | ~60–90 min theory/reading + notes | 2 problems | — |
| Sat | Project/deliverable work | — | 1 system design case study (month 3+) |
| Sun | Review + update `progress-tracker.md` | Weekly mock (month 5+) | Behavioral story review (month 4+) |

## Tracking Progress

Keep `progress-tracker.md` at the repo root with one checklist section per week. Check items off, and log every problem/question you got wrong with a one-line "why" — that log is your best last-two-weeks review material before real interviews.

---

**Next step:** I can scaffold the actual folder structure with starter `notes.md`/`README.md` files in each directory, or start on Month 1 Week 1 content — just say which.
