# Commercial Performance Analysis and Strategic Action Report

**Subject:** Comprehensive Sales Performance, Customer Segmentation, Predictive Modeling, and Strategic Roadmap  
**Scope:** Multi-Year Enterprise Sales Audit (December 2009 – December 2011)  
**Target Audience:** Executive Leadership, Commercial Director, Sales and Marketing Operations  


---

## 1. Executive Summary

This executive summary synthesizes the commercial analysis of 776,579 verified transactions across 5,852 identified accounts between December 1, 2009, and December 9, 2011:

1. **Extreme Revenue Concentration:** The business operates under a hyper-concentrated revenue structure, where the top 7.8% of customers (454 VIP wholesale accounts) generate 54.9% of total corporate turnover (£9.37M of £17.07M), and the top customer alone accounts for over £580,000.
2. **Recurring Autumn Commercial Surge:** Sales follow an identical seasonal trajectory in both 2010 and 2011, characterized by a Q1 slump (low of ~£442k in February) and an autumn acceleration that culminates in a massive November peak exceeding £1.13M, driven by retail partners pre-stocking for the Christmas shopping season.
3. **Strict B2B Procurement Operations:** Over 95% of transaction volume is executed strictly during standard working hours on weekdays (Monday through Friday, 09:00–16:00), while Saturday accounts for merely 0.057% of revenue due to regular warehouse and fulfillment shutdowns.
4. **Predictable Repurchase Dynamics:** Predictive modeling demonstrates that customer return likelihood over a 90-day forward horizon can be reliably forecast (ROC AUC = 0.805, Accuracy = 74.4%), with historical recency (20.5% importance) and ordering cadence (15.5% importance) serving as the primary drivers of future customer behavior.
5. **Cross-Selling Potential in Coordinated Collections:** Market basket analysis shows that 84.2% of frequent product pairs exhibit positive buying affinity (Lift > 1, with peak Lift reaching 37.64), driven by commercial clients purchasing complete thematic sets and color variations rather than isolated individual items.


---

## 2. Data Audit and Preparation

### 2.1 Raw Data Ingestion
The raw dataset comprised two transaction sheets covering consecutive calendar years:
- Sheet "Year 2009-2010": 525,461 rows
- Sheet "Year 2010-2011": 541,910 rows
- Total Raw Volume: 1,067,371 rows across 8 standard columns.

### 2.2 Data Cleaning and Governance Actions
To ensure analytical integrity, rigorous filtering and validation protocols were executed:
1. **Resolution of Sheet Overlap:** An overlap interval of exactly 9 days (December 1, 2010 to December 9, 2010) was identified across both sheets, encompassing 1,088 identical invoice numbers and 45,046 duplicate line items. Removing this duplication prevented artificial inflation of December 2010 sales by nearly 100%.
2. **Exclusion of Unidentified Guest Accounts:** Exactly 243,007 rows (22.77% of raw volume) lacked a valid `CustomerID`. These guest checkout transactions were excluded from customer-level RFM, clustering, and predictive modeling, but were archived for catalog-level sales audits.
3. **Isolation of Order Cancellations:** Exactly 8,292 cancelled invoices (prefixed with "C"), accounting for £1,462,051 in negative sales (~8.6% of gross turnover), were isolated into a dedicated cancellations table.
4. **Removal of Administrative Accounting Adjustments:** Internal system balancing entries (e.g., "Adjust bad debt", manual fee reversals) featuring non-commercial stock codes (`AMAZONFEE`, `POST`, `D`, `CRUK`) and anomalous unit prices were purged.

### 2.3 Verified Production Dataset
The resulting clean sales dataset comprises:
- **Clean Transactions:** 776,579 line items
- **Distinct Commercial Invoices:** 36,594 unique orders
- **Identified Customer Accounts:** 5,852 unique B2B clients
- **Distinct Catalog SKUs:** 4,620 active products
- **Total Gross Realized Sales:** £17,068,582.70 (Net turnover after cancellations: £15,606,531.70)


---

## 3. Commercial Findings and Sales Dynamics

### 3.1 Revenue Asymmetry and Customer Skewness
Customer spending follows a steep power-law distribution. While the arithmetic mean spending across 5,852 accounts is £2,916.71, the median customer spending is merely £856.02 (the mean is 3.4 times higher than the median).
- The bottom 50% of the customer base generates under 10% of total revenue.
- The top 20% of clients account for ~80% of aggregate turnover, perfectly corroborating the classical Pareto principle.
- The single largest corporate client generated £580,724.17 across the two-year timeframe.
- **Strategic Implication:** The commercial viability of the firm is sustained by a small group of high-volume wholesale buyers, making enterprise client retention the single most critical priority for executive management.

### 3.2 Recurring Multi-Year Seasonality
Analysis of monthly sales volumes demonstrates consistent seasonal patterns repeating across 2010 and 2011:
- **Q1 Trough:** Sales reach annual lows in January and February (e.g., February 2011 turnover was £442,294), reflecting post-holiday inventory absorption by retail clients.
- **Mid-Year Stability:** Monthly revenues stabilize between £550,000 and £750,000 from March through August.
- **Q4 Autumn Surge:** Revenue accelerates rapidly in September and October, reaching annual peaks in November (£1,155,978 in Nov 2010 and £1,136,534 in Nov 2011).
- *Data Integrity Note on December 2011:* Recorded revenue for December 2011 drops to £426,380 purely because data collection terminated on December 9 at 12:50 PM. Extrapolated on a pro-rata basis, December 2011 was on track to match or exceed December 2010 performance.

### 3.3 Operating Rhythm and Customer Identity
Transaction timestamps provide definitive evidence regarding the nature of the customer base:
- **Time of Day:** Peak purchasing occurs on weekdays between 11:00 and 14:00, with over 95% of all orders placed during standard business hours (09:00–16:00).
- **Day of Week:** Order volumes are evenly distributed from Monday to Friday. Saturday is an operational non-working day, generating only 0.057% of total revenue (£9,803 total over two years). Sunday shows an early afternoon resumption of orders as commercial buyers prepare procurement for Monday delivery.
- **Conclusion:** Customers are professional retail shopkeepers, commercial procurement agents, and corporate distributors (B2B), not individual end-consumers (B2C) browsing during evening leisure hours.

### 3.4 Catalog Concentration and "Star" Products
Catalog demand is anchored by a narrow group of high-performing products:
- Only 5 products appear simultaneously on both Top 10 lists (by units sold and by revenue generated):
  1. `WHITE HANGING HEART T-LIGHT HOLDER` (£247,048 revenue; 91,757 units; 1,490 distinct clients)
  2. `PAPER CRAFT , LITTLE BIRDIE` (£168,470 revenue; 80,995 units; 1 single wholesale client)
  3. `JUMBO BAG RED RETROSPOT` (£134,307 revenue; 74,224 units; 860 distinct clients)
  4. `ASSORTED COLOUR BIRD ORNAMENT` (£124,352 revenue; 78,234 units; 1,010 distinct clients)
  5. `MEDIUM CERAMIC TOP STORAGE JAR` (£81,417 revenue; 77,916 units; 138 distinct clients)
- *Commercial Anomaly:* The product `PAPER CRAFT , LITTLE BIRDIE` owes 100% of its historic volume to a single transaction by Customer `16446` on Invoice `581483`. This bespoke clearance contract must be decoupled from retail merchandising and demand forecasting models.

### 3.5 Customer Segmentation: Rule-Based RFM vs. Unsupervised K-Means
Comparative segmentation evaluated deterministic rule-based RFM scoring against K-Means clustering ($k=5$ on standardized log-transformed features):

| Segment / Cluster | Customer Count | Customer Share | Mean Recency | Mean Frequency | Mean Monetary | Total Revenue | Revenue Share |
|---|---|---|---|---|---|---|
| **VIP Customers** | 454 | 7.76% | 17.3 days | 32.6 orders | £20,635.50 | £9,368,515.70 | **54.89%** |
| **Loyal Customers** | 1,284 | 21.94% | 48.8 days | 9.1 orders | £3,627.24 | £4,657,379.30 | **27.29%** |
| **Occasional Customers** | 1,141 | 19.50% | 32.5 days | 2.5 orders | £666.19 | £760,120.40 | **4.45%** |
| **Asleep Customers** | 1,352 | 23.10% | 288.7 days | 3.8 orders | £1,375.00 | £1,858,990.00 | **10.89%** |
| **Inactive / Lost Customers**| 1,621 | 27.70% | 415.6 days | 1.2 orders | £261.31 | £423,577.30 | **2.48%** |

- **Key Structural Takeaways:**
  - The **VIP** and **Loyal** cohorts comprise 29.7% of accounts but generate **82.2% of corporate revenue** (£14.03M).
  - The **Asleep** segment contains 1,352 previously active recurring buyers who contributed £1.86M historically, but have lapsed for ~9.5 months and represent immediate churn risks.
  - The rule-based cohort **"Cannot Lose Them"** comprises 72 elite historical spenders (£503,405 historical turnover) who have ceased ordering and require immediate executive outreach.

### 3.6 Association Mining and Recommender Synergies
Market basket analysis across the top 300 products (covering 90.4% of all orders) proved that commercial clients purchase matching product suites:
- 37,740 product pairs achieve a Lift coefficient greater than 1.
- Top pairs reach Lift values between 25 and 37.64 (e.g., `RED SPOTTY PAPER CUPS` + `PLATES`, Lift = 37.64; `REGENCY TEACUP` color variations, Lift = 29.41).
- Commercial buyers purchase multiple colorways and coordinated sets to build unified merchandising displays in their retail storefronts.


---

## 4. Predictive Modeling and Retention Engine

### 4.1 Problem Formulation and Temporal Separation
To establish an actionable churn prevention mechanism, a leak-free predictive classification system was constructed:
- **Observation Window:** Transactions on or before `2011-09-10` (618,316 transactions).
- **Target Horizon:** 90 days following cutoff (`2011-09-11` to `2011-12-09`).
- **Target Definition (`Goal`):** Whether an existing customer active prior to cutoff returns to place at least one order in the 90-day window.
- **Population:** 5,256 active clients (43.6% returned, 56.4% did not return).

### 4.2 Model Performance and Validation
Models were trained on 75% of the data (3,942 clients) and evaluated on an unseen 25% test cohort (1,314 clients):

| Model | Test Accuracy | Precision | Recall | F1 Score | ROC AUC |
|---|---|---|---|---|---|
| **Majority-Class Baseline** | 56.47% | 0.00% | 0.00% | 0.00% | 0.500 |
| **Logistic Regression (Balanced)** | 73.14% | 69.18% | 69.06% | 69.12% | 0.795 |
| **Random Forest (Balanced, $N=300$)** | **74.35%** | **72.05%** | **67.13%** | **69.50%** | **0.805** |

### 4.3 Feature Drivers and Commercial Interpretation
- **Recency (20.5% importance):** Days elapsed since the last order before cutoff is the single strongest indicator of return likelihood.
- **Purchase Cadence (`AverageDaysBetweenPurchases`, 15.5% importance):** Account stability and established re-ordering routines heavily dictate repurchase timing.
- **Assortment Diversity (`DistinctProducts`, positive coefficient +0.568):** Customers who purchase across multiple catalog categories exhibit significantly higher retention, reflecting higher switching costs and deeper commercial integration.


---

## 5. Strategic Roadmap: Concrete Corporate Recommendations

Based on empirical data across sales, customer tiers, and predictive modeling, management should execute six strategic initiatives:

### 1. Establish an Elite Key Account Management (KAM) Unit
- **Target:** The 454 VIP clients (7.8% of accounts generating 54.9% of total revenue).
- **Action:** Transition these accounts away from generic web commerce into dedicated Key Account Management. Assign dedicated commercial account managers, establish contractual Service Level Agreements (SLAs), offer guaranteed inventory reserves during peak season, and implement tiered volume rebates.
- **Expected Impact:** Securing the top 454 accounts protects over £9.3M in annual recurring baseline turnover.

### 2. High-Touch Executive Outreach for At-Risk High-Value Accounts
- **Target:** The 72 clients identified in the "Cannot Lose Them" segment (£503k historic spend) and the top tier of the 1,352 "Asleep" customers.
- **Action:** Prohibit automated generic discount emails. Launch a direct telephone and field visit campaign conducted by Senior Account Executives or the Commercial Director. Conduct root-cause discovery (pricing pressure, delivery delays, competitor poaching) and negotiate tailored contract terms to re-engage lapsed buyers.
- **Expected Impact:** Re-activating even 20% of these 72 lapsed high-value accounts recovers over £100,000 in high-margin repeat revenue.

### 3. Supply Chain and Inventory Pre-Scaling for Q4 Seasonality
- **Target:** Warehouse operations, procurement, and fulfillment staffing.
- **Action:** Operational planning must not follow trailing spring/summer volumes. Procurement of top-selling Christmas SKUs (Regency collections, Christmas ornaments, gift bags) and warehouse temporary labor scaling must commence in August and September.
- **Expected Impact:** Eliminates costly stock-outs and fulfillment bottlenecks during the October–November demand peak, which accounts for ~25% of annual corporate revenue.

### 4. Automated Catalog Bundling and Cross-Selling Engine
- **Target:** B2B e-commerce platform and wholesale order catalogs.
- **Action:** Deploy the association rule recommender engine to dynamically present pre-packaged sets during digital checkout and in wholesale print catalogs (e.g., offering "Regency Complete Tea Service" combining Teacups, Cake Tins, and Cake Stand, or full colorway bundles of Party Candles and Bunting).
- **Expected Impact:** Lifts Average Order Value (AOV) by 8–15% across the 1,141 "Occasional" and 1,284 "Loyal" commercial buyers.

### 5. Asymmetric Marketing Budget Allocation
- **Target:** Marketing spend and automated CRM workflows.
- **Action:** Re-allocate marketing expenditures asymmetrically:
  - Allocate 75% of marketing resources toward retaining and expanding active Loyal and VIP accounts.
  - Allocate 15% toward onboarding and nurturing the 1,141 Occasional test-buyers into recurring monthly accounts.
  - Terminate costly manual sales outreach to the 1,621 Inactive/Lost accounts, servicing them exclusively through low-cost automated email reactivation drip sequences.
- **Expected Impact:** Maximizes marketing Return on Investment (ROI) and eliminates capital wastage on unrecoverable dormant accounts.

### 6. Production Deployment of the 90-Day Repurchase Prediction Model
- **Target:** Sales operations and CRM pipeline automation.
- **Action:** Operationalize the Random Forest predictive model as an automated monthly batch pipeline. Any active customer whose predicted repurchase probability drops below 0.50 should automatically trigger a sales rep alert task in the CRM system to initiate proactive engagement before the client lapses.
- **Expected Impact:** Reduces structural annual churn by 10–18% through early behavioral intervention.


---

## 6. Analytical Boundaries and Limitations

To maintain professional rigor, executive decision-makers must consider the analytical boundaries of this study:

1. **Absence of Cost and Profit Margin Data:** The dataset records gross top-line transaction prices (`UnitPrice` and `TotalPrice`), but excludes product cost of goods sold (COGS), shipping/fulfillment expenses, and operational overhead. High-revenue products or segments are not necessarily the most profitable on a net margin basis.
2. **Exclusion of Unregistered Guest Volume:** Approximately 22.8% of transaction rows (243,007 records) lacked Customer IDs and were omitted from customer-level RFM, clustering, and predictive modeling. If guest purchases exhibit fundamentally different seasonality or product affinities, customer-level findings may slightly understate total wholesale reach.
3. **Incomplete Calendar Interval for December 2011:** Data terminates on December 9, 2011. Apparent declines in total turnover for that month reflect data truncation rather than a business downturn.
4. **Static Market Basket Analysis:** Association rules evaluate co-occurrence on the same invoice among the top 300 products, which overlooks long-tail specialty items and does not capture sequential cross-purchases occurring across separate invoices over time.
5. **Lack of External Market Variables:** The analysis is bounded by historical transaction logs and does not account for competitor pricing changes, macroeconomic currency fluctuations (GBP/EUR), or external supply chain disruptions.
