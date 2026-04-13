# Germany Electricity Demand Forecasting (2016–2025)

> **End-to-end time series forecasting pipeline for national electricity demand using SARIMA, Holt-Winters ETS, and walk-forward cross-validation - built in Python.**

---

## Brief Summary

Forecasted Germany's monthly electricity demand (GWh) using classical time series models, stationarity testing, seasonal decomposition, and 5-fold walk-forward cross-validation to identify the best-performing model with quantified uncertainty.

---

## Overview

Electricity demand forecasting is critical for grid operators, energy traders, and policymakers. Since electricity cannot be stored at scale, supply and demand must be balanced continuously - making accurate monthly forecasts essential for capacity planning, reserve margins, and cross-border energy procurement.

This project applies a full forecasting workflow to 119 months of Eurostat electricity data for Germany, covering pre-pandemic patterns, the COVID-19 demand shock (2020), the 2021–2022 energy crisis, and the structural demand decline that followed.

---

## Problem Statement

**Who needs this forecast?**  
National grid operators (e.g. Bundesnetzagentur) and energy trading desks at large utilities.

**What decisions depend on it?**
- How much generation capacity (coal, gas, renewables) to contract each month
- How much backup/reserve capacity to keep available
- Whether to import/export electricity from neighbouring countries

**Why is forecasting needed?**  
Demand varies from ~34,500 GWh (summer) to ~44,800 GWh (winter) and has declined structurally since 2022 due to efficiency improvements and industrial contraction. A constant-demand assumption would fail to capture these patterns.

---

## Dataset

| Property | Detail |
|---|---|
| **Source** | [Eurostat - NRG_CB_EM](https://ec.europa.eu/eurostat) |
| **Variable** | Electricity available to the internal market (Germany) |
| **Unit** | Gigawatt-hours (GWh) |
| **Frequency** | Monthly |
| **Time Span** | January 2016 – November 2025 |
| **Observations** | 119 |
| **Missing Values** | None |

---

## Tools and Technologies

| Category | Tools |
|---|---|
| **Language** | Python 3.x |
| **Data Manipulation** | pandas, numpy |
| **Time Series Modelling** | statsmodels (SARIMAX, ExponentialSmoothing, ARIMA) |
| **Statistical Tests** | statsmodels (ADF test, Ljung-Box test) |
| **Model Evaluation** | scikit-learn (MAE, RMSE), TimeSeriesSplit |
| **Visualisation** | matplotlib |

---

## Methods

### 1. Data Preprocessing
- Parsed monthly datetime index with `asfreq("MS")`
- Verified zero missing values
- No transformation applied (stable seasonal amplitude confirmed additive structure)

### 2. Exploratory Analysis
- **Decomposition:** Both additive and multiplicative decompositions applied (period=12); additive selected based on constant seasonal amplitude and residual std comparison
- **Stationarity:** ADF test on original series (p=0.78 → non-stationary); first differencing applied (p=0.003 → stationary); d=1, D=0 confirmed
- **Seasonal Unit Root:** ACF of seasonally differenced series examined - no slow decay at seasonal lags → no seasonal unit root
- **ACF/PACF:** Strong PACF spike at lag 1 (AR(1)); seasonal spikes at lag 12 → SARIMA(1,1,0)(1,0,1)[12] selected

### 3. Forecasting Models

| Model | Type | Specification |
|---|---|---|
| Seasonal Naïve | Benchmark | Repeats last 12 observed values |
| Holt-Winters ETS | Exponential Smoothing | Additive trend + additive seasonality |
| ARIMA | ARIMA-type | (1,1,0) - non-seasonal baseline |
| SARIMA | ARIMA-type | (1,1,0)(1,0,1)[12] - full seasonal model |

### 4. Model Evaluation
- **Single split:** Jan 2016 - Dec 2023 train / Jan 2024–Nov 2025 test
- **Cross-validation:** 5-fold walk-forward CV with 12-month test window per fold
- **Metrics:** MAE and RMSE

### 5. Forecast Uncertainty
- ETS: 95% prediction intervals via simulation (1000 repetitions)
- SARIMA: 95% analytical prediction intervals via `get_forecast()`

---

## Key Insights

- Germany's electricity demand shows a **clear downward structural trend** post-2022, driven by energy efficiency improvements and industrial contraction following the energy crisis
- The **COVID-19 shock (2020)** appears as an outlier in residuals - a genuine structural event, not a modelling artefact
- **Single train/test split** suggested Seasonal Naïve as best (MAE=688) - but this was misleading due to the choice of split point
- **5-fold cross-validation** revealed ETS as the true best performer (MAE=899 vs Naïve=1275), as it captures both trend and seasonality across multiple evaluation windows
- **SARIMA residuals** showed significant autocorrelation after excluding initialisation period (Ljung-Box p≈5e-07), suggesting a higher-order specification may be needed

---

## Results & Conclusion

### Cross-Validation Results (5-fold, test size = 12 months)

| Model | MAE | RMSE |
|---|---|---|
| **ETS (Holt-Winters)** | **899.01** | **1110.35** |
| SARIMA | 1288.78 | 1462.21 |
| Seasonal Naïve | 1274.55 | 1580.44 |
| ARIMA | 1757.21 | 1987.47 |

### Conclusion
**ETS (Holt-Winters Additive) is the recommended model.** It explicitly captures both the declining trend and stable seasonal pattern, generalises best across CV folds, and produces better-calibrated prediction intervals than SARIMA. ARIMA performs worst as it is fundamentally misspecified for seasonal data.

---

## Future Work

- **Add exogenous variables:** Temperature data and industrial production index via SARIMAX to capture demand drivers explicitly
- **Structural break detection:** Apply Chow test or regime-switching models to better handle the post-2022 demand decline
- **Higher-order SARIMA:** Investigate MA(1) or higher seasonal terms to address residual autocorrelation
- **ML comparison:** Benchmark against Prophet, XGBoost with lag features, or LSTM to evaluate deep learning approaches on this dataset

---

## Repository Structure

```
├── data/
│   └── nrg_cb_em_electricity_germany.csv    #Eurostat dataset
├── notebooks/
│   └── TS_assignment.ipynb                  #Full analysis notebook
├── report/
│   └── TS_assignment.pdf                    #Final report (PDF)
└── README.md
```

---

## How to Run This Project

### Prerequisites
```bash
pip install pandas numpy matplotlib statsmodels scikit-learn
```

### Steps
1. Clone the repository
```bash
git clone https://github.com/rknith16/Germany_Electricity_Demand_Time_Series_Forecasting_SARIMA_HoltWinters.git
cd germany-electricity-forecasting
```

2. Place the dataset in the `data/` folder

3. Open and run the notebook
```bash
jupyter notebook notebooks/TS_project.ipynb
```

> **Note:** Update the file path in the data loading cell to match your local directory.

---

## Author & Contact

**RISHABH KUMAR**  
E-mail: rknith16@gmail.com 
LinkedIn: www.linkedin.com/in/rishabhnith 
---

*Data source: Eurostat - Supply, transformation and consumption of electricity, monthly data (NRG_CB_EM). Dataset covers Germany, January 2016 - November 2025.*
