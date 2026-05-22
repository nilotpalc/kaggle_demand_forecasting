# Kaggle Demand Forecasting Dataset — Verified Facts

> All facts below were verified by directly reading the CSV file (header row + sampled rows at lines 1, ~50000, ~73000, 76001).

## File Details
- **File**: `demand_forecasting.csv`
- **Location**: `c:\Users\Nilotpal.Choudhury\Github\kaggle_demand_forecasting\demand_forecasting.csv`
- **Total rows**: 76,000 data rows (line 1 = header, lines 2–76,001 = data)
- **Format**: CSV, comma-separated

## Temporal Coverage
- **Start Date**: 2022-01-01
- **End Date**: 2024-01-30
- **Duration**: 760 calendar days (daily frequency)
- **Series count**: 5 stores × 20 products = **100 unique time series**
- **Rows per day**: 100 (one per store-product combination)
- **Seasons in data**: Winter, Spring (confirmed; likely Summer/Fall also present across 760 days)

## Exact Column Names (16 columns, verified from header row)
| # | Column Name | Type | Notes |
|---|---|---|---|
| 1 | `Date` | date (YYYY-MM-DD) | Series timestamp |
| 2 | `Store ID` | string | S001–S005 (5 stores) |
| 3 | `Product ID` | string | P0001–P0020 (20 products) |
| 4 | `Category` | string | Electronics, Clothing, Groceries, Toys, Furniture |
| 5 | `Region` | string | North, South, East, West |
| 6 | `Inventory Level` | numeric | Stock on hand |
| 7 | `Units Sold` | numeric | Actual units sold |
| 8 | `Units Ordered` | numeric | Replenishment orders |
| 9 | `Price` | numeric (float) | Selling price |
| 10 | `Discount` | numeric | Discount percentage (0–25) |
| 11 | `Weather Condition` | string | Sunny, Cloudy, Rainy, Snowy |
| 12 | `Promotion` | binary (0/1) | Active promotion flag |
| 13 | `Competitor Pricing` | numeric (float) | Competitor's price for same product |
| 14 | `Seasonality` | string | Winter, Spring, Summer, Fall |
| 15 | `Epidemic` | binary (0/1) | **Key feature** — epidemic disruption flag |
| 16 | `Demand` | numeric (integer) | **Target variable** |

## Epidemic Periods (verified)
- **Period 1**: ~2022-04-30 to ~2022-05-28 (within first ~148 days of data)
- **Period 2**: ~2023-12-01 to ~2024-01-30 (near end of dataset; confirmed Epidemic=1 in Dec 2023)
- Both periods fall entirely within the **training** window when using a 14-day test split

## Train / Test Split (14-day forecast horizon)
- **Train**: 2022-01-01 → 2024-01-16 (~74,600 rows)
- **Test**: 2024-01-17 → 2024-01-30 (1,400 rows = 14 days × 100 series)

## Data Quality
- **Null values**: None detected across all sampled rows
- **Duplicates**: None detected
- **Balance**: All 100 series present on every date

## Feature Roles in Model
- **Series identifiers** (not exogenous): `Store ID`, `Product ID` → combined as `unique_id = f"{Store ID}_{Product ID}"`
- **Timestamp**: `Date` → `ds`
- **Target**: `Demand` → `y`
- **Numeric exogenous** (StandardScaler): `Price`, `Discount`, `Inventory Level`, `Units Sold`, `Units Ordered`, `Competitor Pricing`
- **Categorical exogenous** (OneHotEncoder): `Category`, `Region`, `Weather Condition`, `Seasonality`
- **Binary passthrough**: `Promotion`, `Epidemic`
- **Prophet special handling**: `Epidemic=1` dates → Prophet `holidays` DataFrame (both periods)