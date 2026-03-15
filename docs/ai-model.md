# EarnGuard AI — AI Model Documentation

---

## Overview

The EarnGuard AI engine is the core differentiator of the platform. It replaces the traditional single-event parametric trigger (e.g., "pay if rainfall > 50mm") with a personalised, worker-level income prediction that makes the insurance far more accurate and fair.

The engine consists of three models, each serving a distinct purpose in the platform:

| Model | Purpose | Algorithm |
|---|---|---|
| Income Prediction Model | Predict expected weekly earnings per worker | Gradient Boosting Regressor |
| Risk Scoring Engine | Classify income volatility risk for premium pricing | Random Forest Classifier |
| Fraud Detection Module | Flag anomalous payout events before disbursement | Isolation Forest |

All models are built in Python using Scikit-learn, served via a Flask microservice, and loaded at startup using `joblib`. The AI engine is internal-only — it is never directly accessible from the frontend or the public internet.

---

## Model 1: Income Prediction Model

### What It Does

Before each policy week begins, this model predicts the specific amount a worker is expected to earn that week. This predicted figure becomes the baseline against which actual earnings are compared at the end of the week.

This is not a generic city-level estimate. It is a **personalised prediction** for each individual worker, accounting for their unique delivery history, their platform, their zone, and the external conditions forecast for the coming week.

### Why This Matters

Traditional parametric insurance asks: *"Did it rain more than X mm?"*
EarnGuard AI asks: *"Given this worker's history and this week's conditions, what should they have earned — and did they actually earn it?"*

This makes the product significantly more accurate and significantly harder to game.

---

### Input Features

**Worker History Features** (sourced from MongoDB `earnings` collection):

| Feature | Type | Description |
|---|---|---|
| `avg_weekly_income_4w` | Float | Average weekly earnings over the last 4 weeks |
| `avg_weekly_income_8w` | Float | Average weekly earnings over the last 8 weeks |
| `income_std_4w` | Float | Standard deviation of earnings over the last 4 weeks |
| `income_trend` | Float | Linear slope of earnings over the last 6 weeks (positive = growing) |
| `mon_avg`, `tue_avg` ... `sun_avg` | Float × 7 | Average daily earnings per day of week (delivery pattern) |

**Worker Profile Features** (sourced from MongoDB `workers` collection):

| Feature | Type | Description |
|---|---|---|
| `platform` | Categorical | Swiggy / Zomato / Amazon Flex / Blinkit / Other |
| `city_zone` | Categorical | Delivery zone (e.g., Koramangala, Andheri West) |
| `city` | Categorical | City (Mumbai, Delhi, Bengaluru, etc.) |

**Temporal Features** (computed at prediction time):

| Feature | Type | Description |
|---|---|---|
| `week_of_year` | Integer | 1–52, captures seasonal demand patterns |
| `is_festival_week` | Boolean | True if the week contains a major Indian festival (Diwali, Holi, Eid, etc.) |
| `is_month_end` | Boolean | True if the week spans the last 3 days of the month (higher order volumes) |

**External Forecast Features** (sourced from OpenWeatherMap + WAQI APIs):

| Feature | Type | Description |
|---|---|---|
| `forecast_rain_mm_max` | Float | Maximum single-day rainfall forecast for the week (mm) |
| `forecast_rain_days` | Integer | Number of days with forecast rainfall > 10mm |
| `forecast_temp_max` | Float | Maximum temperature forecast for the week (°C) |
| `forecast_temp_min` | Float | Minimum temperature forecast for the week (°C) |
| `forecast_aqi_avg` | Float | Average AQI forecast for the week |
| `city_disruption_index` | Float | Historical average disruption days per month for the worker's city (computed from `disruptions` collection) |

---

### Model Architecture

**Algorithm:** `GradientBoostingRegressor` (Scikit-learn)

Gradient Boosting is selected over simpler models (linear regression, decision tree) because:
- It captures **non-linear relationships** — e.g., the income impact of rain is not linear; light rain has minimal effect but heavy rain causes a sharp drop
- It handles **mixed feature types** (numerical + one-hot encoded categorical) without requiring separate preprocessing pipelines
- It is **robust to outliers** in earnings data (festival weeks, platform bonus weeks)
- It produces **feature importance scores** that help explain predictions to regulators and auditors

**Baseline hyperparameters:**
```python
GradientBoostingRegressor(
    n_estimators=200,
    learning_rate=0.05,
    max_depth=4,
    subsample=0.8,
    min_samples_leaf=10,
    random_state=42
)
```

**Preprocessing pipeline:**
```python
Pipeline([
    ('encoder', ColumnTransformer([
        ('onehot', OneHotEncoder(handle_unknown='ignore'),
         ['platform', 'city_zone', 'city'])
    ], remainder='passthrough')),
    ('model', GradientBoostingRegressor(...))
])
```

---

### Training Data

| Requirement | Detail |
|---|---|
| Minimum history per worker | 4 weeks of earnings data |
| Training frequency | Weekly batch retraining every Monday |
| Validation strategy | Walk-forward time-series cross-validation (no data leakage) |
| Cold start | Cohort median (same city + zone + platform) with 10% conservative discount |

---

### Model Output

```json
{
  "worker_id": "W12345",
  "week_start": "2025-07-14",
  "predicted_weekly_income": 6300,
  "confidence_interval": [5800, 6800],
  "risk_tier": "Medium",
  "model_version": "v1.3"
}
```

---

### Evaluation Metrics

| Metric | Target | Rationale |
|---|---|---|
| Mean Absolute Error (MAE) | < ₹400 | Directly interpretable in rupees |
| Mean Absolute Percentage Error (MAPE) | < 12% | Relative accuracy across income levels |
| Coverage accuracy | > 85% | Actual income falls within confidence interval ≥ 85% of the time |

---

## Model 2: Risk Scoring Engine

### What It Does

Classifies each worker into a risk tier — **Low**, **Medium**, or **High** — based on how volatile their income has historically been and how exposed their city and zone are to disruptions. This risk tier is used to:
- Recommend the most appropriate plan to the worker
- Inform future premium pricing adjustments
- Flag workers who may need a higher coverage cap

### Input Features

| Feature | Description |
|---|---|
| `income_std_8w` | Standard deviation of weekly earnings over the last 8 weeks |
| `income_drop_count_3m` | Number of weeks in the last 3 months where earnings dropped > 20% week-on-week |
| `city_disruption_frequency` | Average number of disruption days per month in the worker's city |
| `zone_weather_risk_score` | Composite weather risk score for the delivery zone (computed from historical rainfall + temperature data) |
| `platform_outage_exposure` | Total platform downtime hours experienced by the worker's platform in the last 3 months |

### Model

**Algorithm:** `RandomForestClassifier` (3-class: Low / Medium / High)

Random Forest is used here because:
- Classification task with a small, well-defined feature set
- Produces probability scores alongside class labels, enabling smooth tier boundaries
- Resistant to overfitting on small worker cohorts

### Output

```json
{
  "worker_id": "W12345",
  "risk_tier": "Medium",
  "risk_score": 0.61
}
```

**Risk tier to plan mapping:**

| Risk Tier | Recommended Plan | Rationale |
|---|---|---|
| Low | Basic (₹29) | Stable income, low disruption exposure |
| Medium | Standard (₹59) | Moderate variance, some disruption history |
| High | Pro (₹99) | High variance, monsoon-exposed zone, frequent outages |

---

## Model 3: Fraud Detection Module

### What It Does

Runs on every payout event before funds are disbursed. It scores the event for anomalous patterns that may indicate fraudulent or manipulated claims. The model is unsupervised — it learns what "normal" payout behaviour looks like and flags deviations.

### Why Unsupervised Detection

At launch, there is no labelled fraud dataset. Isolation Forest is ideal because:
- It does not require labelled fraud examples to train
- It is effective at detecting outliers in high-dimensional feature spaces
- The `contamination` parameter can be tuned conservatively to minimise false positives

### Fraud Signals and Features

| Signal | Feature Used | Description |
|---|---|---|
| Geographic clustering | `zone_claim_count` | Number of workers in the same micro-zone claiming simultaneously |
| Disruption mismatch | `disruption_confirmed` | Payout triggered but no disruption event logged for the worker's zone |
| Earnings manipulation | `gps_delivery_count_vs_reported_income` | Reported income inconsistent with GPS delivery activity (production only) |
| New account abuse | `account_age_days` | Worker registered fewer than 7 days before the payout trigger |
| Repeated max claims | `consecutive_cap_claims` | Worker has hit the coverage cap 3 or more consecutive weeks |
| Deviation outlier | `deviation_vs_zone_median` | Worker's income deviation is significantly higher than the median for their zone this week |

### Model

**Algorithm:** `IsolationForest` (Scikit-learn)

```python
IsolationForest(
    n_estimators=100,
    contamination=0.05,  # expect ~5% anomalous events
    random_state=42
)
```

### Output and Decision Logic

```json
{
  "payout_id": "P98765",
  "anomaly_score": 0.82,
  "flag": true,
  "reason": "geographic_clustering",
  "action": "hold_for_review"
}
```

| Anomaly Score | Action |
|---|---|
| < 0.75 | Payout proceeds automatically |
| 0.75 – 0.90 | Payout held — admin review required within 4 hours |
| > 0.90 | Payout blocked — admin must explicitly approve |

---

## Data Pipeline

```
Worker Earnings Data (manual entry / platform API)
              ↓
External API Data (Weather, AQI, Traffic)
              ↓
Feature Engineering (Python / Pandas)
  • Rolling averages and standard deviations
  • One-hot encoding of platform, city, zone
  • Festival calendar flag lookup
  • City disruption index computation
              ↓
Model Inference (Scikit-learn via joblib)
  • Income Prediction → predicted_weekly_income
  • Risk Scoring     → risk_tier
  • Fraud Detection  → anomaly_score (at payout time)
              ↓
Results stored in MongoDB
              ↓
Backend uses results for:
  • Dashboard display (predicted income)
  • Plan recommendation (risk tier)
  • Payout trigger evaluation (deviation check)
  • Fraud gate (anomaly score)
```

---

## Cold Start Strategy

New workers with fewer than 4 weeks of earnings history cannot be personalised immediately. The cold start strategy:

1. Identify the worker's cohort: same `city` + `zone` + `platform`
2. Use the cohort's **median** `predicted_weekly_income` as the worker's prediction
3. Apply a **10% conservative discount** to reduce payout exposure on unverified accounts
4. After 4 weeks of actual earnings data, transition fully to the personalised model
5. Cold-start workers are flagged in the `predictions` record: `"cold_start": true`

---

## Model Versioning and Auditability

- Models are versioned: `v1.0`, `v1.1`, `v1.2`, etc.
- Every prediction record stores `model_version` for full auditability
- Previous model files are archived — rollback is a single file swap
- Feature importance scores are logged weekly to monitor model drift
- If MAE exceeds ₹600 in any weekly validation run, an alert is raised for manual review

---

*EarnGuard AI — AI Model Document v2.0 | Phase-1 Hackathon*
