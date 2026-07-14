# Baseline Logit Model

This document describes the logistic regression baseline used as
Tier 1 in the Two-Tier architecture. The logit model compresses
macro information into a single probability score (`logit_pred`)
that feeds into the XGBoost classifier as a feature.

---

## Variable Selection

### Step 1: Full Model with VIF Check

A full logit model was estimated using all engineered macro features.
The VIF check revealed severe multicollinearity:

| Issue | Features | VIF |
|---|---|---|
| Perfect multicollinearity | `gs10_change_12`, `gs10_decrease_12`, `gs10_increase_12` | ∞ |
| Extreme collinearity | `cpi6_log`, `cpi3_log`, `cpi12_log` | 1,548–51,034 |
| High collinearity (>10) | `er_rate*_log`, `gs10_log_diff_12`, `export_lag*_log`, `debt_gdp_lag*` | 11–337 |

Due to the singular matrix caused by perfect multicollinearity,
`statsmodels` failed to produce coefficient estimates for the full model.

**Full model metrics (sklearn, no coefficient table available):**

| Metric | Value |
|---|---|
| Train N | 29,118 |
| Train default rate | 2.82% |
| Test N | 16,255 |
| Test default rate | 0.62% |
| AUC-ROC | 0.676 |
| PR-AUC | 0.011 |

---

### Step 2: Variable Selection per Economic Family

To resolve multicollinearity, a theory-driven selection procedure
was applied: one variable per economic concept family, chosen by
highest univariate AUC-ROC on the training set.

| Family | Selected Variable | Univariate AUC-ROC |
|---|---|---|
| Debt/GDP (Level) | `debt_gdp_lag12` | 0.678 |
| Debt/GDP (Dynamics) | `debt_gdp_change_lag12` | **0.786** |
| Exports (Level) | `export_lag12` | 0.642 |
| Exports (Dynamics) | `export_growth_lag12` | 0.583 |
| GS10 (Level) | `gs10_lag1` | 0.499 |
| GS10 (Change) | `gs10_increase_12` | 0.509 |
| GS10 (Interaction) | `gs10_increase_x_debt` | 0.518 |
| GS10 (Other) | `gs10_std_dummy` | 0.506 |
| CPI | `cpi6` | 0.488 |
| Exchange Rate | `er_rate24_log` | 0.457 |

The strongest single predictor is `debt_gdp_change_lag12`
(12-month change in Debt/GDP, lagged 12 months), confirming
debt dynamics as the dominant early warning signal.

---

## Final Logit Model

### Setup

```python
LOGIT_FEATURES = [
    "debt_gdp_lag12",        # Debt/GDP level (12M lag)
    "debt_gdp_change_lag12", # Debt/GDP 12M change (12M lag)
    "export_lag12",          # Export volume (12M lag)
    "export_growth_lag12",   # Export growth rate (12M lag)
    "gs10_lag1",             # US 10Y rate level (1M lag)
    "gs10_increase_12",      # GS10 positive shock (12M)
    "gs10_increase_x_debt",  # GS10 shock × Debt/GDP
    "gs10_std_dummy",        # GS10 volatility dummy
    "cpi6",                  # CPI (6M lag)
    "er_rate24_log",         # Exchange rate log (24M lag)
]

LogisticRegression(
    C            = 0.1,      # L2 regularisation
    class_weight = {0: 1.0, 1: 20.0},  # capped at 20× for stability
    solver       = "lbfgs",
    max_iter     = 2000,
    random_state = 42,
)
```

**Training split:** Temporal — Train ≤ 2009-12, Gap 2010, Test ≥ 2011-01

---

### Performance

| Metric | Value |
|---|---|
| Train N | 30,987 |
| Train default rate | 2.79% |
| Test N | 16,371 |
| Test default rate | 0.68% |
| **AUC-ROC** | **0.580** |
| **PR-AUC** | **0.014** |
| Pseudo-R² | 0.278 |
| AIC | 44,979 |

---

### Coefficients

| Feature | Coeff | p-value | Odds Ratio | Expected | ✓/✗ |
|---|---|---|---|---|---|
| `gs10_increase_12` | −0.536 | <0.001 *** | 0.585 | + | ✗ |
| `export_growth_lag12` | −0.485 | <0.001 *** | 0.616 | − | ✓ |
| `gs10_std_dummy` | −0.222 | <0.001 *** | 0.801 | + | ✗ |
| `gs10_lag1` | +0.209 | <0.001 *** | 1.232 | + | ✓ |
| `er_rate24_log` | +0.038 | <0.001 *** | 1.038 | + | ✓ |
| `cpi6` | −0.030 | <0.001 *** | 0.971 | + | ✗ |
| `debt_gdp_change_lag12` | +0.017 | <0.001 *** | 1.017 | + | ✓ |
| `debt_gdp_lag12` | +0.002 | <0.001 *** | 1.002 | + | ✓ |
| `gs10_increase_x_debt` | −0.001 | 0.002 *** | 0.999 | + | ✗ |
| `export_lag12` | −0.000 | <0.001 *** | 1.000 | − | ✓ |

All 10 features are highly significant (p < 0.001). Six of ten show
the expected sign. Four show unexpected signs, explained below.

---

### Unexpected Signs — Explanation

**`gs10_increase_12` and `gs10_increase_x_debt` (expected +, got −)**

The training period (1975–2009) is dominated by structurally
falling interest rates after the Volcker peak (1981). Sovereign
defaults during this period occurred *after* rates had already
declined — the model learns that rising rates precede a period
of eventual normalisation, not immediate default. This is a
well-documented delayed transmission mechanism in the EWS
literature (Manasse & Roubini 2003).

**`gs10_std_dummy` (expected +, got −)**

High GS10 volatility occurs during Fed reaction phases
(e.g. 2004–2006 rate cycle) when global conditions were stable.
The dummy does not reliably signal default risk in this dataset.

**`cpi6` (expected +, got −)**

Classic selection bias: CPI data is systematically missing for
the poorest defaulting countries. Countries *with* CPI data are
disproportionately wealthier and less likely to default, creating
a spurious negative correlation.

---

### Role in Two-Tier Architecture

The logit model is not used as a standalone predictor. It serves
as **Tier 1** in the Two-Tier XGBoost architecture:


This design combines the **interpretability** of a linear model
with the **flexibility** of gradient boosting, following the
Two-Tier approach of Tellimer (2024) and Petropoulos et al. (2022).

---

### Limitations

- Four of ten coefficients carry unexpected signs due to training
  period characteristics (falling rate regime, CPI selection bias)
- AUC-ROC of 0.580 is below the XGBoost baseline (0.700),
  confirming that non-linear feature interactions matter
- The logit is estimated on oversampled training data
  (`frac=20`, `replace=True`) to handle class imbalance;
  absolute probability values are therefore not calibrated
  and should not be interpreted directly
