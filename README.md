# Tamweel Lite — Advanced Machine Learning Methods

An end-to-end machine learning project developed as part of the **Advanced Machine Learning Methods** course at SDAIA Academy.

**Developer:** Abdulaziz Aldhaayan
**Student ID:** S06
**Course:** `SDA-DSC-211 — Advanced Machine Learning Methods | أساليب تعلم الآلة المتقدمة`
**Final version:** Git tag `Final`

> **Note:** This is an educational learner project developed using synthetic course data. It is not an official SDAIA Academy repository.

---

## 📌 Project Overview

Tamweel Lite focuses on a simulated lending-review problem where the goal is to identify applications that should be prioritized for human review.

Failing to identify a risky application is considered much more costly than reviewing an application that turns out to be non-risky, and the number of applications that can be reviewed is limited. The project therefore builds a **cost-sensitive classification and decision-making workflow** rather than maximizing prediction accuracy.

The model produces a **review flag** (`decision = 1`) indicating that an application should be considered for human review. It does not make an actual loan approval or rejection decision.

---

## 🎯 Problem Definition

> Which applications should be selected for additional human review when missing a risky case costs much more than reviewing a non-risky one?

Course-defined decision policy:

```text
False Negative cost = 10
False Positive cost = 1
Maximum review capacity = 12% of requests per period
Loss = 10 × FN + 1 × FP
```

The target is a synthetic default event within 90 days after the application (`default_within_90d`).

---

## 🔎 Project Workflow

1. Checking the available data and defining data roles.
2. Separating datasets using temporal and customer-aware rules.
3. Building baseline and candidate machine learning models.
4. Generating forward out-of-fold (OOF) predictions.
5. Comparing Logistic Regression, XGBoost, and LightGBM.
6. Testing different ensemble approaches.
7. Applying the documented Worth-It Gate.
8. Selecting a cost-sensitive threshold.
9. Applying the 12% review-capacity constraint.
10. Evaluating calibration behavior.
11. Performing interpretability and stability analysis.
12. Retraining the selected final model.
13. Generating predictions for the challenge dataset.
14. Producing the final submission and provenance artifacts.

---

# 🗓️ Results by Day

## Day 1 — Baseline comparison

Single random stratified split used for teaching only (about 158 positives in the comparison set).

| Model | ROC-AUC | Train time |
| ----- | ------: | ---------: |
| Logistic Regression | 0.8213 | 0.06 s |
| LightGBM | 0.8138 | 1.07 s |
| XGBoost | 0.8124 | 2.67 s |

The gaps of about 0.01 ROC-AUC on one split are likely noise, so the tree models showed no clear advantage. ROC-AUC can hide many false positives under class imbalance, which is why Average Precision (AP) is used from Day 2 onward.

## Day 2 — Leakage and validation

* Post-outcome fields (for example `days_past_due_60`) were removed before modeling.
* Forward, customer-purged folds with a 90-day label-maturity rule were used.
* AP gap, leaky minus clean random split: **0.6879** (leakage inflates results massively).
* AP gap, clean random minus honest time/customer split: **−0.0044** (small; the populations differ, 10,000 vs 5,039 requests).
* Hyperparameter search: 8 trials within 120 seconds on a reserved cohort.
* OOF predictions cover **50.39%** of the data (5,039 eligible rows).

## Day 3 — Threshold and capacity (weighted LightGBM, pooled OOF)

| Rule | Threshold | Loss | Flags | Recall | Precision | Capacity |
| ---- | --------: | ---: | ----: | -----: | --------: | -------- |
| Default | 0.5 | 2,403 | 19.94% | 0.5781 | 0.2209 | Violated |
| Capacity-constrained | 0.6583 | 2,639 | 10.44% | 0.4089 | 0.2985 | Satisfied |

* Enforcing the 12% capacity costs **+236** loss units (FN rises from 162 to 227).
* Accuracy misleads: a "flag nobody" rule reaches 0.9238 accuracy (defaults are 7.62% of requests) but has zero recall and the highest loss (3,840).
* Regional false-positive-rate gap: **0.648 percentage points** (western 8.29%, other 7.64%); descriptive only.

## Day 4 — Explainability and calibration (weighted LightGBM)

| Metric (1,733 evaluation rows, 139 positives) | Raw | Sigmoid |
| --------------------------------------------- | --: | ------: |
| Brier | 0.113027 | 0.067112 |
| ECE | 0.146871 | 0.022486 |
| ROC-AUC | 0.7708 | 0.7708 |
| Average Precision | 0.2587 | 0.2587 |

* Top features: `bureau_score` (permutation AP drop 0.1270, mean |SHAP| 0.9042 log-odds) and `dti` (0.0690, 0.5437).
* Local example `TR-009585`: raw score 0.9031, calibrated probability 0.4795.
* Customer-cluster bootstrap (200 replicates): AP 95% interval [0.1974, 0.3379]; Brier change [−0.0542, −0.0374].
* Policy threshold 0.588195 (raw) transported to 0.17331 (calibrated). Capacity status: `CAPACITY_REVIEW_REQUIRED` (2024Q4: 109 risk flags against a limit of 107).

---

# 📊 Day 5 — Model Comparison

Models were evaluated using nested out-of-fold predictions across three forward validation periods (2023Q1, 2023Q3, 2024Q1). Average Precision was the primary metric; Brier Score, ECE, and fold stability were also considered.

| Model               | Mean AP | Fold SD | Mean Brier | Mean ECE |
| ------------------- | ------: | ------: | ---------: | -------: |
| Logistic Regression | 0.39166 | 0.02981 |    0.06327 |  0.01882 |
| Weighted Ensemble   | 0.38942 | 0.02906 |    0.06332 |  0.01772 |
| Stacking            | 0.38314 | 0.02949 |    0.06603 |  0.03106 |
| Equal Ensemble      | 0.37170 | 0.03258 |    0.06435 |  0.02038 |
| XGBoost             | 0.35263 | 0.02904 |    0.06566 |  0.02276 |
| LightGBM            | 0.34549 | 0.04348 |    0.06608 |  0.02311 |

### Final Model

**Logistic Regression** was selected (`KEEP SINGLE`). No ensemble passed the Worth-It Gate: the weighted average changed AP by −0.0022, the equal average by −0.0200, and stacking by −0.0085, none exceeding the fold SD. Stacking also worsened ECE by +0.0122 and Brier by +0.0028.

---

# 🧪 Validation Strategy

The validation process reduces leakage by considering both time and customer identity.

| Role            |  Rows | Customers | Positives |
| --------------- | ----: | --------: | --------: |
| Fit + Selection | 6,576 |     3,931 |       537 |
| Calibration     |   836 |       788 |        78 |
| Excluded        | 2,588 |         — |         — |

* **2,155 OOF predictions** across **3 forward validation periods**.
* Ensemble weights and the stacker were learned on inner OOF only, with customer separation and label maturity.
* Warm-up observations without an outer OOF prediction were excluded from the comparison.
* OOF results are development evidence and are **not an untouched final test set**.

---

# ⚖️ Cost-Sensitive Decision Policy

After selecting the final model, a threshold was chosen that respects the cost function and the capacity requirement.

| Item | Value |
| ---- | ----: |
| Raw OOF threshold | 0.168922 |
| Calibrated (transported) threshold, applied to the challenge set | 0.1223 |

### OOF decision results (raw threshold)

| Metric                   |    Result |
| ------------------------ | --------: |
| True Positives           |        84 |
| False Positives          |       161 |
| False Negatives          |        95 |
| True Negatives           |     1,815 |
| Review Flags             |       245 |
| Flag Rate                |    11.37% |
| Maximum Period Flag Rate |    11.75% |
| Recall                   |    46.93% |
| Precision                |    34.29% |
| Loss                     |     1,111 |
| Capacity Constraint      | Satisfied |

The highest observed period review rate (11.75%) stayed below the 12% limit.

### Regional diagnostic (OOF)

False-positive rate among negatives ranges from **6.31%** (eastern, 507 negatives) to **10.29%** (western, 486 negatives), a gap of **3.98 percentage points**. This is a descriptive diagnostic that calls for review; it is not a fairness certification.

---

# 💰 Cost Sensitivity

Changing the False Negative cost while keeping the selected threshold fixed:

| FN Cost | FP Cost |  Loss |
| ------: | ------: | ----: |
|       8 |       1 |   921 |
|      10 |       1 | 1,111 |
|      12 |       1 | 1,301 |

The official project policy remains `10 × FN + 1 × FP`.

---

# 🚀 Challenge Set Inference

The challenge labels were unavailable and were not used for training, model selection, threshold selection, calibration, evaluation, or performance reporting.

```text
Challenge rows = 2,500
Review capacity = 300
Initially eligible = 330
Final review flags = 300
Removed due to capacity = 30
Capacity utilization = 12%
```

Procedure:

1. Generate probabilities using the frozen final model.
2. Apply the calibrated threshold (0.1223).
3. Rank eligible applications by predicted probability.
4. Enforce the 12% capacity once on the full batch.
5. Apply the documented tie-handling policy (equal-score blocks are never split).

`decision = 1` is a **simulated review flag** only. Because challenge labels are unavailable, no challenge-set accuracy, precision, recall, AP, ROC-AUC, or loss is reported.

---

# 📐 Calibration

A sigmoid calibrator was learned on the reserved calibration period (July–September 2024): **836 rows, 78 positives**.

| Metric      |   Before |    After |
| ----------- | -------: | -------: |
| Brier Score | 0.0765 | 0.0781 |
| ECE         | 0.0211 | 0.0349 |
| Log Loss    | 0.2665 | 0.2773 |

These are **fit diagnostics computed on the same rows the calibrator was learned on**, so they are not independent calibration evidence. On these rows the sigmoid did not improve the raw Logistic Regression scores, and no calibration improvement is claimed for the unlabeled challenge set. Because the mapping is monotone, ranking metrics (ROC-AUC, AP) are unchanged.

---

# 🔍 Interpretability

The interpretability analysis (Day 4) explains the **Day 4 weighted LightGBM model**, not the final Logistic Regression model. It does not transfer automatically; explanations for the final model would need to be recomputed (for example from its coefficients).

* Permutation importance and global SHAP: `bureau_score` and `dti` lead.
* Local SHAP for `TR-009585`: `bureau_score` (+2.2693 log-odds) and `dti` (+1.1981) push the score up the most.
* SHAP values are in raw log-odds and are descriptive only: not causal, not a fairness certification, and not legal or compliance evidence.

---

# 📈 Stability & Monitoring

The project includes period-based checks and a customer-cluster bootstrap. Per batch, monitor:

* Flag volume against the 12% capacity (300 per 2,500 requests)
* Score distribution and prevalence (drift)
* Calibration (Brier and ECE on newly matured labels)
* Regional flag rates with their denominators
* Threshold stability

Retrain, recalibrate, and re-select the threshold together if any of these shift. Changes should be evaluated with new development evidence, not by tuning against the unlabeled challenge set.

---

# 🛠️ Technical Pipeline

```text
Synthetic Data
      ↓
Data Validation & Role Assignment
      ↓
Temporal / Customer-Aware Split
      ↓
Candidate Models
      ↓
Forward OOF Predictions
      ↓
Model Comparison
      ↓
Ensemble Evaluation
      ↓
Worth-It Gate
      ↓
Logistic Regression Selection
      ↓
Cost-Sensitive Threshold
      ↓
12% Capacity Constraint
      ↓
Calibration & Interpretability
      ↓
Final Model Refit
      ↓
Challenge Inference
      ↓
Submission + Provenance
```

---

# 📁 Repository Structure

```text
Tamweel-Lite/
│
├── artifacts/
│   ├── final_model/
│   ├── day5_ensemble_gate.json
│   ├── day5_oof_predictions.csv
│   ├── day5_oof_provenance.json
│   ├── day5_final_provenance.json
│   ├── final_metrics.json
│   ├── final_policy.json
│   └── environment.json
│
├── data/
│
├── presentation/
│   └── final_presentation.pdf
│
├── reports/
│   ├── DECISION_CARD.md
│   ├── ENSEMBLE_DECISION.md
│   ├── INTERPRETABILITY_REPORT.md
│   └── MODEL_CARD.md
│
├── submission/
│   ├── submission.csv
│   └── submission_manifest.json
│
├── 00_readiness_check.ipynb
├── 01_baseline_boosting.ipynb
├── 02_validation_tuning.ipynb
├── 03_cost_sensitive_decision.ipynb
├── 04_explain_calibrate.ipynb
├── 05_final_model.ipynb
├── 99_final_submission_check.ipynb
├── Tamweel_Lite v2.ipynb
│
├── data_contract.json
├── data_manifest.json
├── feature_dictionary.csv
├── tamweel_challenge.csv
├── tamweel_dirty.csv
├── tamweel_oof_matrix.csv
├── tamweel_train.csv
│
├── day4_shap_example.json
├── day4_shap_example.npz
├── day5_oof_example.json
├── LICENSE
└── README.md
```

---

# ⚡ Quick Start

## Environment

```text
Platform: Google Colab Free CPU
Python: see artifacts/environment.json
Seed = 211
n_jobs = 2
FAST_MODE = True
```

No paid compute, external API, GPU, or Google Drive dependency is required.

## Running the Project

The main learner notebook is `Tamweel_Lite v2.ipynb`, which brings together the five stages of the course:

```text
Day 1 — Baseline Boosting
Day 2 — Validation & Tuning
Day 3 — Imbalance, OOF Probabilities & Cost-Sensitive Decisions
Day 4 — Explainability, Calibration & Stability
Day 5 — Ensemble, Final Model & Challenge Inference
```

---

# 📦 Data & Provenance

The project uses **synthetic course data**. The repository contains the data contract, data manifest, feature dictionary, OOF provenance, final model provenance, and environment information.

---

# 📑 Reports & Supporting Evidence

* **Decision Card:** final model decision, threshold, capacity restriction, cost policy, and intended use.
* **Interpretability Report:** SHAP analysis, explanation scope, limitations, and stability evidence.
* **Ensemble Decision:** the ensemble comparison and the reasoning behind keeping a single Logistic Regression model.
* **Model Card:** intended use, limitations, validation approach, and associated risks.
* **Final Presentation:** problem and value, methodology, evidence, interpretation and calibration, and final decision with limitations.

---

# 🔁 Reproducibility

The repository preserves the random seed, environment and package versions, dataset roles, model configuration, validation structure, provenance, generated artifacts, final model, and submission manifest.

The final assessed version is identified by the Git tag **`Final`**.

---

# 🎯 Intended Use

This is an educational machine learning exercise using synthetic lending data. It demonstrates cost-sensitive classification, review prioritization, capacity-constrained decision making, model comparison, calibration analysis, explainability, and reproducible workflows.

The output is **not intended** to approve or reject real loans, make legally binding credit decisions, determine real-world creditworthiness, replace human decision makers, serve as a fairness or legal-compliance certificate, or establish causal relationships.

---

# ⚠️ Limitations

* The dataset is synthetic.
* Challenge labels are unavailable, so challenge-set performance cannot be measured.
* OOF development results come from three overlapping periods and are not an untouched final test set; fold SD is descriptive, not a confidence interval.
* Calibration diagnostics are computed on the calibrator's own fit rows and are not independent evidence.
* The interpretability analysis explains an earlier model (weighted LightGBM), not the final Logistic Regression.
* Regional analysis is descriptive and is not a fairness certification.
* The 12% capacity policy should be reassessed when new development data becomes available; Day 4 already showed one period exceeding capacity once near-threshold cases were added.
* Real-world deployment would require independent data, governance, monitoring, and domain-specific validation.

---

# 👤 Author

**Abdulaziz Aldhaayan**
**Student ID: S06**

Advanced Machine Learning Methods
SDAIA Academy

---

## License

This project is provided under the MIT License.
