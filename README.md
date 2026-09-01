# Retail Personalisation & Recommendation System

An end-to-end recommendation system for generating personalised weekly product recommendations from large-scale retail transaction data.

The project explores how a retailer can move from simple popularity-based recommendations towards a multi-stage personalised recommendation architecture using collaborative filtering, candidate generation and learning-to-rank.

The system is evaluated using temporal offline evaluation designed to simulate a real weekly recommendation cycle.

> **Status:** In development — baseline recommenders and collaborative filtering have been implemented. Candidate generation, ranking and productionisation are in progress.

---

## Business Problem

Retailers with large product catalogues need to decide which products are most relevant to each customer.

The objective of this project is to simulate a weekly personalisation system that:

> **Generates up to 12 product recommendations for each customer using only information available before the recommendation date.**

Recommendations are evaluated against products subsequently purchased during the following seven days.

This creates several practical recommender-system challenges:

- extremely sparse customer–product interactions;
- implicit rather than explicit feedback;
- rapidly changing product popularity;
- customers with limited or no interaction history;
- large and changing product catalogues;
- balancing personalisation with strong popularity signals.

---

## Dataset

The project uses the public **H&M Personalized Fashion Recommendations** dataset.

The dataset contains approximately:

- **1.37 million customers**
- **105,000 articles**
- **31 million transaction records**

The original data is not included in this repository.

### Interaction Characteristics

Exploratory analysis found that the customer–article interaction matrix is approximately **99.98% sparse**.

Purchases represent implicit positive feedback: an unobserved customer–article interaction does not necessarily indicate dislike, since the customer may simply never have encountered the product.

Approximately **87% of observed customer–article pairs occur once**, while around **13% contain repeat purchases**.

These characteristics motivate the use of implicit-feedback recommendation approaches.

---

## Recommendation Architecture

The project is being developed as a multi-stage recommendation system rather than a single-model solution.

```text
Customer & Transaction History
            |
            v
    Candidate Generation
    --------------------
    Recent Popularity
    Collaborative Filtering (ALS)
    Item-Item Similarity
    Additional retrieval models
            |
            v
      Candidate Pool
            |
            v
     Learning-to-Rank
            |
            v
      Re-ranking
    ----------------
    Diversity
    Freshness
    Business Rules
    Cold-start Fallbacks
            |
            v
        Top 12
    Recommendations
```

The modular architecture allows different recommendation approaches to contribute candidates before a ranking model determines the final recommendation order.

---

## Offline Evaluation

A random train/test split would introduce temporal leakage and would not represent how a recommendation system operates in production.

Instead, the project uses a temporal evaluation strategy:

```text
Historical transactions                  Future purchases
------------------------------------|------------------------
            Training                |   7-day evaluation
                                    |
                            Recommendation time
```

Models use only interactions occurring before the evaluation period.

For each customer, the system generates up to 12 recommendations and compares them with products actually purchased during the following seven days.

### Metrics

The primary offline metrics are:

**Recall@12**

Measures the proportion of products actually purchased that appeared in the 12 recommendations.

**MAP@12**

Measures both whether relevant products were retrieved and how highly they were ranked.

Additional evaluation will include catalogue coverage, popularity bias, customer-segment performance and cold-start behaviour.

---

## Baseline Recommenders

Two non-personalised baselines were initially evaluated.

| Model | Recall@12 | MAP@12 |
|---|---:|---:|
| Global Popularity | 0.00775 | 0.00290 |
| **Recent Popularity (7 days)** | **0.02550** | **0.00875** |

Recent popularity substantially outperformed popularity calculated across the full historical period.

Experiments with longer recent-popularity windows (14 and 28 days) produced lower performance, suggesting that **short-term product demand is particularly important for predicting next-week purchases**.

The 7-day recent-popularity model therefore serves as the primary baseline for subsequent personalised models.

---

## Collaborative Filtering — Implicit ALS

The first personalised model uses **Alternating Least Squares (ALS)** for implicit collaborative filtering.

Customer–article interactions are represented as a sparse matrix and factorised into latent customer and article representations.

Two interaction-strength strategies were evaluated:

### Binary interactions

Every observed customer–article interaction receives equal weight regardless of purchase frequency.

### Log-frequency interactions

Repeat purchases provide stronger evidence of preference, while logarithmic transformation prevents extreme repeat counts from dominating the interaction signal.

```text
interaction_strength = log(1 + purchase_count)
```

### Initial Results

| Model | Recall@12 | MAP@12 |
|---|---:|---:|
| **Recent Popularity (7 days)** | **0.02550** | **0.00875** |
| Binary ALS | 0.01203 | 0.00431 |
| Log-Frequency ALS | 0.01195 | 0.00437 |

The initial ALS models do **not** outperform the recent-popularity baseline.

Rather than treating this as a failed experiment, this result highlights an important characteristic of the problem: **recent product demand currently provides a stronger signal than collaborative behaviour alone**.

Binary and log-frequency weighting also produced very similar performance, suggesting that repeat-purchase weighting is not a major driver of performance under the current ALS configuration.

Further experiments will investigate recency-aware interactions, repeat-purchase eligibility, model hyperparameters and hybrid candidate generation.

---

## Customer History Analysis

ALS performance was analysed across customers with different amounts of historical interaction data.

| Historical Articles | Customers | Recall@12 | MAP@12 |
|---|---:|---:|---:|
| 1–2 | 2,300 | 0.01331 | 0.00554 |
| 3–5 | 3,527 | 0.01772 | 0.00686 |
| 6–20 | 14,569 | 0.01527 | 0.00612 |
| 21–50 | 19,353 | 0.01233 | 0.00406 |
| 51+ | 23,663 | 0.00860 | 0.00305 |

Performance does not monotonically increase with customer history.

This motivates further failure analysis rather than assuming that more historical interactions automatically produce better recommendations. Potential factors include differences in future basket size, changing customer preferences and the treatment of previously purchased articles.

---

## Cold Start

ALS requires historical interactions to learn customer and article representations.

Customers without observable transaction history therefore require alternative recommendation strategies.

Planned fallback approaches include:

- recent popularity;
- segment/category popularity;
- content-based recommendations using article metadata.

Cold-start performance will be evaluated separately from customers with established interaction histories.

---

## Current Project Roadmap

- [x] Data quality and interaction analysis
- [x] Temporal offline evaluation framework
- [x] Global popularity baseline
- [x] Recent popularity baseline
- [x] Implicit ALS collaborative filtering
- [x] Binary vs frequency-weighted interactions
- [x] Customer-history performance analysis
- [ ] ALS failure analysis and optimisation
- [ ] Item-item candidate generation
- [ ] Multi-source candidate generation
- [ ] Learning-to-rank
- [ ] Cold-start strategy
- [ ] Diversity and catalogue-coverage evaluation
- [ ] MLflow experiment tracking
- [ ] Batch recommendation pipeline
- [ ] API serving
- [ ] Docker containerisation
- [ ] Automated testing and CI/CD
- [ ] Model and data monitoring design

---

## Repository Structure

```text
retail-personalisation-system/
├── notebooks/
│   ├── 01_data_understanding.ipynb
│   ├── 02_baseline_recommenders.ipynb
│   └── 03_collaborative_filtering.ipynb
├── src/
│   └── recommender/
├── tests/
├── docs/
│   └── recommender_system_notes.md
├── README.md
├── .gitignore
└── LICENSE
```

Exploratory work is initially developed in notebooks. Stable data-processing, recommendation and evaluation components will progressively be moved into reusable modules under `src/`.

---

## Technology

**Current**

- Python
- Pandas
- NumPy
- SciPy
- implicit
- PySpark
- Git / GitHub

**Planned**

- LightGBM / learning-to-rank
- MLflow
- FastAPI
- Docker
- pytest
- GitHub Actions

---

## Key Learnings So Far

The initial experiments demonstrate why recommendation systems require more than selecting a sophisticated algorithm.

A simple recent-popularity model currently outperforms collaborative filtering, highlighting the importance of **temporal relevance and strong baselines**.

The next stages of the project focus on combining complementary recommendation signals rather than relying on a single model:

```text
recency + collaborative behaviour + item similarity
                         ↓
                 candidate generation
                         ↓
                       ranking
```

This reflects the broader goal of the project: designing and evaluating an end-to-end personalisation system rather than optimising a single recommendation algorithm in isolation.

---

## Data Attribution

This project uses the public **H&M Personalized Fashion Recommendations** dataset released through Kaggle.

Dataset files are not redistributed in this repository.
