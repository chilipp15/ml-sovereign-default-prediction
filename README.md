## Description

This project predicts sovereign debt defaults using a novel combination of
Synthetic Control methods and Machine Learning. For each country, a weighted
combination of structurally similar donor countries (identified via Affinity
Propagation clustering) is constructed as a counterfactual twin. Rolling
36-month OLS residuals measure the Mahalanobis distance (`sc_distance`)
between a country's actual macro path and its synthetic twin. A significant
deviation from the twin serves as a causal early warning signal for sovereign
default. This SC-distance feature is then combined with macro variables and
a Two-Tier Logit + XGBoost architecture to predict sovereign defaults
12 months in advance.

**Task Type:** Binary Classification (Sovereign Default Prediction)
with Causal Inference (Synthetic Controls · Average Marginal Effects)

---

## Results Summary

### Best Model Performance

| | |
|---|---|
| **Best Model** | XGBoost Full (Two-Tier: Logit + SC-Residuals) |
| **Primary Metric** | AUC-ROC |
| **Final Performance** | AUC-ROC = 0.764 · PR-AUC = 0.019 |
| **Usefulness Score** | U(0.5) = 0.204 at threshold 0.15 (Alessi & Detken 2011) |
| **Optimal Threshold** | 0.15 |
| **Test Period** | 2011–2020 · N = 25,343 · Default rate = 0.62% |

### Model Comparison

| Model | AUC-ROC | PR-AUC | Notes |
|---|---|---|---|
| SC-Distance standalone EWS | 0.554 | — | Near-random; SC alone insufficient |
| Logit Baseline (Tier 1, 10 features) | 0.638 | 0.011 | Linear macro benchmark |
| XGBoost Baseline (Macro only) | 0.737 | 0.019 | No SC-residuals |
| **XGBoost Full (Macro + SC-Residuals)** | **0.764** | **0.019** | **Best model** |

**Improvement over XGBoost Baseline:**
+0.027 AUC-ROC · Bootstrap 95% CI [+0.003, +0.049] · LR-test p < 0.001

**Out-of-Time Stability:**

| Period | AUC-ROC | PR-AUC |
|---|---|---|
| 2011–2015 | 0.733 | 0.021 |
| 2016–2020 | 0.796 | 0.021 |

Model improves over time, confirming generalisation beyond training distribution.

---

### Key Insights

**Most Important Features (SHAP ranking, Full Model):**

| Rank | Feature | SHAP | Economic Interpretation |
|---|---|---|---|
| 1 | `logit_pred` | 36.6% | Tier-1 compressed macro signal |
| 2 | `gs10_lag1` | GS10 interest rate level | External financing cost |
| 3 | `cpi3` | Inflation (3-month lag) | Macro instability signal |
| 4 | `export_lag12` | Export volume | FX revenue capacity |
| 5 | `debt_gdp_lag12` | Debt/GDP level | Debt sustainability |
| 6–15 | `sc_distance_*` | SC-residual features | Deviation from synthetic twin |

**Causal Marginal Effect (AME):**
Moving one decile higher in SC-distance rank increases
the 12-month default probability by +0.074 PP (p = 0.003),
controlling for Debt/GDP, GS10, CPI, and exports.
Countries in the highest SC-deviation decile face a
**+142% higher relative default probability**
compared to the lowest decile
(baseline: 0.52% → top decile: ~1.04%).

---

**Model Strengths:**

- Combines predictive accuracy (XGBoost AUC-ROC = 0.764) with
  causal interpretability (AME from Logit) — each method does
  what it does best
- Two-Tier architecture: Logit captures linear macro effects
  and compresses them into `logit_pred`; XGBoost corrects
  non-linearities and interaction effects beyond the Logit
- SC-residuals provide a country-specific deviation signal
  that is orthogonal to standard macro indicators —
  confirmed by significant LR-test (p < 0.001)
- Out-of-time AUC improves from 0.733 to 0.796 across
  the two test sub-periods: no overfitting to historical crises
- Usefulness Score U = 0.204 confirms practical value
  beyond naive strategies (always warn / never warn)
- Positive Bootstrap CI [+0.003, +0.049] confirms the
  SC-residual lift is not driven by sampling variance

---

**Model Limitations:**

- Debt/GDP data coverage is 64.7%, limiting donor pool
  quality for some countries and reducing SC-fit reliability
- SC-distance alone is near-random as a standalone EWS
  (AUC = 0.554): the signal only emerges in combination
  with macro features inside XGBoost
- Strong distribution shift between training (1975–2009)
  and test period (2011–2020): GS10 fell from ~7.4% to ~2.4%,
  reducing generalisability of interest-rate features
- Only 169 of ~214 countries have SC-residuals computed,
  requiring median imputation for the remaining countries
- Logit coefficients for CPI and GS10-shock show unexpected
  signs due to selection bias (CPI missing for poorest
  defaulters) and delayed default timing after rate shocks

---

**Business Impact:**

This model provides an actionable early warning system for
sovereign defaults, 12 months in advance:

- **Sovereign bond investors:** risk differentiation across
  emerging markets; countries in the top SC-deviation decile
  are 2.5× as risky as the average country
- **IMF / World Bank:** systematic monitoring of countries
  that deviate from their structural peer group —
  a signal not captured by standard macro surveillance
- **Policymakers in emerging markets:** early identification
  of debt trajectory divergence allows for pre-emptive
  fiscal adjustment before spreads widen
- **Threshold 0.15** balances false alarms (FPR = 0.222)
  against missed defaults (FNR = 0.371), appropriate for
  a policymaker who prioritises catching defaults
  over minimising false warnings

## Documentation

1. **[Literature Review](0_LiteratureReview/README.md)**
2. **[Dataset Characteristics](1_DatasetCharacteristics/exploratory_data_analysis.ipynb)**
3. **[Baseline Model](2_BaselineModel/baseline_model.ipynb)**
4. **[Model Definition and Evaluation](3_Model/model_definition_evaluation)**
5. **[Presentation](4_Presentation/README.md)**

## Cover Image

![Project Cover Image](CoverImage/cover_image.png)
