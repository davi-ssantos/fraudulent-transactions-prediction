# Fraud Detection Case: Intern Data Scientist Brazil

**Author:** Davi de Souza Santos

Machine Learning solution to identify potentially fraudulent transactions for a financial institution's Fraud Prevention team, built as the written case of an Intern Data Scientist selection process.

---
## Goals

### Business problem

The Fraud Prevention team of a financial institution has a consolidated process to review suspicious transactions. As the volume of operations grows, this process, which is largely manual, has become inefficient and hard to scale. This creates a double challenge:

- **Financial loss:** fraudulent transactions that are not detected in time.
- **Customer experience:** legitimate customers whose transactions are wrongly blocked.

The two errors have different costs. A missed fraud (false negative) is a direct loss, while a blocked legitimate transaction (false positive) hurts the customer relationship. Any solution has to balance both, and fraud is a rare event, so a model that simply labels everything as "not fraud" would look accurate while being useless.

The institution wants a Machine Learning solution that automates the identification of potentially fraudulent transactions, using a historical dataset in which part of the transactions were already labeled as fraud.

### Defined Goals

**Main goal:** build a simple predictive model that ranks transactions by fraud risk, and use it to prioritize the January 2026 transactions, which have no label, for manual review.

**Specific goals:**

1. **Explore and validate the data** to describe fraud patterns and data quality, and compare January 2026 feature distributions with the 2025 history.
2. **Select a small, explainable feature set** available when each transaction is made.
3. **Build and compare a simple benchmark and supervised models** using a chronological holdout from 2025.
4. **Evaluate ranking quality and review-capacity scenarios** with PR-AUC, precision, and recall, with ROC-AUC and F1 as supporting diagnostics.
5. **Recommend an operating capacity** for manual review and apply that ranking rule to January 2026; distinguish a capacity-based queue cutoff from a calibrated fraud-probability threshold.
6. **Deliver a reproducible notebook, scored January dataset, and presentation**, documenting assumptions and limitations.

**Scope and limits:** the January 2026 base has no labels, so its classification cannot be measured directly. Performance is estimated on held-out periods of 2025, and the conclusions about January rely on the assumption that its behavior is similar to the training data, which is checked in the EDA.

---

## Results summary

- The full 2025 history contains **523,525 transactions**, including **20,485 labeled fraud cases (3.91%)**. The fraud rate reached **5.42% in December**.
- Fraud accounted for **2.75% of transaction value**. The average fraud transaction was **R$30.86**, compared with **R$44.50** for legitimate transactions.
- The selected model candidate is **HistGradientBoosting** using six transaction-time features: hour, new-device flag, amount, merchant type, device type, and city. On the chronological 2025 holdout, it achieved **PR-AUC 0.6402**; reviewing the top 20% captured **96.4%** of holdout fraud at **20.3% precision**.
- The initial operating recommendation is to review the **top 10% by score**. On the holdout this captured **92.5%** of fraud at **39.0% precision**. The recommendation prioritizes review capacity; it is not an automatic blocking rule.
- The January 2026 scoring batch contains **44,451 transactions**. The exported file flags **4,446 (10%)** for review and includes each row's `fraud_score` and `fraud_prediction`. The January-specific score boundary was **0.074361** in this run.
- January has no ground-truth labels, so its fraud detection performance cannot be measured. Similar input distributions (PSI near zero) do not establish that the fraud pattern remained unchanged.

## Status

- [x] Case framing and data inspection
- [x] Full-year EDA and January-to-2025 distribution comparison
- [x] Simple feature selection and chronological model validation
- [x] January 2026 scoring and CSV export
- [x] Presentation draft and PDF deliverable
- [x] Consolidated notebook deliverable

The core case deliverables are prepared. The final January file is a prioritized manual-review queue, not a verified list of fraudulent transactions. Its quality can only be measured if January ground-truth labels become available.[^ai]

[^ai]: AI tools were used to support code development and documentation, including this README and refinement of sections organization in the Jupyter notebook. The analysis, interpretation of results, and final decisions were made by the author.
