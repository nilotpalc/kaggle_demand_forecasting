# Demand Forecasting Plan

## Dataset
- File: `demand_forecasting.csv` — 76,000 rows, daily, 2022-01-01 to 2024-01-30
- 5 Stores × 20 Products = **100 unique time series**
- Target column: `Demand`
- Series identifiers: `Store ID` + `Product ID` → combined as `unique_id`
- Epidemic flag (0/1): two distinct disruption periods (~Apr–May 2022, ~Dec 2023)
- No missing values

## Forecast Horizon
- **X = 14 days**
- Train: 2022-01-01 → 2024-01-16 (~74,600 rows)
- Test: 2024-01-17 → 2024-01-30 (1,400 rows — 14 days × 100 series)

## Models
| Model | Library | Notes |
|---|---|---|
| AutoARIMA(season_length=7) | statsforecast | Tests weekly seasonality via AIC/BIC — does not force it |
| AutoETS(season_length=7) | statsforecast | Exponential smoothing, tests weekly structure |
| MSTL(season_length=[7,30]) | statsforecast | Decomposes weekly + monthly layers simultaneously |
| Prophet | prophet | Epidemic periods as holidays; other exogenous as add_regressor() |

## Preprocessing Pipeline (scikit-learn)
- **Numeric → StandardScaler:** Price, Discount, Inventory Level, Units Sold, Units Ordered, Competitor Pricing
- **Categorical → OneHotEncoder(drop=first):** Category, Region, Weather Condition, Seasonality
- **Binary passthrough:** Promotion, Epidemic
- Pipeline is fit on training data only; `.transform()` is reusable on any future data with same schema

## Epidemic Handling (Prophet)
Build `epidemic_df` from all rows where `Epidemic=1`. Pass as `Prophet(holidays=epidemic_df)`.
Covers both 2022 and 2023 epidemic episodes.

## Feature Importance
- Surrogate LightGBM model trained on full training set
- Features: all encoded exogenous + day_of_week, month, day_of_month + OrdinalEncoded Store ID & Product ID
- SHAP `TreeExplainer` → beeswarm summary plot + ranked bar chart
- Group one-hot encoded columns back into business features (Category, Region, Weather Condition, Seasonality) and compute grouped SHAP importance
- Translate SHAP outputs into business-facing demand-unit narratives: grouped driver impact table, impact share (%), Category and Region effect tables, and feature-movement vs demand-impact curves

## Evaluation Metrics
MAPE, RMSE, MAE — computed per model, summarised across all 100 series

## Notebook Structure
0. Install libraries
1. Imports
2. Load data + parse dates
3. EDA: shape, dtypes, nulls, duplicates, describe
4. EDA: visualisations (time series, Epidemic periods highlighted, distributions, correlation)
5. Feature groups + create unique_id
6. ColumnTransformer + Pipeline (fit on train, transform train + test)
7. Train/test split + build StatsForecast DataFrames
8. StatsForecast: fit AutoARIMA + AutoETS + MSTL, predict 14 days
9. Prophet: loop 100 series with holidays + regressors
10. Evaluate: MAPE, RMSE, MAE comparison table (all 4 models)
11. Forecast visualisation: 4–6 sample series with all model predictions overlaid
12. Feature importance: LightGBM surrogate + SHAP plots
13. Grouped feature importance: consolidate one-hot encoded variables so Category and Region appear as single features with importance
14. Business-friendly interpretation: convert SHAP outputs into demand-unit impact views for business users (driver table, % share, Category/Region effect, and movement-impact curves)

## Out of Scope
Hyperparameter tuning, cross-validation, deployment/API wrapping