## Model Selection

## !!!! Werte müssen noch angepasst werden !!!!!

**Baseline Model Type:** Two-Tier Architecture — Synthetic Controls (Abadie 2003) for counterfactual estimation + XGBoost for prediction

**Rationale:**
Sovereign default prediction requires both predictive accuracy and causal interpretability. The pipeline combines two complementary approaches:

1. **Synthetic Controls (Tier 1 – Causal):** For each country, a weighted combination of structurally similar donor countries (identified via Affinity Propagation clustering) is constructed as a counterfactual twin. Rolling 36-month OLS residuals measure the Mahalanobis distance between a country's actual macro path and its synthetic twin (`sc_distance`). A large deviation from the twin signals that a country is behaving anomalously relative to its peers — the core early warning hypothesis.

2. **Two-Tier Logit + XGBoost (Tier 2 – Predictive):** A logistic regression (Tier 1) compresses 10 macro features (one per variable family: Debt/GDP level, Debt/GDP dynamics, export level, export growth, GS10 level, GS10 shock, CPI, exchange rate) into a single probability score `logit_pred`. XGBoost (Tier 2) combines `logit_pred` with SC-residual features and all macro features to produce the final 12-month default probability.

---

## Model Performance

**Target Variable:** `DEFAULT_ONSET_12M` — equals 1 in the 12 months before a sovereign default announcement (Asonuma & Trebesch 2016), 0 otherwise. Running default months are excluded from training to prevent label contamination.

**Primary Evaluation Metric:** AUC-ROC (Area Under the ROC Curve)

| Model | AUC-ROC | PR-AUC |
|---|---|---|
| Logit Baseline (Tier 1, 10 features) | 0.638 | 0.011 |
| XGBoost Baseline (Macro only) | 0.737 | 0.019 |
| XGBoost Full (Macro + SC-Residuals) | 0.764 | 0.019 |
| SC-Distance standalone EWS | 0.554 | — |

**SC-Residual Lift:** +0.027 AUC-ROC  
**Bootstrap 95% CI:** [+0.003, +0.049] — statistically significant  
**LR-Test p-value:** < 0.001 — SC-residuals jointly highly significant  
**Usefulness Score U(0.5):** 0.204 at threshold 0.15 (Alessi & Detken 2011)  
**Optimal Classification Threshold:** 0.15 (maximises Usefulness Score)

**Causal Marginal Effect (AME):**  
Moving one decile higher in SC-distance rank increases the 12-month default probability by +0.074 PP (p = 0.003), controlling for Debt/GDP, GS10, CPI, and exports. Countries in the highest SC-deviation decile face a **+142% higher relative default probability** compared to the lowest decile.

---

## Cross-Validation Strategy

Given the temporal nature of the data, standard k-fold cross-validation is not applicable (it would leak future information into training). Instead:

- **Out-of-Time Validation** across two sub-periods of the test set:

| Period | N | Default Rate | AUC-ROC | PR-AUC |
|---|---|---|---|---|
| 2011–2015 | 12,664 | 0.60% | 0.733 | 0.021 |
| 2016–2020 | 12,679 | 0.65% | 0.796 | 0.021 |

AUC-ROC improves from 0.733 to 0.796 across the two sub-periods, confirming temporal stability and generalization to new default events beyond the training distribution.

---

## Evaluation Methodology

**Data Split:**

| Set | Period | N (approx.) | Default Rate |
|---|---|---|---|
| Training | 1975–2009 | 76,000 | 2.26% |
| Gap | 2010 | — | excluded |
| Test | 2011–2020 | 25,343 | 0.62% |

The 12-month gap between training end (2009-12) and test start (2011-01) prevents leakage from the 12-month lag features used in `DEFAULT_ONSET_12M` construction.

Running default months (DEFAULT_DB = 1, DEFAULT_ONSET_12M = 0) are excluded from the training sample because macro variables recover during prolonged restructurings, creating contradictory labels.

**Evaluation Metrics:**

| Metric | Justification |
|---|---|
| **AUC-ROC** | Standard metric for binary classifiers; measures ranking quality independent of threshold; appropriate for imbalanced classes |
| **PR-AUC** | More informative than AUC-ROC at severe class imbalance (2% positive rate); measures precision-recall trade-off directly relevant for rare events |
| **Usefulness Score U(μ)** | Standard EWS metric (Alessi & Detken 2011); μ = 0.5 symmetric, μ = 0.3 penalises missed defaults more heavily (policymaker perspective); positive U confirms the model outperforms naive strategies |
| **Bootstrap 95% CI** | Quantifies uncertainty around the AUC-ROC lift from SC-residuals; 1,000 bootstrap samples; CI excluding zero confirms the lift is not due to sampling variance |
| **LR-Test** | Likelihood-ratio test for joint significance of SC-residual features; p < 0.001 confirms SC-features contribute beyond macro variables |
| **AME (Average Marginal Effect)** | Econometric causal quantification of the SC-deviation effect; p = 0.003; distinguishes prediction (XGBoost) from causal inference (Logit AME) |

**Metric Practical Relevance:**

- **AUC-ROC = 0.764** means: if a policymaker randomly picks one country-month that defaulted and one that did not, the model correctly identifies the riskier one in 76.4% of cases — versus 50% for a random guess.

- **Usefulness Score U = 0.204** means: using the model at threshold 0.15 generates more value than either never warning or always warning. At threshold 0.15, FPR = 0.222 and FNR = 0.371 — the model catches ~63% of defaults while flagging roughly 1 in 5 non-default months as at-risk.

- **AME +0.074 PP per decile** means: for every step a country moves up in SC-deviation rank, its estimated default probability increases by 0.074 percentage points — a +142% relative increase from the bottom to the top decile. This provides actionable risk differentiation beyond standard macro indicators.

- **Out-of-Time improvement (0.733 → 0.796)** means: the model does not overfit to historical crises (e.g. Latin American debt crises of the 1980s) but generalises to more recent events (Ecuador 2008, Greece 2012, Mozambique 2017, Lebanon 2020).

---

## Next Steps

This baseline model serves as a reference point for evaluating more sophisticated models in the [Model Definition and Evaluation](https://github.com/chilipp15/ml-sovereign-default-prediction/blob/main/3_Model/README.md) phase. Potential extensions include:

- Hyperparameter optimisation of XGBoost via time-series cross-validation
- Alternative SC distance measures (e.g. standardised residuals instead of Mahalanobis)
- Extension of the forecast horizon to 24 months as robustness check
- Country-specific analysis for the highest-performing predictions
