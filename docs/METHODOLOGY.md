# Technical Methodology

This document provides detailed technical explanations for the CLV prediction model, aimed at data scientists and technical reviewers.

## Table of Contents
1. [Problem Formulation](#problem-formulation)
2. [Data Generation](#data-generation)
3. [Feature Engineering](#feature-engineering)
4. [Model Selection](#model-selection)
5. [Training Strategy](#training-strategy)
6. [Evaluation Metrics](#evaluation-metrics)
7. [Interpretability](#interpretability)
8. [Limitations & Future Work](#limitations--future-work)

---

## Problem Formulation

### Business Objective
Predict 12-month Customer Lifetime Value (CLV) from 30-day behavioural data for an online casino operator.

### ML Formulation
- **Type**: Supervised regression
- **Target variable**: `clv_12m` (continuous, €0 to ~€10,000+)
- **Prediction horizon**: 12 months from acquisition
- **Observation window**: First 30 days of customer activity

### Why Regression Over Classification?
While classifying customers into value tiers (Low/Medium/High) might seem simpler, regression provides:
1. **Granular ranking** — Essential for prioritising limited retention resources
2. **Business flexibility** — Stakeholders can set their own thresholds
3. **Continuous feedback** — Actual CLV provides richer error signal for retraining

---

## Data Generation

### Synthetic Data Rationale
Real iGaming data is highly sensitive (GDPR, commercial confidentiality). Synthetic data allows:
- Public portfolio demonstration
- Ground truth validation (we *know* the underlying patterns)
- Reproducibility

### Generation Process

```python
# Core CLV formula (simplified)
base_clv = 50
base_clv += 2.0 * first_deposit           # Initial commitment signal
base_clv += 80 * total_deposits_30d       # Deposit frequency (strongest predictor)
base_clv += 0.6 * avg_session_mins_30d    # Engagement time
base_clv += -15 * (days > 3) * days       # Activation delay penalty

# Categorical bonuses
base_clv += channel_Affiliate * 200       # High-quality traffic source
base_clv += channel_Paid_Search * -50     # Lower quality traffic
base_clv += device_Desktop * 100          # Higher engagement platform
base_clv += country_Nordic * 150          # Premium market

# Add realistic noise
noise = np.random.normal(0, 300, size=N)  # €300 std dev
clv_12m = max(base_clv + noise, 0)
```

### Why €300 Noise?
This standard deviation was calibrated to produce:
- **0.39 signal strength** (correlation of key features with CLV)
- This matches industry benchmarks where production models see 0.35–0.60 correlation
- Too little noise → unrealistically high R² (≥0.90)
- Too much noise → model can't learn patterns

---

## Feature Engineering

### Feature Categories

| Category | Count | Examples |
|----------|-------|----------|
| Behavioural (numeric) | 18 | deposits, sessions, session duration |
| Demographic (categorical) | 4 | country, channel, device, game preference |
| Polynomial interactions | 3 | first_deposit × total_deposits |
| **Total after encoding** | **45** | |

### Polynomial Features
For top predictors, we create interaction terms:

```python
PolynomialFeatures(degree=2, include_bias=False)
```

This captures effects like:
- **High initial deposit × high frequency** = Committed high-value player
- **Long sessions × many sessions** = Deep engagement

### One-Hot Encoding
Categorical variables are one-hot encoded with unknown handling:

```python
OneHotEncoder(handle_unknown='ignore', sparse_output=False)
```

`handle_unknown='ignore'` is critical for production — new categories won't crash the model.

### Target Transformation
CLV distributions are typically right-skewed (many low, few high). We apply log transformation:

```python
y = np.log1p(clv_12m)  # log(1 + x) handles zero values
```

Predictions are back-transformed for business metrics:
```python
y_pred_euros = np.expm1(y_pred)  # exp(x) - 1
```

---

## Model Selection

### Why XGBoost?

| Consideration | XGBoost Advantage |
|---------------|-------------------|
| Mixed data types | Native handling of numeric + categorical |
| Non-linear relationships | Tree-based = automatic interaction detection |
| Regularisation | L1/L2 built-in, prevents overfitting |
| Industry adoption | Standard in fintech/gaming ML teams |
| Explainability | Compatible with SHAP |

### Hyperparameters

```python
XGBRegressor(
    n_estimators=500,          # Max trees (capped by early stopping)
    max_depth=6,               # Moderate complexity
    learning_rate=0.1,         # Standard starting point
    subsample=0.8,             # Row sampling (reduces overfitting)
    colsample_bytree=0.8,      # Column sampling per tree
    tree_method='hist',        # Fast histogram-based algorithm
    early_stopping_rounds=30,  # Stop if no improvement for 30 trees
    eval_metric='mae',         # Monitor MAE on validation set
)
```

### Hyperparameter Rationale

**max_depth=6**: Deeper trees risk overfitting to noise. With 200k samples and 45 features, depth 6 provides sufficient expressiveness while maintaining generalisation.

**learning_rate=0.1**: Higher rates (0.1–0.3) work well when combined with early stopping. The model stopped at 50 trees, suggesting 0.1 was appropriate.

**early_stopping_rounds=30**: If validation MAE doesn't improve for 30 consecutive trees, training stops. This is our primary overfitting defence.

---

## Training Strategy

### Data Splits

```
Total: 200,000 customers
├── Training: 128,000 (64%) — Model learning
├── Validation: 32,000 (16%) — Early stopping monitor
└── Test: 40,000 (20%) — Final evaluation
```

### Why Validation Split?
Early stopping requires a held-out set to monitor generalisation. Without it, the model would train until `n_estimators` (500) regardless of overfitting.

### Preprocessing Pipeline

```python
preprocessor = ColumnTransformer([
    ('top', Pipeline([
        ('scaler', StandardScaler()),
        ('poly', PolynomialFeatures(degree=2))
    ]), top_features),
    ('num', 'passthrough', other_numeric),
    ('cat', OneHotEncoder(handle_unknown='ignore'), categorical)
])
```

**Critical**: Preprocessor is **fit only on training data**, then applied to validation/test. This prevents data leakage.

---

## Evaluation Metrics

### Primary Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| **R²** | 0.422 | Model explains 42% of CLV variance |
| **MAE** | €254.93 | Average absolute error per customer |
| **RMSE** | 1.13 (log) | Root mean squared error in log scale |
| **Decile Lift** | 6.4× | Top 10% worth 6.4× bottom 10% |

### Why R² of 42% is Acceptable

R² measures variance explained. For CLV prediction:
- **Textbook ML examples**: Often R² > 0.90 on clean datasets
- **Real-world CLV**: 0.30–0.50 R² is typical with noisy human behaviour
- **Our synthetic noise**: Calibrated to industry-realistic signal strength

**The business cares about ranking, not point estimates.** A model with 42% R² but 6.4× decile lift is highly actionable.

### Decile Analysis Explained

1. Sort customers by predicted CLV
2. Divide into 10 equal groups (deciles)
3. Calculate actual average CLV per decile
4. Compare top decile to bottom decile

```
Decile 1 (lowest predicted): €258.61 actual average
Decile 10 (highest predicted): €1,652.49 actual average
Lift = 1652.49 / 258.61 = 6.4×
```

This proves the model **ranks** correctly, even if exact € values have error.

### Gini Coefficient

Gini measures ranking quality (0 = random, 1 = perfect):
- Our Gini: 0.076
- This is lower than typical because of high noise in synthetic data
- Decile lift is a more interpretable alternative

---

## Interpretability

### SHAP Values

SHAP (SHapley Additive exPlanations) provides:
- **Global importance**: Which features matter most overall?
- **Local explanations**: Why did *this* customer get *this* prediction?

```python
explainer = shap.TreeExplainer(xgb_model)
shap_values = explainer.shap_values(X_test[:100])
```

### Regulatory Compliance

iGaming is heavily regulated. SHAP enables:
- **Audit trails**: Explain why a customer was flagged as high/low value
- **Bias detection**: Ensure protected attributes don't drive predictions unfairly
- **Customer queries**: "Why did you target me for this promotion?"

### BG/NBD Complementary Model

XGBoost predicts **how much** a customer is worth.
BG/NBD predicts **if** the customer is still active.

Combined insight:
> "Customer #12345 is predicted to be worth €5,000 (high value), but has 70% churn probability (high risk) → urgent retention action."

---

## Limitations & Future Work

### Current Limitations

1. **Synthetic data**: Real iGaming data would have different distributions and correlations
2. **Single time point**: Model doesn't capture CLV evolution over time
3. **No external factors**: Economic conditions, competitor actions, regulatory changes not modelled
4. **Static features**: Doesn't incorporate real-time behavioural changes

### Future Enhancements

1. **Time series modelling**: LSTM or Transformer for sequential behaviour patterns
2. **Survival analysis**: Cox proportional hazards for churn timing
3. **Causal inference**: Estimate intervention effects (e.g., bonus impact)
4. **Online learning**: Update model incrementally as new data arrives
5. **A/B testing framework**: Measure actual business impact of CLV-based decisions

---

## Reproducibility

### Random Seeds
All randomness is controlled:
```python
np.random.seed(42)
random_state=42  # In all sklearn/xgboost calls
```

### Environment
See `requirements.txt` for exact package versions.

### Data Generation
The notebook includes the complete data generation code — anyone can recreate the exact synthetic dataset.

---

*For questions on methodology, please open a GitHub issue.*
