# Fraud Detection Case — Intern Data Scientist Brazil

## Slide 1 — Title and introduction

- I’m **Davi de Souza Santos**, a candidate for **Intern Data Scientist — Brazil**.
- I’ll present a simple, capacity-based workflow to rank bank transactions for fraud review.
- Repository: [fraudulent-transactions-prediction](https://github.com/davi-ssantos/fraudulent-transactions-prediction)

---

## Slide 2 — Business problem and approach

- Manual fraud review becomes harder to scale as transaction volume grows.
- The business must balance two costs:
  - **Missed fraud:** direct financial loss.
  - **False alarms:** unnecessary friction for legitimate customers.
- Fraud is rare, so overall accuracy can hide poor fraud detection.
- Approach: learn from labeled 2025 transactions, validate on a later time period, and rank January 2026 transactions by risk.
- The output is a prioritized review queue; it is not an automated transaction-blocking policy.

---

## Slide 3 — What the data says

- **2025 history:** 523,525 transactions; 20,485 labeled fraud cases (**3.91%**).
- Fraud rate rose to **5.42% in December**, an unusual pattern that may not repeat.
- Hour of day is the clearest signal; risk also varies with device status, merchant type, amount, device type, and city.
- Fraud made up **2.75% of transaction value**; the average fraud ticket was **R$30.86**, compared with **R$44.50** for legitimate transactions.
- January's input distributions were close to 2025 (PSI near zero), but distribution similarity does not prove that January's fraud behavior stayed the same.

---

## Slide 4 — Simple features and validation design

- The final model uses six features available when a transaction is made:
  - Hour and `is_new_device`.
  - Transaction amount and merchant type.
  - Device type and city.
- Keep the timestamp for ordering and traceability; do not pass it into the model.
- Exclude weak, redundant, or operationally uncertain fields to keep the approach explainable.
- Use a chronological split: first **70%** of 2025 for training, final **30%** for holdout evaluation.
- Prioritize **PR-AUC** for the rare-fraud problem; use ROC-AUC and precision/recall at review capacity as supporting measures.

---

## Slide 5 — Model results on the temporal holdout

- **Simple lookup benchmark:** PR-AUC **0.5416**; top-20% review captured **94.5%** of fraud.
- **Logistic regression:** PR-AUC **0.2665**; top-20% review captured **74.0%** of fraud.
- **HistGradientBoosting, simple features:** PR-AUC **0.6402**; top-20% review captured **96.4%** of fraud at **20.3% precision**.
- HGB is the selected candidate because it ranked the holdout better than the lookup and logistic baselines. It learns non-linear splits and interactions from the six simple inputs without manually expanding the feature set.
- These results describe one later-2025 holdout; they are not measured performance on January 2026.

---

## Slide 6 — Threshold choice: capacity first

- **Initial recommendation: review the top 10% by score.**
  - Holdout recall: **92.5%**.
  - Holdout precision: **39.0%**.
  - This uses half the review capacity of the top-20% scenario, which captured 96.4% of fraud.
- The holdout cutoff was **0.0590**, but it is not assumed to transfer to January.
- For January, rank January's own scores and select the highest-scoring 10%.
  - The generated January batch contains **44,451** rows; **4,446** were selected.
  - Its score boundary was **0.074361** in this run. This is a batch-specific capacity boundary, not a calibrated universal cutoff.
- Maximum F1 was **0.6219** at threshold **0.2698**, but that point was chosen and evaluated on the same holdout; treat it as exploratory only.

---

## Slide 7 — Initial scoring workflow and business operation

- The notebook refits HistGradientBoosting on all labeled 2025 rows, scores the January file, and exports the original January fields with:
  - `fraud_score`: model score used to rank the batch.
  - `fraud_prediction`: `1` for the review queue, `0` otherwise.
- Output: `src/outputs/Davi de Souza Santos - Intern Data Scientist Brazil.csv`.
- For this batch, route the **4,446 highest-scoring transactions** to the existing manual review process.
- Review teams should record dispositions and monitor queue volume and customer impact; use those outcomes to reassess capacity and policy when labels become available.
- Treat this as an initial batch-scoring workflow, not a production integration or an automatic decline rule.

---

## Slide 8 — Limitations and next steps

- January has no ground-truth labels: its precision, recall, F1, and PR-AUC cannot be measured from this case data.
- Validation uses one chronological holdout; performance may change across future periods.
- December's higher fraud rate may reflect a period-specific pattern.
- The top-10% policy assumes review capacity is the main operating constraint; the business should revisit it as review costs and outcomes become known.
- The scores are used for ranking; probability calibration and the financial cost of each error were not evaluated.
- Next step: revalidate the ranking and capacity policy on later labeled data.
