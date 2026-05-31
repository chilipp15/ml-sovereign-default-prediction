# Baseline Model

**[Notebook](baseline_model.ipynb)**

## Baseline Model Results

### Model Selection
- **Baseline Model Type:** Random Forest or Ridge Regression for Clustering and XGBoost for Prediction
- **Rationale:** Use of best fitting method for clustering and creating best synthetic twins. XGBoost is planned to be used for the prediction of default.

### Model Performance
- **Evaluation Metric:** [e.g., Accuracy, F1-Score, Precision, Recall, MSE, MAE, R², etc.]
- **Performance Score:** Classical Synthetic Control (Abadie): Pre-R² target 0.70–0.92. XGBoost baseline: to be determined after full pipeline execution.
- **Cross-Validation Score:** [Mean and standard deviation of CV scores, e.g., 0.82 ± 0.03]

### Evaluation Methodology
- **Data Split:** Temporal split: Train 1975–2009, Gap 2010 (12-month buffer to prevent leakage from 12-month lag features), Test 2011–2020
- **Evaluation Metrics:** [List all metrics used and justify why they are appropriate for this problem]

### Metric Practical Relevance
[Explain the practical relevance and business impact of each chosen evaluation metric. How do these metrics translate to real-world performance and decision-making? What do the metric values mean in the context of your specific problem domain?]

## Next Steps
This baseline model serves as a reference point for evaluating more sophisticated models in the [Model Definition and Evaluation](../3_Model/README.md) phase.
