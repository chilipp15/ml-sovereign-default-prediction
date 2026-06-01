# Predicting Sovereign Defaults Using Machine Learning and Synthetic Controls

## Repository Link

[https://github.com/chilipp15/ml-sovereign-default-prediction]

## Description
This project predicts sovereign debt defaults using a novel combination of Synthetic Control methods and Machine Learning. Synthetic twins are generated on country-time-specific features. Using residuals aims to identify at what point countries behave differently compared to their synthetic twin. This might be useful as an early warning indicator for default. The early warning indicator is then used to predict sovereign default in a XGBoost estimation

### Task Type

Binary Classification (Sovereign Default Prediction) with Causal Inference (Synthetic Controls)

### Results Summary

#### Best Model Performance
- **Best Model:** [Name and type of the best-performing model"]
- **Evaluation Metric:** [Primary metric used, e.g., Accuracy, F1-Score, MSE, MAE]
- **Final Performance:** [Best score achieved, e.g., 95% accuracy, F1-score of 0.87, MSE of 0.12]

#### Model Comparison
- **Baseline Performance:** [Baseline model performance for comparison]
- **Improvement Over Baseline:** [Quantitative improvement, e.g., "+12% accuracy", "25% reduction in MSE"]
- **Best Alternative Model:** [Second-best model and its performance]

#### Key Insights (what I hope for)
- **Most Important Features:** DEBT_GDP, gs10_change_12, gs10_x_debt_gdp (interaction term: GS10 × Debt/GDP), tau_cumul (cumulative Synthetic Control treatment effect), debt_gdp_change
- **Model Strengths:** Captures the non-linear interaction between rising US interest rates and high debt-to-GDP ratios as a default predictor. The Synthetic Control component provides a causal interpretation of the treatment effect τ, distinguishing countries that deviate significantly from their counterfactual path
- **Model Limitations:** Debt/GDP data coverage is 64.7%, limiting the donor pool quality for some countries. The GS10 change variables are not perfectly exogenous shocks (no AR-residual purification), so τ should be interpreted as a predictive rather than strictly causal effect. Countries with fewer than 12 months of pre-default data are excluded.
- **Business Impact:** Early warning system for sovereign defaults: identifies countries whose debt trajectory deviates significantly from their synthetic counterfactual after GS10 increases. Relevant for sovereign bond investors, IMF/World Bank early warning systems, and policymakers in emerging markets.

## Documentation

1. **[Literature Review](0_LiteratureReview/README.md)**
2. **[Dataset Characteristics](1_DatasetCharacteristics/exploratory_data_analysis.ipynb)**
3. **[Baseline Model](2_BaselineModel/baseline_model.ipynb)**
4. **[Model Definition and Evaluation](3_Model/model_definition_evaluation)**
5. **[Presentation](4_Presentation/README.md)**

## Cover Image

![Project Cover Image](CoverImage/cover_image.png)
