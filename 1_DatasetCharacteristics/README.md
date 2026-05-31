# Dataset Characteristics

**[Notebook](exploratory_data_analysis.ipynb)**

## Dataset Information

### Dataset Source
- **Dataset Link:**
- IMF: https://data.imf.org/en
- Monthly default and restructuring database by Asonuma and Trebesch (2016, JEEA): https://sites.google.com/site/christophtrebesch/data


### Dataset Characteristics
- **Number of Observations:** Approximately 126,900 monthly observations (country-month panel). Temporal resolution: monthly (1975-M01 to 2020-M12). 237 countries, of which 92 have at least one default event.
- **Number of Features:** 40 features total: 4 raw macro variables (GS10, DEBT_GDP, EXPORTS to WORLD, IMPORTS from WORLD) + 36 engineered features (GS10 changes at 1/3/6/12 months, percent changes, log-differences, rolling std dummy, asymmetric shocks, interaction term, debt/GDP changes, export growth, lag features at 1/3/6/12/24 months)

### Target Variable/Label
- **Label Name:** DEFAULT
- **Label Type:** Binary Classification
- **Label Description:** DEFAULT = 1 indicates that a country is in a sovereign default or debt restructuring process in that month. The prediction task is: given macro-economic conditions and Synthetic Control treatment effects, predict the probability that a country will enter a default or restructuring process.
- **Label Values:** DEFAULT = 0: No default or restructuring (majority class, ~85% of observations). DEFAULT = 1: Country is in default or active restructuring process (~15% of observations, 7,574 default-months across 92 countries). Sub-categories tracked separately: strictly preemptive (no missed payments), weakly preemptive (some missed payments), post-default restructuring.
- **Label Distribution:** Imbalanced dataset: approximately 85% non-default (DEFAULT=0) and 15% default (DEFAULT=1). The imbalance is handled via scale_pos_weight in XGBoost and Precision-Recall AUC as the primary evaluation metric rather than accuracy.

### Feature Description
Features are grouped into four categories as described below.

**Example format:**
- **Feature 1 (feature_name):** DEBT_GDP (float): Central government debt as percentage of GDP (%). Coverage: 64.7% of observations. Annual data from IMF and World Bank, forward-filled to monthly frequency. Range: approximately 5% to 300%+ for heavily indebted countries.
- **Feature 2 (feature_name):** [Description of what this feature represents, data type, and any relevant details]
- **Feature Group (group_name):** GS10 Engineering (12 features): Changes at 1, 3, 6, and 12 month horizons (absolute and percent), log-difference over 12 months, rolling standard deviation dummy (=1 if monthly change exceeds 36-month rolling std), asymmetric shock indicators (gs10_increase_12, gs10_decrease_12), and interaction term gs10_x_debt_gdp = gs10_change_12 × DEBT_GDP. Lag features (10 features): DEBT_GDP and EXPORTS lagged at 1, 3, 6, 12, and 24 months to capture persistence and delayed effects.
  
## Exploratory Data Analysis

The exploratory data analysis is conducted in the [exploratory_data_analysis.ipynb](exploratory_data_analysis.ipynb) notebook, which includes:

- Data loading and initial inspection
- Statistical summaries and distributions
- Missing value analysis
- Feature correlation analysis
- Data visualization and insights
- Data quality assessment
