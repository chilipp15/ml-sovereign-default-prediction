# Dataset Characteristics

`analysis_df` is the primary panel dataset used throughout this project.
It is constructed by merging six independent data sources into a unified
country-month panel spanning 1975 to 2020.

---

## Construction Pipeline

The dataset is built in `generate_default_dataset.ipynb`. The pipeline
follows seven steps:

1. Load and clean each raw source independently
2. Expand annual Debt/GDP data to monthly frequency (constant within year)
3. Build a unified country-time frame from all sources (left-join anchor)
4. Merge default events onto the frame
5. Merge GS10 (global, time only — same value for all countries per month)
6. Merge trade, Debt/GDP, exchange rates, and CPI sequentially
7. Resolve debt-type country name variants and fill macro values from
   the corresponding standard country row

> **Note on manual changes:** Between the `generate_default_dataset.ipynb`
> code and the `analysis_df` used in the main analysis, manual changes were
> made in the intermediate Excel files. These changes primarily concern
> **country name standardisation** to improve merging between the
> DEFAULT_DATABASE (Asonuma & Trebesch 2016) and the macro data sources.
> Country names in the DEFAULT_DATABASE often include debt-type suffixes
> (e.g. `"Ukraine (Chase loan)"`) or differ in spelling from the IMF/BIS
> naming conventions used in the macro sources. These were harmonised
> manually and are not fully captured in the notebook code.

---

## Data Sources

| Variable | Source | Frequency | Coverage |
|---|---|---|---|
| `DEBT_GDP` | Global Debt Database (GDD) — IMF | Annual → expanded to monthly | ~190 countries, 1950–2020 |
| `GS10` | US 10-Year Treasury Constant Maturity Rate — FRED | Monthly | Global (one value per month) |
| `IMPORTS from WORLD` | IMF Direction of Trade Statistics (DOTS) | Monthly | ~200 countries |
| `EXPORTS to WORLD` | IMF Direction of Trade Statistics (DOTS) | Monthly | ~200 countries |
| `er_rate` | IMF International Financial Statistics (IFS) | Monthly | ~180 countries |
| `cpi` | IMF International Financial Statistics (IFS) | Monthly | ~180 countries |
| `DEFAULT` | Sovereign Debt Restructuring Database — Asonuma & Trebesch (2016) | Event-level → monthly | 186 restructuring events, 1950–2015 |

---

## Raw Input Files

| File | Description |
|---|---|
| `GDD_annual.csv` | Annual Debt/GDP ratios per country (IMF GDD) |
| `GS10_monthly.csv` | Monthly US 10-Year Treasury rate (FRED) |
| `monthly_trade.csv` | Monthly imports and exports per country (IMF DOTS) |
| `DEFAULT_DATABASE.xlsx` | Sovereign restructuring events (Asonuma & Trebesch 2016) |
| `exchange_rates.csv` | Monthly bilateral exchange rates per country (IMF IFS) |
| `CPI_data.csv` | Monthly CPI index per country (IMF IFS) |

---

## Final Variables

| Column | Type | Description |
|---|---|---|
| `COUNTRY` | string | Country name (standardised) |
| `TIME` | string | Month in format `YYYY-Mmm` (e.g. `2001-M11`) |
| `DEFAULT` | int (0/1) | 1 during active restructuring period |
| `DEFAULT_DB` | int (0/1) | 1 during restructuring (from DEFAULT_DATABASE merge) |
| `DEFAULT_ONSET` | int (0/1) | 1 in announcement month only |
| `DEFAULT_ONSET_12M` | int (0/1) | 1 in the 12 months before announcement **(target variable)** |
| `Strictly preemptive` | int (0/1) | Restructuring type flag |
| `Weakly preemptive` | int (0/1) | Restructuring type flag |
| `Post-default` | int (0/1) | Restructuring type flag |
| `IMPORTS from WORLD` | float | Monthly imports in USD |
| `EXPORTS to WORLD` | float | Monthly exports in USD |
| `GS10` | float | US 10-Year Treasury rate (%) |
| `DEBT_GDP` | float | Gross government debt as % of GDP |
| `er_rate` | float | Exchange rate (local currency per USD) |
| `cpi` | float | Consumer Price Index |

---

## Dataset Characteristics

| Characteristic | Value |
|---|---|
| **Unit of observation** | Country-month |
| **Time period** | January 1975 – December 2020 |
| **Total observations** | ~113,700 country-months |
| **Number of countries** | ~214 |
| **Default events** | 186 restructurings (Asonuma & Trebesch 2016) |
| **Target variable** | `DEFAULT_ONSET_12M` |
| **Positive target months** | 2,118 (DEFAULT_ONSET_12M = 1) |
| **Positive rate (full sample)** | ~1.86% |
| **Positive rate (training set)** | ~2.26% |
| **Training period** | 1975-01 – 2009-12 |
| **Gap (excluded)** | 2010 (12-month buffer) |
| **Test period** | 2011-01 – 2020-12 |

---

## Data Coverage Notes

Coverage varies substantially across variables and countries:

| Variable | Approx. coverage |
|---|---|
| `GS10` | 100% (global rate, no missing) |
| `EXPORTS to WORLD` | ~75% |
| `IMPORTS from WORLD` | ~75% |
| `DEBT_GDP` | ~64.7% |
| `er_rate` | ~70% |
| `cpi` | ~68% |

Low `DEBT_GDP` coverage is the binding constraint for the Synthetic Control
donor pool construction: countries with insufficient Debt/GDP history are
excluded from donor pools, resulting in ~38 countries without SC-residuals
in the final analysis dataset.

---

## Country Name Standardisation

Country name matching was the primary challenge during dataset construction.
The DEFAULT_DATABASE uses original restructuring-event names that often
include debt-type suffixes or differ from IMF naming conventions.
The following mappings were applied **in code**:

| DEFAULT_DATABASE name | Standardised name |
|---|---|
| `Chad (Glencore loans)` | `Chad` |
| `Moldova (Eurobonds)` | `Moldova, Republic of` |
| `Moldova (Gazprom debt)` | `Moldova` |
| `Mozambique` | `Mozambique, Republic of` |
| `Republic of Mozambique (EMATUM Notes)` | `Mozambique, Republic of` |
| `Pakistan (bank debt)` | `Pakistan` |
| `Russia` | `Russian Federation` |
| `Russia (GKOs, non-resid.)` | `Russian Federation` |
| `Serbia and Montenegro` | `Serbia, Republic of` |
| `Tanzania` | `Tanzania, United Republic of` |
| `Turkey` | `Türkiye, Republic of` |
| `Ukraine (Chase loan)` | `Ukraine` |
| `Ukraine (Eurobonds)` | `Ukraine` |
| `Venezuela, RB` | `Venezuela, República Bolivariana de` |

Additional country renamings were applied **manually** in the intermediate
Excel files to align the macro data sources (IMF DOTS, IFS, GDD) with the
DEFAULT_DATABASE. These manual changes are not fully reflected in the
notebook code.

---

## References
- **Asonuma, Tamon and Christoph Trebesch (2016).** "Sovereign Debt Restructurings: Preemptive or Post-Default", Journal of the European Economic Association Vol 15(1), Pages 175-214

- **IMF Global Debt Database (GDD).**
  [imf.org/external/datamapper](https://www.imf.org/external/datamapper/datasets/GDD)

- **IMF Direction of Trade Statistics (DOTS).**
  [data.imf.org](https://data.imf.org/?sk=9d6028d4-f14a-464c-a2f2-59b2cd424b85)

- **IMF International Financial Statistics (IFS).**
  [data.imf.org](https://data.imf.org/?sk=4c514d48-b6ba-49ed-8ab9-52b0c1a0179b)

- **Federal Reserve Bank of St. Louis (FRED) — GS10.**
  [fred.stlouisfed.org/series/GS10](https://fred.stlouisfed.org/series/GS10)
