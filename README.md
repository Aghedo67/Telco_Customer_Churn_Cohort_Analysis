# RetainIQ — Subscription Analytics Intelligence Platform

## Business Problem

StreamVault UK, a subscription streaming platform, experienced slowing MRR growth entering Q3. The Head of Growth commissioned this analysis to understand which subscriber cohorts are churning, what drives it, and where to prioritise retention investment — with a target of 5% reduction in monthly churn within two quarters.

---

## Project Structure

```
RetainIQ/
├── data/
│   ├── raw/                        # Downloaded datasets (not in Git — see data/raw/README.md)
│   └── processed/                  # Cleaned, analysis-ready data
├── notebooks/
│   ├── 01_data_cleaning_eda.ipynb  # Data cleaning, EDA, revenue at risk calculation
│   ├── 02_cohort_analysis.ipynb    # Cohort retention heatmap, curves, LTV by cohort
│   ├── 03_revenue_analytics.ipynb  # MRR movement, ARR, ARPU, LTV segmentation, Pareto
│   └── 04_churn_model.ipynb        # XGBoost churn model, SHAP explainability, risk scoring
├── app_4.py                        # Interactive Streamlit dashboard (Streamlit Cloud entry point)
├── app/
│   └── streamlit_app.py            # Local dashboard entry point
├── outputs/
│   └── figures/                    # Saved charts (01–23)
├── models/                         # Saved model artefacts
├── requirements.txt
└── README.md
```

---

## Key Findings

| Metric | Finding |
|--------|---------|
| Monthly churn rate | 26.5% — concentrated in months 0–3 of tenure |
| Highest-risk contract | Month-to-month: 42.7% churn vs 11.3% (one year) and 2.8% (two year) |
| Top churn risk combination | Fibre optic + electronic check payment + no online security |
| Average LTV (active subscribers) | ~£2,500 per subscriber |
| MRR at risk | ~£139K/month from churning subscribers |
| Retention driver | Subscribers with 3+ service add-ons churn at half the rate of those with none |
| XGBoost churn model AUC | ~0.845 — correctly ranks a churner above a non-churner 84.5% of the time |
| Retention campaign ROI | Targeting top 200 high-risk subscribers at £10/contact breaks even in under 2 months |

---

## Methodology

### 1 — Data Cleaning & EDA (`Notebook 01`)
- Cleaned two datasets: IBM Telco (7,043 subscriber records) and Online Retail II (~1M transactions)
- Fixed mixed-type columns, encoded the churn target, resolved null values in `TotalCharges`
- Answered five business questions: overall churn rate, tenure effect, pricing effect, service add-on effect, and revenue at risk
- Quantified MRR lost to churn and calculated the ROI threshold for retention campaign investment

### 2 — Cohort Retention Analysis (`Notebook 02`)
- Constructed acquisition cohorts from 1M+ e-commerce transactions using `InvoiceMonth` and `CohortMonth` period arithmetic
- Built a retention matrix: `cohort_pivot.divide(cohort_size, axis=0)` — each cell shows the fraction of a cohort still active at each tenure month
- Visualised as a heatmap (RdYlGn colourmap) to immediately surface which cohorts retained best
- Plotted average retention curve, month-1 retention by cohort (leading indicator), and LTV curves by acquisition period

### 3 — Revenue Analytics (`Notebook 03`)
- Reconstructed **MRR movement** (New, Returning, Reactivated, Churned MRR) using Python set operations — efficient O(n × months) loop over the transaction history
- Calculated **ARPU**, **ARR**, and **predictive LTV** using the geometric series formula: `LTV = ARPU / Monthly Churn Rate`
- Segmented LTV by contract type and plotted as a heatmap to surface the highest-value retention targets
- Ran a **Pareto analysis** — identified the top 20% of customers contributing ~80% of revenue

### 4 — Churn Prediction Model (`Notebook 04`)
- Engineered features from subscriber attributes: contract type (ordinal), service add-ons (binary), payment method (electronic check flag), derived `avg_monthly_spend` and `service_count`
- Built a **baseline Logistic Regression** model to establish the performance floor before introducing complexity
- Trained an **XGBoost classifier** with `scale_pos_weight` to handle the 74/26 class imbalance without data duplication
- Selected the classification **threshold** based on business cost trade-offs (precision vs recall) rather than defaulting to 0.5
- Applied **SHAP** (SHapley Additive exPlanations) for both global feature importance and individual subscriber explanations
- Scored all active subscribers and ranked them by churn probability — output feeds the live dashboard

---

## Live Dashboard

**[https://telcocustomerchurncohortanalysis-apxcnnf2db73pnranf3msw.streamlit.app/](https://telcocustomerchurncohortanalysis-apxcnnf2db73pnranf3msw.streamlit.app/)**

Five tabs:
- **Executive Summary** — churn rate KPIs, MRR at risk, contract and tenure breakdown
- **Cohort Retention** — interactive heatmap and average retention curve
- **LTV Analysis** — predictive LTV by segment, distribution of churned vs retained
- **Churn Risk** — live churn probability scores, MRR by risk tier, top-50 at-risk table
- **Revenue Calculator** — interactive retention campaign ROI calculator with sensitivity analysis

---

## Technical Stack

| Layer | Tools |
|-------|-------|
| Data processing | Python, pandas, numpy |
| Visualisation | matplotlib, seaborn, plotly |
| Machine learning | scikit-learn, XGBoost |
| Explainability | SHAP |
| Dashboard | Streamlit |

---

## How to Run

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Download datasets (see data/raw/README.md for direct links)

# 3. Run notebooks in order (Google Colab or Jupyter)
jupyter notebook notebooks/01_data_cleaning_eda.ipynb

# 4. Launch dashboard locally
streamlit run app_4.py
```

---

## Data Sources

- [IBM Telco Customer Churn](https://www.kaggle.com/datasets/blastchar/telco-customer-churn) — subscriber-level snapshot, 7,043 records, 21 features
- [Online Retail II (UCI Machine Learning Repository)](https://archive.ics.uci.edu/dataset/502/online+retail+ii) — UK e-commerce transactions 2009–2011, ~1M records across two sheets

---


