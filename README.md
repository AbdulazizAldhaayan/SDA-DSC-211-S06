# Tamweel Lite — Advanced Machine Learning Methods

An end-to-end machine learning project developed as part of the **Advanced Machine Learning Methods** course at SDAIA Academy.

**Developer:** Abdulaziz Aldhaayan
**Student ID:** S06
**Course:** `SDA-DSC-211 — Advanced Machine Learning Methods | أساليب تعلم الآلة المتقدمة`

> **Note:** This is an educational learner project developed using synthetic course data. It is not an official SDAIA Academy repository.

---

## 📌 Project Overview

Tamweel Lite focuses on a simulated lending-review problem where the goal is to identify applications that should be prioritized for human review.

The main challenge is that failing to identify a risky application is considered much more costly than reviewing an application that turns out to be non-risky. At the same time, the number of applications that can be reviewed is limited.

The project therefore focuses on building a **cost-sensitive classification and decision-making workflow**, rather than simply maximizing prediction accuracy.

The model produces a **review flag** indicating whether an application should be considered for further human review. It does not make an actual loan approval or rejection decision.

---

## 🎯 Problem Definition

The central question addressed by the project is:

> Which applications should be selected for additional human review when missing a risky case has a significantly higher cost than reviewing a non-risky case?

The decision policy uses the following course-defined costs:

```text
False Negative cost = 10
False Positive cost = 1
Maximum review capacity = 12%
```

Therefore, the final decision objective can be represented as:

```text
Loss = 10 × FN + 1 × FP
```

while ensuring that the review workload remains within the allowed capacity.

---

## 🔎 Project Workflow

The complete machine learning pipeline includes:

1. Checking the available data and defining data roles.
2. Separating datasets using temporal and customer-aware rules.
3. Building baseline and candidate machine learning models.
4. Generating forward out-of-fold predictions.
5. Comparing Logistic Regression, XGBoost, and LightGBM.
6. Testing different ensemble approaches.
7. Applying the documented Worth-It Gate.
8. Selecting an appropriate cost-sensitive threshold.
9. Applying the 12% review-capacity constraint.
10. Evaluating calibration behavior.
11. Performing interpretability and stability analysis.
12. Retraining the selected final model.
13. Generating predictions for the challenge dataset.
14. Producing the final submission and supporting provenance artifacts.

---

# 📊 Model Comparison

The models were evaluated using nested out-of-fold predictions across three forward validation periods.

Average Precision was used as the primary comparison metric, while additional measures such as Brier Score, ECE, fold stability, and the ensemble decision criteria were also considered.

| Model               | Mean AP | Fold SD | Mean Brier | Mean ECE |
| ------------------- | ------: | ------: | ---------: | -------: |
| Logistic Regression | 0.39166 | 0.02981 |    0.06327 |  0.01882 |
| Weighted Ensemble   | 0.38942 | 0.02906 |    0.06332 |  0.01772 |
| Stacking            | 0.38314 | 0.02949 |    0.06603 |  0.03106 |
| Equal Ensemble      | 0.37170 | 0.03258 |    0.06435 |  0.02038 |
| XGBoost             | 0.35263 | 0.02904 |    0.06566 |  0.02276 |
| LightGBM            | 0.34549 | 0.04348 |    0.06608 |  0.02311 |

### Final Model

**Logistic Regression** was selected as the final model.

The tested ensemble approaches did not provide enough improvement to pass the project's documented Worth-It Gate. As a result, the simpler Logistic Regression model was retained.

---

# 🧪 Validation Strategy

The validation process was designed to reduce the risk of information leakage by considering both time and customer identity.

The development data was divided into different roles for fitting, selection, and calibration.

| Role            |  Rows | Customers | Positives |
| --------------- | ----: | --------: | --------: |
| Fit + Selection | 6,576 |     3,931 |       537 |
| Calibration     |   836 |       788 |        78 |
| Excluded        | 2,588 |         — |         — |

The nested forward OOF process generated:

* **2,155 OOF predictions**
* **3 forward validation periods**
* Warm-up observations without an outer OOF prediction were excluded from the model comparison.

The OOF results are considered development evidence and are **not treated as an untouched final test set**.

---

# ⚖️ Cost-Sensitive Decision Policy

After selecting the final model, a threshold was chosen while respecting the project's cost function and review-capacity requirement.

### Selected threshold

```text
0.168922
```

### OOF decision results

| Metric                   |    Result |
| ------------------------ | --------: |
| Threshold                |  0.168922 |
| True Positives           |        84 |
| False Positives          |       161 |
| False Negatives          |        95 |
| True Negatives           |     1,815 |
| Review Flags             |       245 |
| Flag Rate                |    11.37% |
| Maximum Period Flag Rate |    11.75% |
| Recall                   |    46.93% |
| Precision                |     8.15% |
| Loss                     |     1,111 |
| Capacity Constraint      | Satisfied |

The highest observed review rate was **11.75%**, which remained below the required 12% capacity limit.

---

# 💰 Cost Sensitivity

The effect of changing the False Negative cost was also examined while keeping the selected threshold fixed.

| FN Cost | FP Cost |  Loss |
| ------: | ------: | ----: |
|       8 |       1 |   921 |
|      10 |       1 | 1,111 |
|      12 |       1 | 1,301 |

This analysis is intended to show how the decision loss changes under different cost assumptions. The official project policy remains:

```text
10 × FN + 1 × FP
```

---

# 🚀 Challenge Set Inference

The challenge labels were not available and were therefore not used during:

* Training
* Model selection
* Threshold selection
* Calibration
* Evaluation
* Performance reporting

The final inference process was applied to:

```text
Challenge rows = 2,500
Review capacity = 300
Initially eligible = 330
Final review flags = 300
Removed due to capacity = 30
Capacity utilization = 12%
```

The inference procedure was:

1. Generate probabilities using the frozen final model.
2. Apply the selected threshold.
3. Rank eligible applications according to their predicted probabilities.
4. Enforce the maximum review capacity.
5. Apply the documented tie-handling policy.

A value of:

```text
decision = 1
```

represents a **simulated review flag** only.

Because the challenge labels were unavailable, no challenge-set accuracy, precision, recall, AP, ROC-AUC, or loss is reported.

---

# 📐 Calibration

Calibration was evaluated independently from the model-selection process.

The calibration dataset contained **836 observations**.

| Metric            |   Before |    After |
| ----------------- | -------: | -------: |
| ROC-AUC           | 0.789037 | 0.789037 |
| Average Precision | 0.789037 | 0.789037 |
| ECE               | 0.266489 | 0.277296 |
| Log Loss          | 0.076473 | 0.078058 |
| Brier Score       | 0.287803 | 0.287803 |

The calibration procedure did not show an improvement in the reported diagnostics. ECE and Log Loss increased slightly, while Brier Score, ROC-AUC, and Average Precision remained unchanged.

---

# 🔍 Interpretability

The repository also contains an interpretability analysis based on an earlier model version.

The analysis includes:

* Permutation importance
* Global SHAP explanations
* Local SHAP explanations
* Stability analysis
* Calibration analysis

For example, one local explanation showed contributions from features such as:

```text
bureau_score
dti
loan_amount_sar
```

The SHAP results are intended for descriptive interpretation only. They should not be considered causal explanations, fairness certification, or legal/compliance evidence.

---

# 📈 Stability & Monitoring

The project includes period-based stability checks and customer-cluster bootstrap analysis.

Potential metrics for future monitoring include:

* Average Precision
* Brier Score
* Expected Calibration Error
* Calibration stability
* Probability and score drift
* Changes in positive-class prevalence
* False Positive / False Negative trade-offs
* Review capacity
* Threshold stability

Any future change to the model or decision threshold should be evaluated using new development evidence rather than tuning against the unlabeled challenge dataset.

---

# 🛠️ Technical Pipeline

The main workflow can be summarized as follows:

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

The documented project workflow was executed using:

```text
Google Colab Free CPU
Python 3.13.16
Seed = 211
n_jobs = 2
FAST_MODE = True
```

The project does not require paid compute, an external API, a GPU, or a Google Drive dependency for the documented workflow.

## Running the Project

The main learner notebook is:

```text
Tamweel_Lite v2.ipynb
```

The notebook brings together the five stages of the course:

```text
Day 1 — Baseline Boosting
Day 2 — Validation & Tuning
Day 3 — Imbalance, OOF Probabilities & Cost-Sensitive Decisions
Day 4 — Explainability, Calibration & Stability
Day 5 — Ensemble, Final Model & Challenge Inference
```

---

# 📦 Data & Provenance

The project is based on **synthetic course data**.

The challenge labels were intentionally unavailable and were not used for model training, selection, threshold optimization, calibration, or evaluation.

The repository contains supporting provenance and reproducibility artifacts, including:

* Data contract
* Data manifest
* Feature dictionary
* OOF provenance
* Final model provenance
* Environment information

---

# 📑 Reports & Supporting Evidence

The repository includes several supporting documents:

### Decision Card

Summarizes the final model decision, threshold, capacity restriction, cost policy, and intended use.

### Interpretability Report

Documents the SHAP analysis, explanation scope, limitations, and stability evidence.

### Ensemble Decision

Explains the ensemble comparison and the reasoning behind retaining a single Logistic Regression model.

### Model Card

Describes the model's intended use, limitations, validation approach, and associated risks.

### Final Presentation

The final presentation covers:

1. Problem and business value
2. Methodology
3. Model evidence and selection
4. Interpretation and limitations
5. Final decision and next steps

---

# 🔁 Reproducibility

Reproducibility is an important part of this project.

The repository preserves information related to:

* Random seed
* Environment and package versions
* Dataset roles
* Model configuration
* Validation structure
* Provenance
* Generated artifacts
* Final model
* Submission manifest

The final assessed version should be identified through the corresponding Git commit and preserved project artifacts.

---

# 🎯 Intended Use

This project is an educational machine learning exercise using synthetic lending data.

It demonstrates:

* Cost-sensitive classification
* Review prioritization
* Capacity-constrained decision making
* Model comparison
* Calibration analysis
* Explainability
* Reproducible machine learning workflows

The output is **not intended** to:

* Approve or reject real loans
* Make legally binding credit decisions
* Determine real-world creditworthiness
* Replace human decision makers
* Serve as a fairness or legal-compliance certificate
* Establish causal relationships from model explanations

---

# ⚠️ Limitations

Several limitations should be considered:

* The dataset is synthetic.
* Challenge labels are unavailable.
* Challenge-set performance therefore cannot be measured directly.
* OOF development results are not an untouched final test set.
* Calibration-fit diagnostics should not be interpreted as independent calibration evaluation.
* The interpretability analysis is based on an earlier model version.
* Regional analysis is descriptive and is not a fairness certification.
* The 12% capacity policy should be reassessed when new development data becomes available.
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
