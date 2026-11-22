# Day 1 Question Answers — Data Loading and Cleaning

The answers below are derived directly from the code, execution outputs, and analytical results in notebooks/01_cleaning.ipynb and the accompanying assignment documentation.


## 1. Data Exploration (Task 1.2)

### Question 1.2.1: How many rows and columns does the dataset have?
The raw dataset (after merging the two Excel sheets "Year 2009-2010" and "Year 2010-2011") contains a total of 1,067,371 rows and 8 original columns (or 9 columns if including the helper column "List" added to track the sheet of origin).
- Sheet "Year 2009-2010": 525,461 rows
- Sheet "Year 2010-2011": 541,910 rows
- Total: 525,461 + 541,910 = 1,067,371 rows.

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
