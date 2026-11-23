# Student Internship — Online Retail Data Analysis

**Duration:** 7 working days (can be shortened to 5 — see *Shortened Version* at the end)  
**Tools:** Python (pandas, matplotlib, scikit-learn)  
**Data:** `podaci.xlsx` — approximately 1,000,000 transactions from a UK online retailer of gifts and homeware, period **01/12/2009 – 09/12/2011**

---

> **Stuck?** Accompanying this assignment is **[STUDENT_VODIC.md](STUDENT_VODIC.md)** —
> a detailed step-by-step guide with code, expected numbers after each
> operation, and result figures.
>
> This document explains **what** needs to be done. The guide explains **how**.
> Try doing it yourself first — the guide is there when you get stuck, not as a replacement for critical thinking.

---

## What is the Goal

To experience the **entire lifecycle of a data analysis project**: from a raw Excel file that nobody has inspected, to a business report readable by someone who does not know Python.

By the end of the internship, you should be able to answer:

- How reliable is our data in the first place, and what did we have to filter out?
- Who are our customers and how do they differ?
- Which customers drive revenue, and which ones cost us money?
- Can we predict who will return to make another purchase?
- What specific actions do we recommend to the business?

> **The single most important rule of the entire internship:**  
> A chart without a conclusion is worthless. **Every single chart** must be accompanied by at least one sentence describing what it shows and what that means for the business. If you cannot write that sentence, you do not need that chart.

---

## Deliverables

| # | Deliverable | Deadline |
|---|---|---|
| 1 | Jupyter notebook or `.py` script for each day | End of each day |
| 2 | `grafovi/` folder containing all figures/plots | Ongoing |
| 3 | **Completed `odgovori.csv`** | Ongoing, final on Day 7 |
| 4 | Final report (2–4 pages, Markdown or PDF) | Day 7 |
| 5 | Short presentation (5–8 slides, 10 minutes) | Day 7 |

Everything must be tracked in a Git repository. Commit at the end of each day with a meaningful commit message (e.g., `dan3: RFM analysis and segments`, not `update`).

### About the `odgovori.csv` File

Copy `sablon_odgovori.csv`, name it `odgovori.csv`, and **at the end of each day, fill in the numbers you obtained**. It looks like this:

```csv
kljuc,vrednost
dan1_redova_posle_ciscenja,776579
dan1_broj_kupaca,5852
...
```

The mentor validates this automatically using a single command. Therefore:

- **Enter the number you ACTUALLY obtained**, not the number you think is expected.
- If you haven't finished something yet — leave it blank or enter `?`.
- Do not modify the `kljuc` column and do not delete rows.

> This is not a copy-paste test. If your number differs, the automated validation will indicate **what likely went wrong** — giving you much faster feedback than waiting for your mentor.

---

## Environment Setup (complete before Day 1)

```bash
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # Linux / macOS

pip install pandas numpy openpyxl pyarrow matplotlib scikit-learn jupyterlab tabulate
```

Verify that everything works:

```python
import pandas as pd
df = pd.read_excel("podaci.xlsx", sheet_name=None)
print(list(df.keys()))
# Expected: ['Year 2009-2010', 'Year 2010-2011']
```

> **A tip that will save you hours:** Reading this Excel file takes **1.5–2 minutes**.  
> Do not re-read it in every notebook cell. Read it once, save it to a fast format, and work with that format going forward:
> ```python
> df.to_parquet("podaci_sirovo.parquet")   # saving
> df = pd.read_parquet("podaci_sirovo.parquet")   # loading takes under 2 seconds
> ```

---

## How the Tasks Are Structured

Each day includes:

- **Tasks** — specific steps to complete.
- **✅ Checkpoints** — numbers you should get. If your number differs significantly, you made an error somewhere — go back and verify.
- **❓ Questions** — answer them in words within the notebook. This is the most heavily graded part.
- **⚠️ Pitfalls** — common traps where mistakes happen most often.

Checkpoints are designed to **relieve uncertainty**, not constrain you. If you obtain a different number and can justify why (e.g., a different cleaning order) — that is completely acceptable, just document your reasoning.

---

# DAY 1 — Data Loading and Cleaning

**Goal:** Arrive at a dataset you can trust, knowing exactly what was removed and why.

## Tasks

### 1.1 Load both sheets and concatenate them

The file contains **two sheets**: `Year 2009-2010` and `Year 2010-2011`. You need a single DataFrame containing all the data.

```python
listovi = pd.read_excel("podaci.xlsx", sheet_name=None)
```

### 1.2 Explore the data

Call `.info()`, `.describe()`, `.head()`, `.isna().sum()`, and `.nunique()`.

❓ **Questions:**
- How many rows and columns does the dataset have?
- Which columns have missing values and how many?
- Look at the `min` for `Quantity` and `Price` in `.describe()`. What do you observe, and how do you explain negative values?
- Columns are named `Invoice`, `Price`, and `Customer ID`. In most tutorials online, they are named `InvoiceNo`, `UnitPrice`, and `CustomerID`. Rename them to their standard longer forms — why is having an agreed naming convention a good idea?

### 1.3 ⚠️ Check if the sheets overlap

This is the **most critical step of the day**, yet almost everyone skips it.

Compare the minimum and maximum `InvoiceDate` for each sheet. Then check if there are invoice numbers that appear in **both** sheets.

❓ **Questions:**
- Do the date ranges overlap? In what exact date interval?
- How many invoices exist in both sheets?
- If those rows had remained duplicated, what would have happened to the reported sales figures for December 2010?

### 1.4 Clean the data — in this exact order

Work **step by step**, and record how many rows remain after each step.

| Step | Action |
|---|---|
| 1 | Remove duplicate rows (identical rows) |
| 2 | Extract cancellations — `InvoiceNo` starting with `C` — into a **separate** DataFrame |
| 3 | Remove rows with missing `CustomerID` |
| 4 | Remove administrative/service codes: `POST`, `DOT`, `M`, `C2`, `D`, `S`, `B`, `BANK CHARGES`, `ADJUST`, `AMAZONFEE`, `CRUK`, `PADS`, `TEST001`, `gift_0001_*` |
| 5 | Remove `Quantity <= 0` |
| 6 | Remove `UnitPrice <= 0` |

⚠️ **Do NOT permanently discard cancellations** — save them to a separate file. You will need them on Day 6, and this is one of the most interesting analyses of the entire internship.

### 1.5 Create derived columns

`TotalPrice = Quantity * UnitPrice`, plus `Godina` (Year), `Mesec` (Month), `Datum` (Date), `DanUNedelji` (DayOfWeek), and `Sat` (Hour) extracted from `InvoiceDate`.

### 1.6 Save the result

Save the cleaned dataset to `.parquet` — each subsequent day should start by loading this file, not by re-cleaning the data from scratch.

## ✅ Checkpoints

| Metric | Expected Value |
|---|---|
| Raw rows | **1,067,371** |
| Columns (excluding any sheet identifier column added) | 8 |
| Missing `Customer ID` (raw) | 243,007 (22.8%) |
| Missing `Description` (raw) | 4,382 |
| Sheet date overlap period | **01–09/12/2010** |
| Invoices appearing in both sheets | **1,088** |
| Duplicates removed | ~34,335 |
| Cancelled items | ~19,104 (across ~8,292 invoices) |
| **Rows after cleaning** | **~776,579 (72.8% of original)** |
| Unique customers | ~5,852 |
| Unique invoices | ~36,594 |
| Unique items | ~4,620 |
| Total revenue (`TotalPrice`) | ~17,068,583 |

## ❓ Questions for the Notebook

1. What percentage of the data did you filter out? Is that too much?
2. Rows without `CustomerID` make up almost a quarter of the dataset. Why do we discard them for customer-level analysis, but would **not** discard them when reporting overall business sales?
3. After removing cancellations, how many rows with `Quantity <= 0` remained? What does this tell you about the source of negative quantities?

## 📦 Day 1 Deliverable

A notebook with a table: "cleaning step → rows removed → justification" along with brief written commentary.

---

# DAY 2 — Exploratory Data Analysis (EDA)

**Goal:** Understand the distribution and properties of the data before building any models.

## Tasks

### 2.1 Distributions of key variables

Plot histograms for `Quantity`, `UnitPrice`, and `TotalPrice`.

⚠️ The initial histogram will look like a **single bar squished against zero**. This is not an error — that is how highly skewed distributions look. Plot the same histogram on a **logarithmic scale** (`ax.set_xscale("log")`) and compare.

Also plot box plots and compute the number of outliers using the IQR rule.

### 2.2 Inspect the largest values

Print the top 10 line items by `TotalPrice`.

❓ Are these data errors or genuine purchases? How did you determine that? Should they be removed?

### 2.3 Sales by country

Compute revenue, customer count, and average basket value per country.

⚠️ One single country will completely dominate the chart. Structure the visualization so that both its dominance and the distribution of the remaining countries are clear.

### 2.4 Sales over time

Create monthly and daily aggregations of `TotalPrice` and plot the time series. Add a 7-day rolling moving average to the daily time series.

⚠️ **Pitfall:** The final month in the dataset is **incomplete** (data ends on 09/12/2011). If you plot it like the other months, it will look like a catastrophic sales collapse. Distinguish it visually or exclude it from the comparison.

### 2.5 When customers buy

Create a `day of week × hour of day` heatmap showing total revenue.

## ✅ Checkpoints

| Metric | Expected Value |
|---|---|
| `Quantity` — median / mean | 6 / 13.5 |
| `UnitPrice` — median / mean | 1.95 / 2.93 |
| `TotalPrice` — median / mean | 12.45 / 21.98 |
| UK share of total revenue | **~83.7%** |
| Top 5 countries excluding UK | EIRE, Netherlands, Germany, France, Australia |
| Highest full month | **2010-11**, ~1,155,978 |
| Lowest month | 2011-02, ~442,294 |
| Saturday share of revenue | **~0.06%** |

## ❓ Questions for the Notebook

1. Why is the mean of `TotalPrice` nearly double the median? Which of these two metrics would you report to an executive, and why?
2. Ireland (EIRE) and the Netherlands rank high in revenue but have **very few customers**. Calculate the average order basket per country. What can you deduce about the nature of these customers?
3. Is seasonality present? Does the pattern repeat across both years? Why is repetition across years essential for drawing conclusions?
4. What happens on Saturdays? Is this a data error?
5. Examine the heatmap. At what times of day do purchases occur? What does this suggest regarding whether customers are B2C consumers or B2B businesses?

## 📦 Day 2 Deliverable

A notebook containing at least 5 charts, **each with written business takeaways directly beneath it**.

---

# DAY 3 — Products and RFM Analysis

**Goal:** Discover what sells and who our customers are.

## Tasks

### 3.1 Best-selling products

Create **two** top 10 lists: by total quantity sold and by total revenue generated.

❓ How many products appear on both lists? What does this imply?

### 3.2 Pareto analysis

Sort products in descending order by revenue, compute cumulative percentage, and plot the Pareto curve.

❓ What percentage of product catalog items drives 80% of revenue? Does the 80/20 rule hold?

### 3.3 RFM analysis

For each customer, calculate:

- **Recency** — number of days since their last purchase
- **Frequency** — number of distinct orders (`nunique` of invoices, not `count`!)
- **Monetary** — total spend

⚠️ **Pitfall:** For Recency, you need an analysis reference date. **Do not use today's date** — the dataset is from 2011, and using today's date will classify all customers as "lost" 15 years ago. Use `max(InvoiceDate) + 1 day`.

### 3.4 Scoring and segments

Divide customers into 5 groups (quintiles) across each of the three dimensions, assigning scores from 1 to 5.

⚠️ For Recency, **lower is better** — a customer who purchased 3 days ago receives a score of 5.

⚠️ **Technical Pitfall:** Using `pd.qcut` on `Frequency` will throw an error or produce empty bins because many customers have the exact same frequency value (1 or 2). Solution: rank first using `series.rank(method="first")`.

Then assign business-meaningful segment names based on R and F scores, for example:

| R | F | Segment |
|---|---|---|
| 5 | 4–5 | Champions |
| 3–4 | 4–5 | Loyal Customers |
| 4–5 | 2–3 | Potential Loyalists |
| 5 | 1 | New Customers |
| 1–2 | 3–4 | At Risk |
| 1–2 | 5 | Cannot Lose Them |
| 1–2 | 1–2 | Lost |

(Define all remaining combinations — there must be 25 combinations total.)

## ✅ Checkpoints

| Metric | Expected Value |
|---|---|
| Number of products (grouped by `StockCode` + `Description`) | ~5,274 |
| Number of unique `StockCode` | ~4,620 |
| Top product by revenue | **REGENCY CAKESTAND 3 TIER** (~277,656) |
| Top product by quantity | **WORLD WAR 2 GLIDERS ASSTD DESIGNS** (~105,185 units) |
| Pareto: % of catalog items for 80% of revenue | **~22%** |
| Customers in RFM table | **5,852** |
| Recency — median | ~95 days |
| Frequency — median | ~3 |
| Monetary — median / mean | ~856 / ~2,917 |
| "Champions" segment | **~14% of customers → ~51% of revenue** |

## ❓ Questions for the Notebook

1. Why do the top lists by quantity and revenue differ so substantially?
2. Grouping by `StockCode` yields ~4,620 products, while grouping by `StockCode` + `Description` yields ~5,274. Where does this discrepancy come from? What does it imply about description reliability, and which key should be used?
3. One product in the top revenue list has only **1 customer**. Find it. Is this normal, and how should it be treated in the analysis?
4. Median spending is ~856, while the mean is ~2,917. Explain this divergence.
5. Which segment contains the largest number of customers? Which generates the most revenue? Are they the same segment? What does this mean for marketing budget allocation?
6. For the "Cannot Lose Them" segment — describe the profile of a typical customer and suggest a concrete retention action.

## 📦 Day 3 Deliverable

A notebook containing the RFM table, a segment summary table (customer count + revenue share per segment), and a segmentation visualization.

---

# DAY 4 — Customer Segmentation (K-Means)

**Goal:** Allow an unsupervised algorithm to find natural groupings and compare them with rule-based RFM segments.

## Tasks

### 4.1 Data preparation

Take `Recency`, `Frequency`, and `Monetary` from Day 3 and:

1. Apply `np.log1p()` to all three.
2. Standardize them using `StandardScaler`.

❓ Calculate `skew()` before and after the log transformation. Why are **both** steps mandatory? What would specifically happen to the clusters if you skipped standardization?

### 4.2 Determine the optimal number of clusters

For `k` ranging from 2 to 10, compute inertia (elbow method) and the `silhouette_score`. Plot both evaluation curves.

### 4.3 Cluster and interpret

Run K-Means with your chosen `k`. For each cluster, calculate the average R, F, M, customer count, and revenue share, and **give it a descriptive business name**.

⚠️ The name must be grounded in the numbers. Not "Cluster 3", but "VIP Customers — purchase every couple of weeks, average 33 orders".

### 4.4 Compare with Day 3

Build a `pd.crosstab` comparing rule-based RFM segments against K-Means clusters.

## ✅ Checkpoints

| Metric | Expected Value |
|---|---|
| `skew` of Monetary before log | ~25 |
| `skew` of Monetary after log | ~0.3 |
| Elbow suggests | **k = 5** |
| Highest silhouette score | **k = 2** (~0.44) |
| Highest value cluster | **~8% of customers → ~55% of revenue** |

## ❓ Questions for the Notebook

1. The silhouette score suggests `k=2`, while the elbow method suggests `k=5`. Which would you choose and **why**? (There is no single correct answer — your reasoning is what is evaluated.)
2. Describe each cluster in **a single sentence understandable by a Sales Director**.
3. Do K-Means clusters and RFM segments align? Where do they diverge the most?
4. Which approach would you use in production and for what purpose? Mention one advantage of each.

## 📦 Day 4 Deliverable

A notebook with elbow and silhouette plots, cluster visualizations, and **named** profiles for each cluster.

---

# DAY 5 — Prediction: Will the Customer Return?

> **This is the most critical day of the internship.** It highlights a trap that most beginners fall into, and one that interviewers notice immediately in job interviews.

**Goal:** Build a leak-free model and understand why a "lower" evaluation score here is actually the genuine and correct result.

## Tasks

### 5.1 First make the mistake — intentionally

Do exactly this:

1. Calculate RFM over the **entire** dataset.
2. Define target: `Target = 1 if Recency <= 30, else 0`.
3. Train a logistic regression model on features `Recency`, `Frequency`, `Monetary`.
4. Measure accuracy and ROC AUC.

✅ You will achieve accuracy above **0.97** and AUC above **0.99**.

❓ **Question:** Why is this result worthless? Inspect the target definition and feature set carefully.

### 5.2 Now formulate the task correctly

Split the timeline into two windows:

```
|------------------- PAST (OBSERVATION) ----------|--- FUTURE (TARGET) ---|
        features calculated ONLY from here         ↑   measure TARGET here
                                             CUTOFF DATE
                                      (Max Date − 90 days)
```

- **Cutoff Date** = Maximum date in dataset − 90 days
- **Features** — computed **strictly** from transactions prior to the cutoff date
- **Target** — whether the customer purchased anything **after** the cutoff date (1/0)
- Only customers who were active **before** the cutoff date enter the modeling dataset

### 5.3 Feature engineering

At the customer level, using exclusively past data: RFM, total items purchased, number of unique products, average order basket, relationship age (days from first purchase to cutoff), average inter-purchase interval, whether UK-based, and activity in the last 90 days before cutoff.

### 5.4 Train and compare models

You must evaluate **three** models:

1. `DummyClassifier(strategy="most_frequent")` — baseline
2. `LogisticRegression` — with `StandardScaler` inside a `Pipeline`
3. `RandomForestClassifier`

Split data into train/test using `stratify=y`. Measure Accuracy, Precision, Recall, F1, and ROC AUC. Plot confusion matrices and ROC curves.

### 5.5 Interpret the model

Extract `feature_importances_` from Random Forest and feature coefficients from Logistic Regression.

## ✅ Checkpoints

| Metric | Expected Value |
|---|---|
| AUC of flawed model from 5.1 | **~0.999** |
| Cutoff Date | **2011-09-10** |
| Customers in modeling dataset | ~5,256 |
| Percentage of returning customers | **~44%** |
| Baseline accuracy | ~56.5% |
| Logistic Regression — AUC | **~0.79** |
| Random Forest — AUC | **~0.80** |
| Top feature | **Recency** |

## ❓ Questions for the Notebook

1. Explain in your own words what data leakage is and how you avoided it.
2. The baseline model has an accuracy of 56.5%, and your model achieves ~74%. Is your model good? Why is accuracy insufficient on its own here?
3. Explain the confusion matrix **in words**: what is a "false alarm" (false positive) and what is a "missed customer" (false negative)? Which error is more costly for the business and why?
4. `Recency` is the most important feature. Is this the **same** data leakage as in 5.1? Explain — this is the core lesson of the day.
5. The company decides to send a €5 promotional coupon to customers predicted to return. Would you raise or lower the classification decision threshold from 0.5? What additional information do you need to answer definitively?

## 📦 Day 5 Deliverable

A notebook containing **both** models (flawed and leak-free), comparison tables, confusion matrices, and conclusions explaining which model is valid and why.

---

# DAY 6 — Advanced Topic (Choose One)

**Goal:** Autonomously tackle a problem that is not broken down step-by-step.

Here you receive fewer instructions by design. Select **one** option.

## Option A — Anomaly Detection (Cancellations)

Load the cancellations dataset saved on Day 1.

- What is the total value of cancelled orders, and what % of total gross revenue does it represent?
- How does the cancellation rate fluctuate over time by month? Set a threshold (e.g., `mean + 2σ`) and flag anomaly months.
- Which products have the highest cancellations — by total monetary value and by **rate** (cancelled / total ordered)?
- Are there specific customers with an unusually high cancellation rate?

✅ Checkpoints: ~8,292 cancelled invoices, value ~1,462,051 (**~8.6%** of revenue), 2011-12 is a prominent outlier (~40%).

❓ Find a product group with a cancellation rate **close to 50%**. What could be the underlying reason, and what would you recommend to management?

⚠️ Before concluding the cause, verify **how many distinct cancellation line items** account for that rate. A 50% rate stemming from a single large reversal vs. a 50% rate originating from seven different customers are **not the same phenomenon** and should not lead to the same recommendation.

## Option B — Sales Forecasting for Next Month

- Construct a daily time series of total revenue.
- Split into train/test **chronologically** (the final 60 days = test set). Why not randomly?
- Build at least three models, including a **naive baseline** (using the value from the same day of the previous week).
- Evaluate MAE, RMSE, and MAPE. Forecast the next 30 days.

⚠️ If you use lag features (`revenue 7 days ago`), you cannot forecast 30 days ahead without recursive multi-step forecasting. Consider which features can be computed for any future calendar date (e.g., calendar/date features).

✅ Checkpoints: Naive baseline MAPE ~33%, best model MAPE ~27%.

❓ Why is a ~27% error acceptable at a daily level, but would be unacceptable at an aggregated monthly level?

## Option C — Recommender System

- For the top N most frequent items (e.g., 300), calculate how often each pair co-occurs on the same invoice.
- Compute **support**, **confidence**, and **lift**.
- List the strongest product associations and create a function `recommend(stock_code)`.

⚠️ Across all ~4,600 items, the full co-occurrence matrix contains over 21 million pairs. Restrict yourself to frequent items — an association between two items that sold only 3 times is not statistically meaningful anyway.

✅ Checkpoints: On 300 items and ~33,000 invoices, you obtain ~28,000 pairs with lift > 1; the strongest pair achieves a lift of ~37.

❓ Inspect the strongest pairs — what do they have in common? What does this reveal about customer shopping patterns? What is the primary **limitation** of this recommender approach?

## 📦 Day 6 Deliverable

A notebook or `.py` script implementing the chosen topic + a concise summary of insights.

---

# DAY 7 — Final Report and Presentation

**Goal:** Communicate findings clearly to an audience that has never seen your code.

## Tasks

### 7.1 Write the report (2–4 pages)

Required sections:

1. **Executive Summary** — 5 key findings, each a single sentence supported by metrics.  
   *Write this last, read this first.*
2. **Data & Preparation** — what you received, what was removed, what remained.
3. **Key Findings** — 4–6 core insights, each accompanied by one chart.
4. **Predictive Modeling** — problem framing, validation results, business impact.
5. **Recommendations** — 5–7 actionable initiatives tied directly to findings.
6. **Limitations** — what your analysis **cannot** conclude or claim.

⚠️ The "Limitations" section is not an admission of weakness — it demonstrates that you understand the boundaries of your work. An analysis without stated limitations appears amateurish.

### 7.2 Create the presentation (5–8 slides, 10 minutes)

One slide = one core takeaway. No 20-row tables. No code snippets.

### 7.3 Prepare for Q&A

Be ready for questions such as:

- "Why did you discard a quarter of the raw data?"
- "How good is your predictive model really?"
- "What would you prioritize if you had another week?"
- "What evidence supports your claim that Champions are the most valuable cohort?"

## 📦 Day 7 Deliverable

Final report + presentation slides + clean Git repository.

---

## Grading Criteria

| Dimension | Weight | Criteria |
|---|---|---|
| Technical Correctness | 30% | Numerical accuracy, justified data cleaning, absence of data leakage |
| Business Insights | 30% | Clear business interpretations and actionable context for every finding |
| Code Quality | 20% | Readability, reproducibility, clear naming, no redundant copy-pasted blocks |
| Data Visualization | 10% | Intuitive charts that stand on their own, proper titles, and labeled axes/units |
| Communication | 10% | Report and presentation understandable to non-technical stakeholders |

**Automatic Grade Deductions:**

- Any chart presented without an analytical takeaway
- Using `datetime.now()` for Recency calculations
- Presenting a model with data leakage without noting it
- Claiming "98% accuracy" without comparing against a baseline
- Interpreting the incomplete month of December 2011 as a "sales collapse"

---

## Common Pitfalls from Previous Cohorts

1. **Skipping sheet overlap validation** → December 2010 figures are doubled.
2. **Using `count()` instead of `nunique()` for Frequency** → counts item rows instead of distinct orders.
3. **Using current date for Recency** → flags every customer as "churned/lost".
4. **Data leakage on Day 5** → gives a deceptively perfect score on a useless model.
5. **Failing to standardize features before K-Means** → Monetary scale alone dominates cluster boundaries.
6. **Plotting 15 charts without writing down conclusions.**
7. **Discarding outliers purely because they are outliers** → removes the largest wholesale customers (i.e., the core business).
8. **Stating "the average customer spends 2,917"** → an arithmetic mean that describes no actual individual customer.

---

## Shortened Version (5 Days)

| Day | Focus |
|---|---|
| 1 | Day 1 (Cleaning) |
| 2 | Day 2 (EDA) |
| 3 | Day 3 (Products + RFM) |
| 4 | Day 5 (Predictive Modeling) — **never skip this day** |
| 5 | Day 7 (Final Report) |

K-Means (Day 4) and Advanced Topics (Day 6) are omitted in the 5-day track. Day 5 is never omitted — understanding and preventing data leakage is the most valuable lesson of the entire internship.

---

## Glossary of Terms

| Term | Definition |
|---|---|
| **RFM** | Recency, Frequency, Monetary — three core metrics profiling customer behavior |
| **Outlier** | A data point significantly distant from others; not necessarily erroneous |
| **Skewness** | Asymmetry of a probability distribution; high skew implies the mean is non-representative |
| **Data Leakage** | When a model accesses information during training that would not be available at inference |
| **Baseline** | The simplest reference model (e.g., majority class); any valid model must outperform it |
| **Precision** | Out of all predicted positive cases, what fraction is truly positive |
| **Recall** | Out of all actual positive cases, what fraction was correctly identified |
| **ROC AUC** | Ranking capability of a binary classifier; 0.5 = random guess, 1.0 = perfect ranking |
| **Inertia** | Sum of squared distances of samples to their closest cluster center |
| **Silhouette Score** | Measure of how well-separated clusters are (ranging from −1 to 1) |
| **Lift** | How much more often an item pair is bought together compared to random chance |

---

*Direct all questions to your mentor. It is far better to clarify on Day 2 than to submit flawed work on Day 7.*
