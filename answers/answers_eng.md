# Day 1 Question Answers — Data Loading and Cleaning

The answers below are derived directly from the code, execution outputs, and analytical results in notebooks/01_cleaning.ipynb and the accompanying assignment documentation.


## 1. Data Exploration (Task 1.2)

### Question 1.2.1: How many rows and columns does the dataset have?
The raw dataset (after merging the two Excel sheets "Year 2009-2010" and "Year 2010-2011") contains a total of 1,067,371 rows and 8 original columns (or 9 columns if including the helper column "List" added to track the sheet of origin).
- Sheet "Year 2009-2010": 525,461 rows
- Sheet "Year 2010-2011": 541,910 rows
Total: 525,461 + 541,910 = 1,067,371 rows.

### Question 1.2.2: Which columns have missing values and how many?
In the raw data, missing values are present in two columns:
- Customer ID: 243,007 missing values (22.77% of all rows in the dataset).
- Description: 4,382 missing values in the raw Excel file (converted to string representation 'nan' in the notebook).
All other columns (Invoice, StockCode, Quantity, InvoiceDate, Price, Country) contain no missing values (0 null entries).

### Question 1.2.3: Look at min for Quantity and Price in .describe(). What do you see and how do you explain negative values?
Descriptive statistics reveal substantial negative outliers:
- Quantity: minimum value is -80,995 (with a maximum of +80,995).
- Price: minimum value is -53,594.36 (with a maximum of 38,970.00).

Explanation:
- Negative values in the Quantity column represent order cancellations, product returns, and warehouse stock reconciliations. All such transactions are tagged with an invoice number beginning with the letter "C" (Cancelled).
- Negative values in the Price column do not represent unit prices of sold items, but rather accounting and manual adjustments (such as "Adjust bad debt") entered by system administrators to balance financial ledgers.

### Question 1.2.4: The columns are named Invoice, Price, Customer ID. In most tutorials online, they are named InvoiceNo, UnitPrice, CustomerID. Rename them to the longer form — why is it a good idea to have an agreed set of names?
Renaming columns to standard conventions is an industry best practice for several reasons:
1. Syntax convenience: Removing spaces ("Customer ID" -> "CustomerID") allows dot notation attribute access in pandas (e.g., df.CustomerID) and prevents syntax errors.
2. Semantic precision: The name "UnitPrice" clarifies that the field contains the single-unit price of an item rather than the total transaction amount.
3. Standardization and maintainability: Adopting a consistent, agreed schema aligns the codebase with common external libraries, online documentation, and team coding standards, significantly reducing onboarding friction and debugging overhead.


## 2. Sheet Overlap Verification (Task 1.3)

### Question 1.3.1: Do the periods overlap? In what exact interval?
Yes, the time periods of the two sheets overlap.
- Sheet "Year 2009-2010" spans from 2009-12-01 07:45:00 to 2010-12-09 20:01:00.
- Sheet "Year 2010-2011" spans from 2010-12-01 08:26:00 to 2011-12-09 12:50:00.
The overlap covers an interval of exactly 9 days, from December 1, 2010 through December 9, 2010.

### Question 1.3.2: How many invoices exist in both sheets?
Exactly 1,088 identical invoice numbers (InvoiceNo) appear in both sheets. These invoices encompass a total of 45,046 line-item rows distributed across the two sheets.

### Question 1.3.3: If those rows had remained duplicated, what would have happened to the reported sales for December 2010?
If these duplicate entries had not been removed, sales for the first 9 days of December 2010 would have been counted twice. This would have artificially inflated December 2010 revenue by nearly 100% during that timeframe, distorting holiday seasonality patterns and producing misleading growth and customer retention metrics.


## 3. Final Day 1 Questions (Notebook Questions)

### Question 3.1: What percentage of the data did you filter out? Is that too much?
The cleaning pipeline executed the following reduction steps:
- Initial raw rows: 1,067,371
- Duplicate rows removed: 34,335
- Cancellations separated ('C' invoices): 19,104
- Rows without CustomerID removed: 234,437
- Administrative codes removed: 2,856
- Rows with Quantity <= 0 removed: 0
- Rows with UnitPrice <= 0 removed: 60
- Clean dataset remaining: 776,579 rows

In total, 290,792 rows were removed, representing exactly 27.24% of the raw data (72.76% of the original data was retained).

This reduction is not excessive and is fully justified. The overwhelming majority of filtered rows were either anonymous transactions lacking a customer identifier (over 234,000 rows) or cross-sheet duplicates (over 34,000 rows). For behavioral cohort analysis, customer segmentation, and predictive lifetime modeling, including anonymous or corrupted records would introduce severe noise and undermine downstream decisions.

### Question 3.2: Rows without CustomerID make up almost a quarter of the dataset. Why do we discard them for customer-level analysis, but would not discard them when reporting overall business sales?
The distinction stems from differing analytical objectives:
1. Why they are excluded from customer-level analysis: Without a unique CustomerID, transactions cannot be attributed to an individual entity. It is impossible to calculate Recency, Frequency, or Monetary metrics per customer. If retained, every anonymous order would be treated as an isolated one-time customer, completely skewing churn rates, loyalty distributions, and segmentation models.
2. Why they are not excluded from total business sales: These items were genuinely purchased, shipped, and paid for. Dropping guest transactions from financial and operational sales reporting would understate corporate revenue by more than 20%, presenting an incorrect picture of total sales volume, inventory turnover, and commercial performance.

### Question 3.3: After removing cancellations, how many rows with Quantity <= 0 remained? What does that result tell you about where negative quantities come from?
After isolating cancellation invoices (those prefixed with "C"), the step filtering Quantity <= 0 removed exactly 0 rows (0 rows with Quantity <= 0 remained).

This result proves that all negative quantities in the original dataset stemmed exclusively from formal cancellations and merchandise returns. There were no hidden data-entry errors or untracked negative stock adjustments outside the standard cancellation workflow, confirming that the "C" invoice prefix serves as a complete and reliable indicator of all reversed transactions.

---

# Day 2 Question Answers — Exploratory Data Analysis (EDA)

The answers below are derived directly from the code, execution outputs, and visualizations in notebooks/02_eda.ipynb and the accompanying assignment documentation.


## 1. Top Values and Outliers (Task 2.2)

### Question 2.2: Are the largest TotalPrice line items data errors or genuine purchases? How did you determine that? Should they be removed?
Inspection of the top 10 items by TotalPrice (led by 168,469.60 £ for "PAPER CRAFT , LITTLE BIRDIE" and 77,183.60 £ for "MEDIUM CERAMIC TOP STORAGE JAR") shows:

1. Data errors vs. genuine purchases:
These records are genuine commercial B2B purchases (wholesale transactions), not technical errors or corrupted data.
2. How we determined this:
- The unit prices (UnitPrice) are completely ordinary, realistic catalog prices (e.g., 1.04 £, 1.66 £, 2.08 £, 2.10 £ per unit). The massive TotalPrice values result entirely from large bulk order quantities (Quantity ranging from 2,400 to 80,995 units).
- Product descriptions correspond to standard, high-volume catalog merchandise (storage jars, seasonal decor, fairy cake flannels, tea/coffee tins, paper lanterns).
- The customer IDs and invoice numbers represent valid, recurring commercial accounts. Several clients place repeated bulk orders (e.g., CustomerID 15749 ordered 3,114 units of the same item in both January and April 2011; CustomerID 15838 placed bulk seasonal candle-holder orders in October and November 2010; CustomerID 14646 in the Netherlands is an established major wholesale partner).
3. Should they be removed:
No, absolutely not. Deleting these transactions would be a severe analytical error. In wholesale retail, a small fraction of high-volume clients accounts for the majority of revenue. Removing them would effectively erase top customers and true corporate revenue. These extreme values should not be dropped; rather, statistical modeling should accommodate them using robust methods (such as log transformations or rank-based scaling).


## 2. Final Day 2 Questions (Notebook Questions)

### Question 2.1: Why is the mean of TotalPrice nearly double the median? Which of these two metrics would you report to an executive, and why?
1. Why the mean and median diverge:
In the dataset, the median TotalPrice is 12.45 £, whereas the arithmetic mean is 21.98 £ (the mean is 1.77 times the median). This divergence is driven by extreme positive skewness (right-skewed distribution). The vast majority of transaction rows consist of modest retail purchases (50% of items are priced below 12.45 £, and 75% are below 24.90 £). However, a small number of massive wholesale orders (ranging up to tens and hundreds of thousands of pounds) pull the arithmetic mean upward.
2. Which metric to report to an executive and why:
Both metrics should be included, but with clearly distinguished business roles:
- The median (12.45 £) represents the "typical" purchase line item for an ordinary customer, as it is robust against extreme outliers and accurately reflects everyday order behavior.
- The mean (21.98 £) is an aggregate financial indicator reflecting the average revenue generated per transaction line (Total Revenue = Total Rows * Mean).
If an executive were presented solely with the mean of ~22 £, they would mistakenly assume that an ordinary transaction brings in that amount, when in reality over 70% of transactions fall below it. Consequently, the median must always be highlighted as the representative measure of customer transaction size, accompanied by the mean to capture overall financial yield.

### Question 2.2: Ireland and the Netherlands rank high in revenue but have very few customers. Calculate the average basket per country. What can you deduce about the nature of these customers?
Sales metrics by country indicate:
- United Kingdom: 14,288,773.50 £ revenue (83.71%), 5,334 customers, 33,361 invoices, average basket (Revenue / Invoices) = 428.31 £.
- EIRE (Ireland): 586,626.08 £ revenue (3.44%), 3 customers, 528 invoices, average basket = 1,111.03 £ (over 195,000 £ in revenue per customer).
- Netherlands: 549,773.41 £ revenue (3.22%), 22 customers, 216 invoices, average basket = 2,545.25 £ (over 24,000 £ in revenue per customer).
- Australia: 167,800.01 £ revenue (0.98%), 15 customers, 89 invoices, average basket = 1,885.39 £.

Deductions about customer profiles:
These accounts are large B2B wholesale partners and regional distributors, not individual retail consumers (B2C). In Ireland and the Netherlands, the business does not operate as a conventional e-commerce site serving thousands of small retail shoppers. Instead, it functions as a primary international supplier shipping bulk container-level merchandise to commercial trade partners and retail chains for local re-distribution.

### Question 2.3: Is seasonality present? Does the pattern repeat across both years? Why is repetition across years essential for drawing conclusions?
1. Presence and consistency of seasonality:
Yes, pronounced and consistent seasonality is evident, repeating across both 2010 and 2011. In Q1 (particularly January and February), sales reach annual lows (e.g., February 2011 generated 442,294 £). Activity begins accelerating in late summer/autumn (September and October) and surges to a major annual peak in November (1,155,978 £ in November 2010 and 1,136,534 £ in November 2011).
2. Why multi-year repetition is essential:
If this surge had occurred only in a single year, it could easily be dismissed as a one-time anomaly, a temporary marketing campaign, or a macro-economic fluke. Observing the exact same seasonal trajectory across two consecutive years validates that this is a stable, recurring business cycle driven by retailers stocking up ahead of the Christmas shopping season. For management, this confirms that procurement, supply chain logistics, and warehouse staffing must be scaled up months in advance (August through October) to handle the November volume.

### Question 2.4: What happens on Saturdays? Is this a data error?
1. What happens on Saturdays:
Saturday accounts for only 9,803.05 £ across the entire two-year dataset, representing just 0.057% (~0.06%) of total corporate revenue.
2. Is it a data error:
No, this is not a data error. Saturday is an operational non-working day for the company's fulfillment center, office, and warehouse facilities. The tiny volume recorded on Saturdays reflects occasional automated online orders logged by web servers or maintenance entries. The company operates on a Sunday-through-Friday commercial cycle (closing on Saturday and resuming dispatch on Sunday afternoon for Monday delivery), which is a common operating model in British distribution and wholesale logistics.

### Question 2.5: Examine the heatmap. At what times of day do purchases occur? What does this suggest regarding whether customers are B2C consumers or B2B businesses?
1. Purchasing hours:
The day-by-hour revenue heatmap indicates that transaction activity is strictly concentrated during standard workdays (Monday through Friday) between 09:00 and 16:00, with peak activity between 11:00 and 14:00. Prior to 08:00 and after 17:00, sales volume drops sharply, while evening and nighttime hours are virtually inactive.
2. B2C consumers vs. B2B businesses:
This purchasing timeline strongly indicates that customers are commercial businesses (B2B buyers, retail shop owners, and corporate procurement staff). Individual retail consumers (B2C) typically browse and purchase during leisure hours—specifically in the evenings (18:00–23:00) and over weekends. Because over 95% of transaction volume occurs strictly during regular business working hours on weekdays, combined with large unit order quantities, the customer base is unequivocally dominated by commercial B2B clients.

---

# Day 3 Question Answers — Products and RFM Analysis

The answers below are derived directly from the code, execution outputs, and segmentation results in notebooks/03_rfm.ipynb and the accompanying assignment documentation.


## 1. Best-Selling Products (Task 3.1)

### Question 3.1: How many products appear on both lists (by quantity and by revenue)? What does this mean?
Exactly 5 products appear on both top 10 lists (by total quantity sold and by total revenue generated):
1. WHITE HANGING HEART T-LIGHT HOLDER (StockCode 85123A) — 247,048.01 £ revenue (#2), 91,757 units (#2), 1,490 customers
2. PAPER CRAFT , LITTLE BIRDIE (StockCode 23843) — 168,469.60 £ revenue (#3), 80,995 units (#3), 1 customer
3. JUMBO BAG RED RETROSPOT (StockCode 85099B) — 134,307.44 £ revenue (#4), 74,224 units (#6), 860 customers
4. ASSORTED COLOUR BIRD ORNAMENT (StockCode 84879) — 124,351.86 £ revenue (#5), 78,234 units (#4), 1,010 customers
5. MEDIUM CERAMIC TOP STORAGE JAR (StockCode 23166) — 81,416.73 £ revenue (#7), 77,916 units (#5), 138 customers

What this means:
These 5 products represent the core commercial pillars ("star products") of the company. They achieve a rare synergy: massive, recurring customer demand combined with healthy unit economics capable of generating immense cash flow. In contrast, products appearing solely on the quantity list (such as WORLD WAR 2 GLIDERS with over 105,000 units sold but only ~24,000 £ in revenue) are low-margin, high-volume impulse add-ons, while products appearing solely on the revenue list (such as REGENCY CAKESTAND 3 TIER with ~277,656 £) are higher-ticket premium items sold in moderate volumes.


## 2. Pareto Analysis (Task 3.2)

### Question 3.2: What percentage of the catalog generates 80% of revenue? Does the 80/20 rule hold?
1. Catalog percentage generating 80% of revenue:
Exactly 1,146 products, representing 21.7% (~22%) of the total product assortment (grouped by StockCode + Description), generate 80% of total revenue.
2. Does the 80/20 rule hold:
Yes, the Pareto principle (80/20 rule) holds remarkably well: approximately 22% of catalog items account for 80% of company revenue. This demonstrates extreme revenue concentration within the top quintile of catalog SKUs, while the remaining ~78% forms a long tail that contributes only 20% of revenue. For supply chain operations, inventory management and supplier negotiations must prioritize this essential 22%.


## 3. Final Day 3 Questions (Notebook Questions)

### Question 3.1: Why do the "top by quantity" and "top by revenue" lists differ so much?
The two lists diverge due to substantial variation in unit prices across catalog categories:
- Top products by quantity consist of low-priced novelty and craft items (such as miniature glider planes or paper baking cups) selling for pennies to around one pound. Although they move in massive physical volume, their aggregate monetary contribution is relatively small.
- Top products by revenue are dominated by higher-value items (such as the tiered cake stand REGENCY CAKESTAND 3 TIER priced at £10–£12). Selling fewer units of a higher-priced item yields significantly higher revenue than selling tens of thousands of inexpensive accessories.

### Question 3.2: Grouping by StockCode yields ~4,620 products, while grouping by StockCode + Description yields ~5,274. Where does the difference come from? What does this mean for the reliability of product descriptions and which key would you use?
1. Source of the discrepancy:
The difference arises because 598 distinct product codes (`StockCode`) have multiple text descriptions (`Description`) recorded across the two-year period. This happens due to:
- Typographical errors and spelling variations (e.g., SPUNGE vs. SPONGE, inconsistent punctuation, commas, or slashes).
- Rebranding and seasonal product title updates (e.g., WHITE SPOTS vs. POLKADOT).
- Inconsistent casing or ad-hoc abbreviation by warehouse operators during inventory entry.
2. Reliability of product descriptions:
The text `Description` is unreliable as an analytical identifier because it is prone to data entry errors and drifts over time.
3. Which key to use:
The canonical alphanumeric `StockCode` must always be used as the primary grouping key. When human-readable product names are required for dashboards or reports, the most recent or most frequent `Description` associated with that `StockCode` should be mapped as an attribute.

### Question 3.3: One product in the top list has only 1 customer. Find it. Is that normal and what should be done with it in the analysis?
1. Identification:
The product is `PAPER CRAFT , LITTLE BIRDIE` (StockCode `23843`). It sold 80,995 units and generated 168,469.60 £ in revenue from a single customer (CustomerID `16446`) on a single invoice (`581483`) on December 9, 2011.
2. Is it normal:
No, this is an extreme commercial anomaly—most likely a custom institutional B2B contract, bespoke manufacturing order, or total warehouse clearance sale. It is highly atypical for the third-highest-grossing product in company history to be backed by a single purchase order.
3. How to handle it in analysis:
- In corporate financial reports of total realized revenue, the transaction must be retained as the funds were genuinely received.
- In merchandising, catalog demand forecasting, and recommendation algorithms (Market Basket analysis), this product should be isolated or treated separately. Otherwise, algorithms would treat it as a top-selling retail hit with broad consumer demand, leading to disastrous over-stocking and distorted cross-selling rules.

### Question 3.4: Median spending is ~856, and mean is ~2,917. Explain the difference.
In the RFM dataset covering 5,852 customers, median Monetary spend is 856.02 £, while the arithmetic mean is 2,916.71 £ (the mean is 3.4 times higher than the median).

This gap is caused by severe positive skewness (power-law distribution) in customer spending:
- Half of all customers (50%) spend less than 856 £ across the entire two-year timeframe.
- Conversely, a tiny elite of high-volume wholesale buyers and distributors spend hundreds of thousands of pounds (the top customer spent over 580,000 £). These extreme high-value outliers pull the arithmetic mean sharply upward.
The median represents what an ordinary customer spends, while the mean represents total corporate turnover divided by customer count and does not describe any typical customer.

### Question 3.5: Which segment has the most customers? Which generates the most money? Is it the same segment? What does that difference mean for the marketing budget?
1. Segment distribution:
- Segment with the most customers: `Lost` with 1,510 customers (25.8% of the entire customer base). The second largest is `Loyal Customers` with 1,151 customers (19.7%).
- Segment generating the most revenue: `Champions` with 8,781,126.40 £ (51.4% of total company revenue). The second highest is `Loyal Customers` with 4,658,851.80 £ (27.3%).
2. Are they the same segment:
No, they are polar opposites. The `Lost` cohort contains over a quarter of all customers (25.8%) but delivers merely 3.6% of revenue. In contrast, `Champions` represent only 14.2% of customer accounts (830 clients) yet generate over half of all corporate revenue.
3. Marketing budget implications:
The marketing budget must be allocated asymmetrically based on Return on Investment (ROI):
- The lion's share of retention budget (dedicated account managers, priority fulfillment, VIP terms, exclusive volume rebates) must be dedicated to safeguarding and nurturing `Champions` and `Loyal Customers`. Together, these cohorts represent ~34% of clients but produce nearly 79% of total revenue. Losing even a handful of Champions inflicts immediate, severe revenue losses.
- Allocating expensive marketing spend to broad win-back campaigns across all 1,510 lost customers yields poor ROI. The `Lost` cohort should be engaged via low-cost automated email sequences, while high-touch outreach is concentrated strictly on high-value accounts at risk (`Cannot Lose Them` and `At Risk`).

### Question 3.6: For the segment "Cannot Lose Them" — describe the typical customer and propose one concrete action.
1. Profile of the typical customer:
The `Cannot Lose Them` segment comprises 72 high-value clients (1.2% of customer accounts) who historically generated over half a million pounds (503,405.10 £, or 2.9% of revenue), averaging nearly 7,000 £ per customer.
Profile: These customers exhibit the highest historical Frequency and Monetary scores (F=5, high M), meaning they were once among the firm's most frequent and profitable commercial buyers. However, their Recency score has plummeted to 1 or 2—they have not placed an order in many months. They have either encountered service friction, altered their procurement, or defected to a competitor.
2. Proposed concrete action:
Do not send generic automated coupons or marketing newsletters. Initiate direct, personal executive outreach led by a Senior Account Executive or Sales Director (via telephone or face-to-face visit). Conduct a relationship review: identify the root cause of lapsed orders (logistics delays, damaged goods, pricing pressure, competitor poaching), offer tailored remedies (e.g., custom volume pricing, dedicated customer service, or extended payment terms), and secure a renewed supply contract.

---

# Day 4 Question Answers — Customer Segmentation (K-Means)

The answers below are derived directly from the code, execution outputs, and clustering models in [04_kmeans.ipynb](notebooks/04_kmeans.ipynb) and the accompanying assignment documentation.


## 1. Data Preparation (Task 4.1)

### Question 4.1.1: Calculate skewness before and after log transformation. Why are both steps (log1p and StandardScaler) mandatory? What would specifically happen to the clusters if you skipped standardization?

#### 1. Skewness Values Before and After `np.log1p()`
The skewness measured across 5,852 customers before and after log transformation:

| Feature | Raw Skewness | Log-Transformed Skewness (`np.log1p`) | Skewness Reduction |
|---|---|---|---|
| **Recency** | 0.89 | -0.49 | Moderate positive skew converted to a near-symmetric distribution |
| **Frequency** | 12.04 | 1.00 | Heavy positive skew reduced by 91.7% |
| **Monetary** | 25.33 | 0.27 | Extreme power-law skew reduced by 98.9% (near-perfect normal symmetry) |

#### 2. Why Both Steps (log1p and StandardScaler) Are Mandatory
K-Means clustering relies fundamentally on **Euclidean distance** in $n$-dimensional space:
$$d(x, y) = \sqrt{\sum_{i=1}^n (x_i - y_i)^2}$$

Because Euclidean distance operates geometrically in spherical metric space, the data must satisfy two strict structural prerequisites:
1. **Log Transformation (`np.log1p`):** Resolves severe distributional skewness and compresses extreme multi-order-of-magnitude outliers (e.g., spending from 3.75 £ up to 580,000 £). Without log transformation, hyper-dense clusters form around the 95% majority of low-value customers, while a handful of wholesale "whales" are isolated into trivial single-customer outlier clusters or distort the centroid positions entirely. Log transformation linearizes multiplicative differences into additive scales.
2. **Standardization (`StandardScaler`):** Centers each feature to a mean of $\mu = 0$ and unit variance $\sigma = 1$. Even after log transformation, the numerical ranges and variances of Recency, Frequency, and Monetary differ. StandardScaler guarantees that all three dimensions contribute with equal geometric weight to the distance calculations.

#### 3. Specific Consequences If Standardization Is Skipped
If `StandardScaler` is omitted, the Euclidean distance calculation becomes completely dominated by whichever feature has the widest numerical range and largest raw variance. 
- In unscaled raw data, Monetary values span hundreds of thousands of pounds, while Frequency ranges from 1 to 300 and Recency from 0 to 738. In that case, distance is 99.9% driven by Monetary spend, effectively collapsing the algorithm into a 1D Monetary cut and rendering Frequency and Recency completely irrelevant.
- Even if applied after `np.log1p` without `StandardScaler`, log-Recency (spanning ~0 to 6.6) and log-Monetary (spanning ~1.5 to 13.3) would exert unequal geometric leverage compared to log-Frequency (spanning ~0.69 to 5.7). Features with slightly higher variances pull centroids toward their axes, creating elongated, distorted ellipsoidal clusters instead of compact, spherical, meaningful behavioral segments.


## 2. Determining Number of Clusters (Task 4.2 / Question 4.1)

### Question 4.1: The silhouette score suggests k=2, while the elbow method suggests k=5. Which would you choose and why?

#### 1. Empirical Results Across $k \in [2, 10]$
Evaluation metrics computed across cluster numbers:

| $k$ | Inertia (Elbow) | Silhouette Score | Business Usability |
|---|---|---|---|
| **k = 2** | 8,562.3 | **0.444** (Global Maximum) | Trivial binary split (Active vs. Inactive); negligible commercial value |
| **k = 3** | 6,338.4 | 0.351 | Broad 3-tier grouping; merges distinct high-value segments |
| **k = 4** | 4,908.2 | 0.365 | Decent mathematical structure, but misses nascent/occasional distinction |
| **k = 5** | **4,090.2** | **0.340** (Optimal Elbow Bend) | **Selected Model: Highly actionable multi-tier strategic portfolio** |
| **k = 6** | 3,549.3 | 0.336 | Over-fragments loyal and churned sub-groups |
| **k = 7** | 3,181.9 | 0.301 | Diminishing returns; silhouette declines sharply |
| **k = 8** | 2,892.4 | 0.294 | Overfitting; sub-clusters lack distinct marketing actions |
| **k = 9** | 2,650.1 | 0.285 | Unstable centroids; poor interpretability |
| **k = 10** | 2,460.3 | 0.284 | Excessive granularity for management execution |

#### 2. Selected Choice and Justification: Choose $k = 5$
We choose **$k = 5$** for production and commercial decision-making based on the following reasons:

1. **Mathematical Meaning of $k = 2$:** 
   The silhouette score reaches its peak at $k=2$ (0.444) because the customer population has a strong polarity between active/recent buyers and completely dormant/churned buyers. While a 2-cluster partition is mathematically separated with wide margins, it merely divides customers into "Current vs. Former" or "Big vs. Small". It fails to differentiate between a VIP whale spending 50,000 £ and a steady repeat buyer spending 2,000 £, or between a churned customer who left 400 days ago versus an at-risk customer who last purchased 9 months ago.
2. **Elbow Method Inflection at $k = 5$:** 
   The inertia curve exhibits its clearest inflection ("elbow bend") at $k=5$ (inertia drops from 8,562 at $k=2$ down to 4,090 at $k=5$, after which the rate of inertia reduction flattens significantly into diminishing marginal returns).
3. **Actionability and Commercial Return:** 
   Clustering is an applied unsupervised business tool, not a purely theoretical exercise. $k=5$ strikes the optimal trade-off between statistical validity (solid silhouette of 0.340 with cohesive clusters) and operational utility. It reveals 5 distinct behavioral cohorts that map directly to real-world corporate actions: VIP retention, loyal cross-selling, nurturing recent low-ticket buyers, aggressive win-back of slipping customers, and low-cost suppression of dead accounts.


## 3. Cluster Profiles & Business Names (Task 4.3 / Question 4.2)

### Question 4.2: Describe each cluster in a single sentence understandable by a Sales Director.

The final K-Means model ($k=5$, random_state=42) produced the following quantitative customer breakdown across 5,852 total accounts and 17,068,582.70 £ in total realized revenue:

| Cluster | Business Name | Customers Count | % Customers | Mean Recency (days) | Mean Frequency (orders) | Mean Monetary (£) | Total Revenue (£) | % Revenue |
|---|---|---|---|---|---|---|---|---|
| **0** | **VIP customers** | 454 | 7.76% | 17.3 | 32.6 | 20,635.50 £ | 9,368,515.70 £ | **54.89%** |
| **1** | **Loyal customers** | 1,284 | 21.94% | 48.8 | 9.1 | 3,627.24 £ | 4,657,379.30 £ | **27.29%** |
| **2** | **Occasional customers** | 1,141 | 19.50% | 32.5 | 2.5 | 666.19 £ | 760,120.40 £ | **4.45%** |
| **3** | **Asleep customers** | 1,352 | 23.10% | 288.7 | 3.8 | 1,375.00 £ | 1,858,990.00 £ | **10.89%** |
| **4** | **Inactive customers** | 1,621 | 27.70% | 415.6 | 1.2 | 261.31 £ | 423,577.30 £ | **2.48%** |

#### One-Sentence Executive Summaries for the Sales Director:

1. **VIP customers (Cluster 0):**
   > *"Our elite core of 454 top-tier wholesale accounts who purchase every couple of weeks (averaging 33 orders) and generate nearly 55% of the company's entire revenue."*
2. **Loyal customers (Cluster 1):**
   > *"Our highly reliable repeat commercial clients who order every month or two (averaging 9 orders) and form the steady revenue backbone providing over 27% of our business."*
3. **Occasional customers (Cluster 2):**
   > *"Newly acquired or light buyers who have ordered recently (around a month ago) with small test baskets (averaging 2-3 orders and 666 £) who require targeted upsell campaigns to turn into loyal recurring accounts."*
4. **Asleep customers (Cluster 3):**
   > *"Substantial former repeat buyers who spent over 1,300 £ across roughly 4 orders historically, but have gone completely silent for the past 9 to 10 months and are on the brink of total churn."*
5. **Inactive customers (Cluster 4):**
   > *"Dormant one-time shoppers who made a single low-value purchase (averaging 261 £) over 14 months ago and should be excluded from expensive sales outreach."*


## 4. Comparison with Day 3 Rule-based RFM (Task 4.4 / Questions 4.3 & 4.4)

### Question 4.3: Do K-Means clusters and RFM segments align? Where do they diverge the most?

#### 1. Empirical Alignment Across Methodologies (`pd.crosstab`)
Cross-tabulation matrix between Day 3 RFM rule-based segments and Day 4 K-Means clusters:

| RFM Segment (Day 3) | VIP customers | Loyal customers | Occasional customers | Asleep customers | Inactive customers | Total |
|---|---|---|---|---|---|---|
| **Champions** | **362** | **392** | 76 | 0 | 0 | 830 |
| **Loyal Customers** | 86 | **805** | 258 | 2 | 0 | 1,151 |
| **Potential Loyalists** | 0 | 51 | **661** | 1 | 0 | 713 |
| **New Customers** | 0 | 1 | **57** | 0 | 0 | 58 |
| **Promising** | 0 | 45 | **89** | 0 | 0 | 134 |
| **Need Attention** | 0 | **189** | 17 | 0 | 0 | 206 |
| **Cannot Lose Them** | 5 | 33 | 0 | **34** | 0 | 72 |
| **At Risk** | 0 | 0 | 0 | **652** | 102 | 754 |
| **Asleep** | 0 | 0 | 0 | **412** | 22 | 434 |
| **Lost** | 0 | 0 | 0 | 205 | **1,295** | 1,500 |
| **Total** | **454** | **1,284** | **1,141** | **1,352** | **1,621** | **5,852** |

#### 2. Where They Align:
- **At the Behavioral Extremes:** 
  - There is nearly unanimous agreement on dead accounts: 1,295 out of 1,500 `Lost` customers from RFM fall directly into K-Means `Inactive customers`, with the remainder in `Asleep customers`. Zero `Lost` accounts touch active clusters.
  - Similarly, `Champions` are captured almost exclusively by `VIP customers` (362) and `Loyal customers` (392).
  - `Potential Loyalists` and `New Customers` overwhelmingly converge into `Occasional customers` (661 and 57 respectively).

#### 3. Where They Diverge the Most:
- **The High-Value Split (`Champions` vs. `VIP customers`):** 
  Rule-based RFM assigned 830 customers into `Champions` based on fixed top-quantile scores ($R \in [4, 5], F \in [4, 5], M \in [4, 5]$). K-Means proved that this cohort is actually heterogeneous: it separated the extreme multi-million-pound wholesale tier (`VIP customers`, 454 accounts averaging 20,635 £) from standard recurring commercial buyers (`Loyal customers`, averaging 3,627 £).
- **The Middle and At-Risk Tiers (`At Risk`, `Cannot Lose Them`, `Need Attention`):** 
  Fixed quantile thresholds create artificial boundary cutoffs (e.g. a customer scoring 2.9 vs 3.1 in Recency lands in completely different RFM buckets). K-Means groups customers by continuous distance in 3D log space:
  - `At Risk` customers are partitioned naturally between `Asleep customers` (652 accounts with higher past engagement) and `Inactive customers` (102 accounts with minimal past depth).
  - `Cannot Lose Them` (72 accounts) is split between `Asleep customers` (34 accounts whose recency has degraded significantly) and `Loyal customers` / `VIP customers` (38 accounts whose combined monetary-frequency strength keeps them closer to active centroids despite declining recency).


### Question 4.4: Which approach would you use in production and for what purpose? Mention one advantage of each.

#### 1. Recommended Production Architecture: Dual-Layer Segmentation
In enterprise environments, these two approaches should not be treated as mutually exclusive competitors, but rather as complementary analytical layers:

```
+---------------------------------------------------------------------------------------+
| STRATEGIC LAYER: K-Means Clustering (Quarterly / Biannual Re-clustering)             |
| Used by: Executive Leadership, Finance, Supply Chain, Strategic Marketing             |
| Purpose: Macro customer portfolio health, budget sizing, SLA tier allocation          |
+---------------------------------------------------------------------------------------+
                                           |
                                           v
+---------------------------------------------------------------------------------------+
| OPERATIONAL LAYER: Rule-Based RFM Scoring (Daily / Weekly Automated Batch Jobs)       |
| Used by: CRM Teams, Marketing Automation Platforms (HubSpot, Braze, Klaviyo), Sales   |
| Purpose: Automated drip workflows, instant trigger campaigns, discount entitlement    |
+---------------------------------------------------------------------------------------+
```

#### 2. Specific Production Purpose and Key Advantage for Each:

| Approach | Production Purpose | Distinct Primary Advantage |
|---|---|---|
| **Rule-Based RFM** | **Daily CRM & Marketing Automation:** Driving daily rule-based workflows, automated re-engagement email triggers, loyalty tier progression, and sales rep alert tasks based on simple scalar criteria. | **100% Deterministic Transparency and Ease of Communication:** Business stakeholders and marketing teams immediately understand why a customer is in a segment without understanding machine learning. Rules can be updated instantly in CRM software without retraining pipelines or dealing with centroid drift. |
| **K-Means Clustering** | **Executive Strategy & Budget Allocation:** Portfolio segmentation, designing tiered commercial service level agreements (e.g., dedicating account managers only to the 454 VIPs), setting inventory allocations, and identifying natural customer clusters across non-linear boundaries. | **Data-Driven Multi-Dimensional Discovery without Arbitrary Cutoffs:** Finds natural behavioral densities in the continuous joint distribution of data rather than imposing artificial 20% quintile buckets. Prevents treating customers sitting right on an arbitrary quantile boundary as radically different personas. |

---

# Day 5 Question Answers — Predictive Modeling: Customer Return Prediction

The answers below are derived directly from the code, execution outputs, and machine learning models in [05_predict.ipynb](notebooks/05_predict.ipynb) and the accompanying assignment documentation.


## 1. The Intentional Error & Data Leakage (Task 5.1)

### Task 5.1 Question: Why is the naive model result (Accuracy > 0.98, ROC AUC > 0.999) completely worthless?

In Task 5.1, an intentional modeling mistake was engineered:
- Recency was calculated across the entire dataset: `Recency = (max_date + 1 - latest_date).days`.
- The target was defined as: `Goal = 1 if Recency <= 30 else 0`.
- A Logistic Regression classifier was trained using features `[Recency, Frequency, Monetary]` to predict `Goal`.
- Test Evaluation: **Accuracy = 0.9801 (98.01%)**, **ROC AUC = 0.9998 (~1.0000)**.

#### Why This Result Is Completely Worthless:
1. **Direct Mathematical Tautology (Self-Fulfilling Target):**
   The target variable `Goal` is literally defined as a deterministic mathematical threshold on the input feature `Recency` ($Recency \le 30$). Feeding `Recency` into the model as an explanatory feature to predict `Goal` means the algorithm is not forecasting customer purchasing behavior; it is simply learning a single, trivial mathematical inequality ($x \le 30 \implies y = 1$).
2. **Temporal Data Leakage (Using the Unobservable Future):**
   In this naive setup, `Recency` incorporates all transactions up to the very last timestamp in the dataset. In a real-world production deployment today, we cannot know a customer's transaction recency 30 days in the future unless we have already lived through that future. Predicting the present using future knowledge produces near-perfect historical evaluation metrics that fail completely when deployed in live environments.


## 2. Rigorous Time-Based Formulation & Feature Engineering (Tasks 5.2 & 5.3)

### 1. Chronological Timeline Split Architecture
To prevent any forward-looking data leakage, the timeline was split into two separate windows:

```
|------------------- OBSERVATION WINDOW (PAST) -------------------|--- TARGET WINDOW (FUTURE) ---|
           Features calculated STRICTLY from here                 ↑       Measure TARGET here
                                                             CUTOFF DATE
                                                             2011-09-10
                                                      (Max Date − 90 days)
```

- **Maximum Dataset Date:** `2011-12-09 12:50:00`
- **Cutoff Date (Max Date − 90 days):** `2011-09-10`
- **Past Transactions (Observation Window):** 618,316 transactions (79.6% of dataset)
- **Future Transactions (Target Window):** 158,263 transactions (20.4% of dataset)
- **Customer Modeling Cohort:** 5,256 unique customers active prior to the cutoff date
- **Target Definition (`Goal`):** Whether the customer placed at least one purchase in the 90 days following the cutoff date:
  - **Class 1 (Returned):** 2,289 customers (**43.55% ~ 43.6%**)
  - **Class 0 (Did Not Return):** 2,967 customers (**56.45% ~ 56.4%**)

### 2. Feature Set (Engineered Exclusively from Pre-Cutoff Data)
A total of 13 behavioral features were computed per customer using solely past data:
1. `Recency`: Elapsed days from the customer's last purchase prior to cutoff to the cutoff date.
2. `Frequency`: Number of distinct invoices placed before cutoff.
3. `Monetary`: Total gross expenditure (£) before cutoff.
4. `NumberOfItems`: Total line items (rows) purchased before cutoff.
5. `DistinctProducts`: Number of distinct product SKUs (`StockCode`) purchased before cutoff.
6. `AverageQuantity`: Mean units ordered per line item before cutoff.
7. `AveragePrice`: Mean unit price (£) of items purchased before cutoff.
8. `AverageCart`: Mean basket spend per invoice (`Monetary / Frequency`).
9. `CustomerTenure`: Total relationship age in days (`Cutoff Date - First Purchase Date`).
10. `AverageDaysBetweenPurchases`: Mean inter-purchase cadence (`CustomerTenure / (Frequency - 1)`).
11. `IsUK`: Binary indicator whether the customer's country is the United Kingdom.
12. `OrdersLast90Days`: Number of invoices placed in the 90-day window immediately preceding the cutoff date (`[Cutoff - 90d, Cutoff]`).
13. `SpendingLast90Days`: Total spend (£) in the 90-day window immediately preceding the cutoff date.


## 3. Model Training, Evaluation & Benchmarking (Task 5.4)

### 1. Experimental Setup
- **Train/Test Split:** 75% Training (3,942 customers) / 25% Testing (1,314 customers), stratified on `Goal`.
- **Test Class Distribution:** 742 Class 0 (56.47%) / 572 Class 1 (43.53%).
- **Models Benchmarked:**
  1. `DummyClassifier(strategy="most_frequent")` — Majority-class baseline.
  2. `LogisticRegression(class_weight="balanced", max_iter=2000)` — Linear model wrapped in a `Pipeline` with `StandardScaler`.
  3. `RandomForestClassifier(n_estimators=300, max_depth=12, min_samples_leaf=20, class_weight="balanced")` — Non-linear tree ensemble.

### 2. Comparative Evaluation Metrics (Test Set, $N = 1,314$)

| Model | Accuracy | Precision | Recall | F1 Score | ROC AUC |
|---|---|---|---|---|---|
| **Baseline (Dummy)** | 0.5647 (56.5%) | 0.0000 | 0.0000 | 0.0000 | 0.5000 |
| **Logistic Regression** | 0.7314 (73.1%) | 0.6918 (69.2%) | 0.6906 (69.1%) | 0.6912 (69.1%) | 0.7949 (~0.795) |
| **Random Forest** | **0.7435 (74.4%)** | **0.7205 (72.0%)** | **0.6713 (67.1%)** | **0.6950 (69.5%)** | **0.8047 (~0.805)** |

### 3. Confusion Matrices Breakdown (Test Set, $N = 1,314$)

```
Baseline (Dummy):
[[742    0]   --> TN = 742, FP = 0
 [572    0]]  --> FN = 572, TP = 0

Logistic Regression:
[[566  176]   --> TN = 566, FP = 176 (False alarms)
 [177  395]]  --> FN = 177 (Missed buyers), TP = 395 (Captured buyers)

Random Forest:
[[593  149]   --> TN = 593, FP = 149 (False alarms)
 [188  384]]  --> FN = 188 (Missed buyers), TP = 384 (Captured buyers)
```


## 4. Model Interpretation & Feature Significance (Task 5.5)

### 1. Feature Importance (Random Forest Gini Importance)
The Random Forest model identified customer recency and purchase cadence as the primary drivers of future returns:

| Rank | Feature | Relative Importance | Analytical Interpretation |
|---|---|---|---|
| **1** | `Recency` | **0.2052 (20.5%)** | The dominant predictor: recency of recent engagement is paramount. |
| **2** | `AverageDaysBetweenPurchases` | **0.1546 (15.5%)** | Customer order cadence: regular ordering cycles signal reliable return. |
| **3** | `NumberOfItems` | **0.1029 (10.3%)** | Total historical purchase depth and activity volume. |
| **4** | `SpendingLast90Days` | **0.0899 (9.0%)** | Recent monetary commitment immediately prior to the cutoff. |
| **5** | `Frequency` | **0.0898 (9.0%)** | Cumulative lifetime invoice count. |
| **6** | `Monetary` | **0.0890 (8.9%)** | Total historical spend. |
| **7** | `OrdersLast90Days` | **0.0629 (6.3%)** | Velocity of orders placed in the most recent 90-day window. |
| **8** | `DistinctProducts` | **0.0597 (6.0%)** | Product variety: catalog exploration correlates with relationship stickiness. |
| **9** | `AverageCart` | **0.0401 (4.0%)** | Average order size. |
| **10** | `CustomerTenure` | **0.0396 (4.0%)** | Length of commercial relationship. |
| **11** | `AveragePrice` | **0.0338 (3.4%)** | Mean unit price of purchased goods. |
| **12** | `AverageQuantity` | **0.0321 (3.2%)** | Bulk quantity per line item. |
| **13** | `IsUK` | **0.0003 (0.03%)** | Geography has virtually zero predictive power for repurchase probability. |

### 2. Logistic Regression Standardized Coefficients
Standardized coefficients indicate directional impact on the log-odds of a customer returning:
- **Strongest Positive Drivers (Increase probability of return):**
  - `DistinctProducts` (+0.5684): Broader catalog penetration creates switching costs and high retention.
  - `OrdersLast90Days` (+0.4792): Active purchasing velocity right before cutoff strongly indicates continuous buying.
  - `Frequency` (+0.3393): High lifetime purchase counts reinforce customer loyalty.
  - `SpendingLast90Days` (+0.1912): Recent capital allocated to the supplier.
  - `CustomerTenure` (+0.1178): Mature accounts have established ordering routines.
- **Strongest Negative Drivers (Decrease probability of return):**
  - `AveragePrice` (-0.5587): High unit prices often reflect bespoke or one-off equipment/clearance purchases with low repeat frequency.
  - `Recency` (-0.5393): As elapsed days since the last purchase increase, the likelihood of future return decays sharply.
  - `NumberOfItems` (-0.4109) & `AverageDaysBetweenPurchases` (-0.2949): Longer intervals between orders naturally decrease the chance of re-ordering within a tight 90-day forward window.


## 5. Answers to Core Notebook Questions

### Question 5.1: Explain in your own words what data leakage is and how you avoided it.
1. **What Data Leakage Is:**
   Data leakage occurs when information from outside the training dataset—specifically from the target variable itself, future events, or the validation/test fold—inadvertently contaminates the training feature space. This provides the machine learning model with an artificial, unrealizable advantage during training. The model learns spurious shortcuts rather than true underlying behavioral signals, resulting in near-perfect evaluation scores during offline development (such as the 0.9998 AUC observed in Task 5.1) that collapse completely into failure when deployed in live production.
2. **How We Avoided It:**
   - **Chronological Cutoff Boundary:** We strictly enforced a temporal split (`Cutoff Date = 2011-09-10`, 90 days before dataset end). All explanatory features were engineered exclusively from transactions dated *on or before* this cutoff point.
   - **Independent Future Target Window:** The prediction target (`Goal`) was derived strictly from transactions occurring *after* the cutoff date.
   - **Leak-Free Transformation Pipelines:** Preprocessing steps (e.g., `StandardScaler`) were fitted exclusively on training folds using `Pipeline`, ensuring test distribution parameters were never exposed to the model during training.

### Question 5.2: The baseline model has an accuracy of 56.5%, and your model achieves ~74%. Is your model good? Why is accuracy insufficient on its own here?
1. **Is the Model Good?**
   Yes, the model is highly effective. Moving from a naive baseline of 56.5% to **74.4% accuracy**, accompanied by an **ROC AUC of 0.805** and balanced Precision (72.0%) and Recall (67.1%), represents a strong, commercially actionable predictive signal for customer repurchase forecasting. It successfully identifies 2 out of every 3 returning customers while maintaining high precision.
2. **Why Accuracy Is Insufficient on Its Own:**
   - **Class Imbalance & Trivial Baselines:** In our test set, 56.5% of customers did not return. A simple `DummyClassifier` that blindly predicts "No customer will ever return" achieves 56.5% accuracy with zero predictive intelligence, zero recall, and zero business value.
   - **Asymmetry of Business Costs:** Accuracy treats False Positives (spending a marketing budget on a non-returner) and False Negatives (ignoring a returning or churning customer) as equally bad. In real commerce, the financial loss of losing a high-value customer dwarfs the cost of sending an unnecessary message.
   - **Single Fixed Decision Threshold:** Accuracy only evaluates model performance at an arbitrary probability threshold of 0.5. It does not measure the quality of probability ranking across different operating points, which is properly captured by ROC AUC (0.805).

### Question 5.3: Explain the confusion matrix in words: what is a "false alarm" (False Positive) and what is a "missed customer" (False Negative)? Which error is more costly for the business and why?
1. **Verbal Explanation of Errors (Test Set, Random Forest):**
   - **False Positive ("False Alarm", $FP = 149$):** The model predicts that the customer *will return*, but in reality, the customer *does not return* (they stay inactive/churned).
   - **False Negative ("Missed Customer", $FN = 188$):** The model predicts that the customer *will not return*, but in reality, they *do return* (or they lapse without being identified for retention).
   - **True Positive ($TP = 384$):** Correctly predicted returners.
   - **True Negative ($TN = 593$):** Correctly predicted dormant customers.
2. **Which Error Is More Costly and Why?**
   The relative cost depends on the commercial action tied to the prediction:
   - **In Customer Retention & Churn Prevention:** A **False Negative is far more costly**. If a churning commercial client is classified as a non-buyer and neglected, the company loses their entire future Customer Lifetime Value (CLV), which often amounts to thousands of pounds in wholesale B2B commerce. Conversely, a False Positive merely incurs the minor cost of a marketing communication or check-in call.
   - **In High-Cost Direct Incentive Campaigns:** If each positive prediction triggers an expensive physical gift or substantial commercial credit (£50–£100), a **False Positive** wastes hard cash on dead accounts who never redeem or purchase.
   - In standard B2B enterprise sales, **False Negatives are generally 5× to 10× more expensive**, because retaining an existing customer costs significantly less than acquiring a replacement.

### Question 5.4: `Recency` is the most important feature. Is this the same data leakage as in 5.1? Explain — this is the core lesson of the day.
**No, it is fundamentally NOT the same!** This distinction constitutes the single most important conceptual lesson of predictive time-series modeling:
- **In Task 5.1 (Leakage):** `Recency` was calculated over the *entire dataset* (including the future observation period), and the target itself was explicitly defined as $Recency \le 30$. The model was given the answer to the test; `Recency` was mathematically identical to the target.
- **In Task 5.2/5.3 (Valid Predictive Signal):** `Recency` was calculated strictly from past transactions *relative to the historical cutoff date* (`Cutoff - Latest Pre-Cutoff Date`). It measures how recently the customer purchased *before the cutoff*. The target measures whether the customer made any purchase *in the subsequent 90-day future window*.
- Past recency is a legitimate behavioral proxy: a customer who placed an order 4 days before the cutoff date is organically much more likely to order again in the next 90 days than a customer whose last purchase was 300 days prior. It respects the arrow of time: using past behavioral momentum to forecast future actions across an impermeable chronological boundary.

### Question 5.5: The company decides to send a €5 promotional coupon to customers predicted to return. Would you raise or lower the classification decision threshold from 0.5? What additional information do you need to answer definitively?
1. **Decision on the Classification Threshold:**
   - **Commercial Dilemma:** Sending coupons to customers who have a high organic propensity to return (e.g. $p > 0.8$) means subsidizing purchases that would have happened anyway ("cannibalizing margin"). Conversely, sending coupons to customers with extremely low propensity ($p < 0.2$) wastes coupon budget on dead leads who will never respond.
   - If the business mandate is strictly to reward customers predicted to return with minimal wasted marketing spend, we should **raise the decision threshold (e.g., to 0.65 or 0.70)**. This increases Precision, ensures coupons are sent only to high-confidence buyers, and eliminates costly False Positives.
   - *Alternative Marketing Best Practice (Uplift):* In modern marketing, the coupon should be targeted neither at guaranteed returners nor at dead accounts, but specifically at the **"Persuadables"**—customers with moderate probabilities (e.g., $p \in [0.35, 0.60]$) where an economic incentive generates true incremental lift.
2. **Additional Information Required for a Mathematically Optimal Decision:**
   To compute the exact profit-maximizing threshold $p^*$, we require four specific financial parameters:
   1. **Average Order Value (AOV) and Gross Margin ($M$):** How much gross profit in Euros is generated by an incremental purchase order?
   2. **Incremental Lift (Redemption Lift, $L$):** What proportion of targeted customers only buy *because* of the €5 coupon compared to an uncontacted control group?
   3. **Cannibalization / Deadweight Loss:** What percentage of coupon recipients would have purchased at full price without the €5 discount?
   4. **Delivery & Fulfillment Cost ($C$):** The cost of delivering the communication (email, SMS, direct mail) plus the €5 face value.
   With these inputs, the expected profit function $\mathbb{E}[\Pi(p)] = \sum (\text{Revenue Lift} - \text{Campaign Costs})$ can be optimized to select the exact optimal threshold $p^*$.


