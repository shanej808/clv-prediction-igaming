# Business Stakeholder Guide

A plain-English guide to understanding and using the CLV prediction model.

---

## What Does This Model Do?

This model looks at a customer's **first 30 days** of activity and predicts their **12-month total value** to the business.

Think of it like a credit score, but for customer value:
- **High score** = likely to generate significant revenue
- **Low score** = likely to churn or remain low-engagement

---

## Why Should I Care?

### Current State (Without CLV Model)
- Every new customer gets the same treatment
- VIP identification happens months after they've proven themselves
- Retention spend is reactive ("they stopped playing, quick do something!")
- Acquisition budgets allocated by channel cost, not quality

### Future State (With CLV Model)
- **Day 31**: Know who your future VIPs are before competitors do
- **Targeted investment**: Spend retention budget where ROI is highest
- **Smart acquisition**: Pay more for high-quality traffic sources (affiliates)
- **Early intervention**: Flag at-risk high-value customers before they churn

---

## Key Numbers You Need to Know

| Metric | What It Means | Our Result |
|--------|---------------|------------|
| **6.4× Decile Lift** | Top 10% predicted customers are worth 6.4× the bottom 10% | ✅ Strong ranking |
| **€255 MAE** | On average, predictions are within €255 of actual value | ✅ Acceptable for decisions |
| **42% R²** | Model captures 42% of what drives CLV (rest is unpredictable human behaviour) | ✅ Industry-realistic |

### What About That 42%?

You might think "only 42%?" — but human behaviour is inherently unpredictable. Even the best casino CLV models in production see 35-60% correlation. 

**What matters is that the model ranks customers correctly.** A 6.4× lift means your marketing team can confidently treat decile 10 differently from decile 1.

---

## What Drives Customer Value?

The model identified these as the strongest predictors:

### 🏆 Top Drivers

1. **Total deposits in first 30 days** — The single best predictor. More deposits = more committed.

2. **First deposit size** — Customers who deposit big initially tend to deposit big forever.

3. **Activation speed** — Players who play on Day 0 are worth 36% more than those who wait until Day 7+.

### 💡 Acquisition Insights

| Channel | CLV vs Average |
|---------|----------------|
| Affiliate | +€159 |
| Social | +€12 |
| Email | +€8 |
| Organic | -€15 |
| Paid Search | -€84 |

**Recommendation**: Review paid search ROI. Affiliates deliver significantly higher-quality customers.

### 🌍 Geographic Insights

| Market | CLV Premium |
|--------|-------------|
| Nordic (Norway, Finland, Sweden) | +€150 |
| UK | Baseline |
| Other EEA | Varies |

**Recommendation**: Nordic acquisition costs may be justified by higher CLV.

### 📱 Device Insights

| Device | CLV vs Average |
|--------|----------------|
| Desktop | +€100 |
| Mobile | Baseline |
| Tablet | -€30 |

Desktop players have longer sessions and make larger deposits.

---

## Customer Segments

Based on predicted CLV, customers fall into four segments:

| Segment | Predicted CLV | % of Base | Recommended Action |
|---------|---------------|-----------|-------------------|
| **Key Accounts** | €5,000+ | 1.2% | VIP treatment, dedicated manager |
| **High Value** | €1,500–€5,000 | 13.2% | Retention priority, cross-sell |
| **Medium** | €500–€1,500 | 43.5% | Engagement campaigns, increase frequency |
| **Low** | <€500 | 42.1% | Monitor, cost-efficient contact only |

---

## How To Use This In Practice

### For Marketing Teams

1. **Campaign targeting**: Use segments to personalise offers
   - Key Accounts: Exclusive bonuses, luxury incentives
   - High Value: Loyalty rewards, early access
   - Medium: Engagement boosters, deposit matches
   - Low: Volume promotions, reactivation attempts

2. **Acquisition bidding**: Adjust CPA targets by predicted segment
   - Pay more for traffic likely to become High/Key
   - Reduce spend on low-quality sources

### For CRM/Retention Teams

1. **Prioritisation**: When VIP team is capacity-constrained, predicted CLV helps triage

2. **Churn prevention**: Combine CLV with BG/NBD churn probability
   - High CLV + High churn risk = **urgent intervention**
   - Low CLV + High churn risk = automated win-back only

### For Finance/Analytics

1. **Cohort tracking**: Compare predicted vs actual CLV after 12 months
   - Feeds back into model retraining
   - Identifies drift in player behaviour

2. **Forecasting**: Predict future revenue from acquisition volumes and predicted CLV distribution

---

## Frequently Asked Questions

### "Can I trust a prediction for a single customer?"

**Partially.** Individual predictions have ~€255 error. Use CLV for:
- ✅ Ranking and prioritisation
- ✅ Segment assignment
- ⚠️ Not for precise revenue forecasting

### "Why not just use total deposits as CLV?"

Total deposits in 30 days correlates strongly with CLV, but:
- Doesn't capture channel effects (affiliate vs paid search)
- Ignores activation speed signals
- Misses device/geography premiums
- The model combines multiple signals for better ranking

### "How often should we retrain?"

**Monthly recommended.** Player behaviour and market conditions change. Model performance should be monitored weekly.

### "What if a new feature becomes important?"

The model can incorporate new features in retraining. Examples:
- New game type launch
- Regulatory changes (e.g., deposit limits)
- New acquisition channel

### "Is this GDPR compliant?"

Yes, with proper implementation:
- Predictions use pseudonymised IDs (not names/emails)
- SHAP values provide explainability for individual decisions
- No protected characteristics (age, gender) used as features
- Data subject access requests can include "your predicted segment"

---

## Glossary

| Term | Plain English |
|------|---------------|
| **CLV** | Customer Lifetime Value — total revenue from a customer over their relationship |
| **Decile** | One of 10 equal-sized groups when ranking customers |
| **Lift** | How much better than random — 6.4× means top group is 6.4× better than bottom |
| **MAE** | Mean Absolute Error — average prediction mistake in euros |
| **R²** | How much variation the model explains (0-100%) |
| **SHAP** | A method to explain why a specific prediction was made |
| **BG/NBD** | Probabilistic model estimating if a customer is still "alive" (active) |

---

## Contact

For questions about using CLV predictions in your team:
- **Technical queries**: Open a GitHub issue
- **Business strategy**: [Your contact details]

---

*This guide accompanies the CLV Prediction Model v1.0*
