# E-Commerce Sales Audit, Customer Segmentation, and Predictive Modeling

This repository contains an end-to-end commercial data science audit of a UK-based wholesale giftware business covering transactions between December 2009 and December 2011. The project encompasses raw data auditing, cleaning, exploratory sales analysis, RFM segmentation, unsupervised K-Means clustering, leak-free predictive classification of customer repurchase, and market basket recommendation algorithms.


---

## Project Overview

- **Dataset:** 776,579 verified commercial transactions across 36,594 unique orders and 5,852 B2B customer accounts.
- **Gross Turnover:** £17.07M gross realized revenue (£15.61M net revenue post-cancellations).
- **Primary Objective:** Analyze sales dynamics, profile customer behavior, forecast repurchase probabilities, and define actionable commercial recommendations for executive leadership.


---

## Key Findings

- **Revenue Asymmetry:** The top 7.8% of customers (454 VIP wholesale accounts) generate 54.9% of corporate turnover (£9.37M), confirming an extreme Pareto distribution where client retention is paramount.
- **Autumn Demand Surge:** Consistent multi-year seasonality features an annual trough in Q1 (February turnover ~£442k) and a rapid acceleration to a November peak exceeding £1.13M, driven by retail partner Christmas restocking.
- **Wholesale B2B Identity:** Over 95% of orders are placed between 09:00 and 16:00 on weekdays, with Saturday representing an operational warehouse shutdown (generating only 0.057% of revenue).
- **Repurchase Forecasting:** A leak-free 90-day predictive model achieved an ROC AUC of 0.805 and 74.4% accuracy (Random Forest), with past recency (20.5% importance) and purchase cadence (15.5% importance) as primary drivers.
- **Product Set Affinity:** Market basket analysis revealed that commercial buyers purchase matching design suites and multi-color variations, with top co-occurrence lift reaching 37.64.


---

## Reports and Deliverables

Comprehensive management reports and detailed day-by-day technical solutions are available in both English and Serbian:

- **Executive Sales Report (English):** [report/sales_report_eng.md](report/sales_report_eng.md)
- **Executive Sales Report (Serbian):** [report/sales_report_srb.md](report/sales_report_srb.md)
- **Complete Question Answers (English):** [answers/answers_eng.md](answers/answers_eng.md)
- **Complete Question Answers (Serbian):** [answers/answers_srb.md](answers/answers_srb.md)


---

## Project Structure

```text
online-shop-analytics/
├── answers/
│   ├── answers_eng.md           # Technical answers for Days 1 through 6 (English)
│   └── answers_srb.md           # Technical answers for Days 1 through 6 (Serbian)
├── data/
│   ├── processed/               # Clean parquet datasets (clean, model, rules)
│   └── raw/                     # Original raw Excel spreadsheets
├── graphs/
│   ├── d2/                      # EDA and temporal sales distribution charts
│   ├── d3/                      # RFM segmentation and Pareto curves
│   ├── d4/                      # K-Means elbow, silhouette, and cluster plots
│   ├── d5/                      # Model ROC curves, confusion matrices, feature importances
│   └── d6/                      # Association rules and top Lift bar charts
├── notebooks/
│   ├── 01_cleaning.ipynb        # Data ingestion, overlap removal, cancellations
│   ├── 02_eda.ipynb             # Exploratory analysis, time/day heatmaps, seasonality
│   ├── 03_rfm.ipynb             # Pareto distributions, product demand, RFM rules
│   ├── 04_kmeans.ipynb          # Log-scaling, StandardScaler, K-Means clustering (k=5)
│   ├── 05_predict.ipynb         # Time-based cutoff, leak-free classification (RF, LR)
│   └── 06_recommendation.ipynb   # Market basket analysis, support/confidence/lift, recommender
├── report/
│   ├── sales_report_eng.md      # Executive business and sales audit report (English)
│   └── sales_report_srb.md      # Executive business and sales audit report (Serbian)
├── task/
│   ├── STUDENTI_ZADATAK.md      # Detailed task specifications in Serbian
│   ├── STUDENT_VODIC.md         # Student guidance documentation
│   └── TASK_ENG.md              # Detailed task specifications in English
└── README.md                    # Project documentation and navigation guide
```


---

## Analytical Workflow

1. **01_cleaning.ipynb:** Resolved 9-day multi-sheet overlap (1,088 invoices), removed administrative bad-debt adjustments, isolated 8,292 cancellations (£1.46M), and exported 776,579 clean records to Parquet.
2. **02_eda.ipynb:** Evaluated seasonality, day-of-week purchasing patterns, and hourly heatmaps confirming B2B customer identity.
3. **03_rfm.ipynb:** Computed customer-level Recency, Frequency, and Monetary metrics; identified top star catalog items and segmented the base into 10 deterministic RFM cohorts.
4. **04_kmeans.ipynb:** Normalized skewed features with `np.log1p` and `StandardScaler`; optimized cluster count via Elbow and Silhouette analysis to establish 5 strategic customer tiers (VIP, Loyal, Occasional, Asleep, Inactive).
5. **05_predict.ipynb:** Established a strict chronological cutoff date (`2011-09-10`) to eliminate future leakage; engineered 13 pre-cutoff features and trained classification models predicting 90-day customer return (Random Forest ROC AUC: 0.805 vs. Dummy baseline: 0.500).
6. **06_recommendation.ipynb:** Mined item co-occurrences across top 300 products covering 90.4% of orders, generating 37,740 valid association rules (Lift > 1) and an automated bi-directional recommendation engine.

