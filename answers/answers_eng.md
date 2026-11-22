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
