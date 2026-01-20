# Customer Lifetime Value Prediction for iGaming

> Predicting 12-month customer lifetime value from 30-day behavioural data using XGBoost gradient boosting.

![Python](https://img.shields.io/badge/Python-3.10+-blue.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)
![Status](https://img.shields.io/badge/Status-Portfolio_Project-orange.svg)

## Overview

This project demonstrates a production-ready CLV prediction pipeline for the iGaming industry. Using synthetic data from a hypothetical online casino ("Ca-Shane-O"), it showcases how machine learning can transform early customer behaviour into actionable business intelligence.

**Key Results:**
- **42% R²** — Model explains 42% of CLV variation (realistic for industry data with inherent noise)
- **6.4× Decile Lift** — Top 10% predicted customers are worth 6.4× more than bottom 10%
- **€255 MAE** — Average prediction error per customer
- **45 Features** — Including polynomial interactions and one-hot encoded categories

## Business Context

### The Problem
iGaming operators spend significant resources acquiring customers, but not all customers deliver equal value. Traditional approaches treat all customers the same, leading to:
- Wasted marketing spend on low-value acquisitions
- Under-investment in high-potential customers
- Reactive rather than proactive retention strategies

### The Solution
This model predicts a customer's 12-month value from their **first 30 days of activity**, enabling:
- **Targeted acquisition**: Allocate budgets to channels delivering high-CLV customers
- **Proactive retention**: Identify VIPs early for premium treatment
- **Resource optimisation**: Focus retention spend on customers worth saving

## Key Findings

### Top CLV Drivers (from SHAP Analysis)

| Feature | Impact | Business Implication |
|---------|--------|---------------------|
| Total deposits (30d) | +€80 per deposit | Deposit frequency > single large deposits |
| First deposit size | 2× multiplier | First impression predicts lifetime value |
| Days to first play | -€15/day after Day 3 | Activation speed matters — Day 0 players worth 36% more |
| Channel: Affiliate | +€159 vs average | Affiliate partnerships deliver quality |
| Channel: Paid Search | -€84 vs average | Review paid search ROI |
| Device: Desktop | +€100 | Desktop players show higher engagement |
| Market: Nordic | +€150 premium | Norway, Finland, Sweden justify acquisition costs |

### Customer Segmentation

```
Key Accounts (€5,000+):     2,347 customers (1.2%)
High Value (€1,500-€5,000): 26,439 customers (13.2%)
Medium (€500-€1,500):       87,011 customers (43.5%)
Low (<€500):                84,203 customers (42.1%)
```

## Technical Implementation

### Model Architecture

```
Primary Model: XGBoost Gradient Boosting
├── Features: 45 (18 behavioural + 24 categorical + 3 polynomial interactions)
├── Hyperparameters: max_depth=6, learning_rate=0.1, early_stopping_rounds=30
├── Training: 160,000 samples | Validation: 32,000 | Test: 40,000
└── Target: log1p(CLV) to handle right-skewed distribution

Complementary Model: BG/NBD (Beta-Geometric/Negative Binomial)
└── Purpose: Estimate P(customer still active) for churn risk scoring
```

### Feature Engineering

**Polynomial Interactions** capture non-linear effects:
- `first_deposit × total_deposits` — High initial investment + repeat behaviour
- Scaled and standardised before polynomial expansion

**One-Hot Encoding** for categorical variables:
- Country (10 EEA markets)
- Acquisition channel (Affiliate, Paid Search, Social, Email, Organic)
- Device (Mobile, Desktop, Tablet)
- First game type (Slots, Roulette, Poker, Baccarat, Other)

### Why These Results Are Realistic

An R² of 42% might seem "moderate" compared to textbook examples, but it reflects **realistic industry conditions**:

1. **Inherent unpredictability**: Human behaviour has randomness no model can capture
2. **30-day prediction window**: Longer observation → better predictions, but less business utility
3. **Synthetic noise**: €300 standard deviation noise embedded in data matches real-world variance
4. **Industry benchmarks**: Production casino models typically achieve 35-60% correlation with key features

The **6.4× decile lift** is the more important metric for business impact — the model successfully **ranks** customers by value, even if exact € predictions vary.

## Repository Structure

```
clv-prediction-igaming/
├── README.md                 # This file
├── requirements.txt          # Python dependencies
├── LICENSE                   # MIT License
├── .gitignore               # Git ignore rules
├── notebooks/
│   └── clv_model.ipynb      # Main analysis notebook (cleaned for GitHub)
├── data/
│   ├── sample_data.csv      # Synthetic sample (1,000 rows)
│   └── sample_predictions.csv
├── images/                   # Visualisations for Medium article
│   ├── 01_actual_vs_predicted.png
│   ├── 02_feature_importance.png
│   ├── 03_decile_analysis.png
│   ├── 04_correlation_heatmap.png
│   └── 05_shap_summary.png
├── models/
│   └── README.md            # Model serialisation notes
└── docs/
    ├── METHODOLOGY.md       # Detailed technical methodology
    └── BUSINESS_GUIDE.md    # Non-technical stakeholder guide
```

## Quick Start

### Prerequisites
- Python 3.10+
- pip or conda

### Installation

```bash
# Clone the repository
git clone https://github.com/yourshanej808/clv-prediction-igaming.git
cd clv-prediction-igaming

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Notebook

```bash
# Launch Jupyter
jupyter notebook notebooks/clv_model.ipynb
```

Or open directly in Google Colab: [![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/yourshanej808/clv-prediction-igaming/blob/main/notebooks/clv_model.ipynb)

## Methodology Highlights

### Data Generation
The synthetic dataset was created with embedded "ground truth" rules to validate model learning:
- First deposit × 2.0 contribution to CLV
- Affiliate channel bonus (+€200)
- Desktop device bonus (+€100)
- Late activation penalty (-€15/day after Day 3)

### Model Selection
XGBoost was chosen for:
- **Industry standard**: Widely used in production financial models
- **Handles mixed data**: Numbers and categories naturally
- **Feature interactions**: Captures non-linear relationships
- **Interpretability**: SHAP values explain individual predictions

### Evaluation Strategy
- **Train/Validation/Test split**: 64% / 16% / 20%
- **Early stopping**: Prevents overfitting by monitoring validation MAE
- **Decile analysis**: Business-focused metric showing ranking accuracy
- **SHAP values**: Regulatory-compliant explainability

## Technologies Used

| Category | Technologies |
|----------|-------------|
| **Core ML** | XGBoost, scikit-learn |
| **Data Processing** | pandas, NumPy |
| **Visualisation** | matplotlib, seaborn |
| **Interpretability** | SHAP |
| **Probabilistic Modelling** | lifetimes (BG/NBD) |
| **Environment** | Python 3.10, Jupyter |

## Responsible Gaming Note

This model includes responsible gaming considerations:
- **Deposit limits flag**: Customers with self-imposed limits are flagged
- **Limit breach tracking**: Identifies customers who reached their limits
- **GDPR compliance**: All data pseudonymised (8-digit player IDs)
- **Model transparency**: SHAP values provide audit trail for decisions

## Future Enhancements

- [ ] **Interactive dashboard**: Streamlit app for stakeholder exploration
- [ ] **Real-time scoring API**: FastAPI endpoint for production deployment
- [ ] **Cohort tracking**: Monitor prediction accuracy over time
- [ ] **A/B test framework**: Measure business impact of CLV-based decisions

## Author

**Shane Jackson** — iGaming professional building a data science portfolio through hands-on learning.

This project represents genuine skill development: understanding every technical concept deeply enough to discuss confidently with both technical and business stakeholders.

## Acknowledgements

- **Claude (Anthropic)** — AI assistance for code implementation and debugging
- **Perplexity** — Research on industry best practices
- **XGBoost team** — For the excellent gradient boosting library
- **SHAP contributors** — For making ML interpretability accessible

## License

This project is licensed under the MIT License — see [LICENSE](LICENSE) for details.

---

*Created as part of continuous professional development. Feedback welcome via GitHub issues.*
