# 🛒 Corporación Favorita — Grocery Sales Analysis

<p align="center">
  <strong>Large-Scale Data Analysis | Exploratory Data Analysis | Business Intelligence</strong>
</p>

A large-scale data analysis project exploring grocery sales patterns, store performance, product contribution, and external factors using Python, Polars, and Power BI.

---

## 📌 Project Overview

This project analyzes historical grocery sales data from **Corporación Favorita**, an Ecuadorian grocery retailer, to uncover business patterns, evaluate store and product performance, and investigate potential relationships between sales activity and external factors.

The project focuses on transforming a large transactional dataset into meaningful business insights through data exploration, cleaning, analytical investigations, and interactive visualization.

Rather than building a forecasting model, the primary objective is to understand historical sales behavior and provide a structured view of business performance.

## 🎯 Project Highlights

- Processed and analyzed **125+ million sales records** using Polars.
- Performed Exploratory Data Analysis (EDA) and data quality validation across multiple datasets.
- Investigated sales performance across **54 stores** and **4,100 catalog products**.
- Analyzed temporal patterns, store performance, product contribution, and promotional activity.
- Conducted sensitivity analysis to evaluate the impact of retaining negative sales records.
- Developed an interactive **six-page Power BI report**.

---

## 📂 Dataset Overview

The project uses multiple datasets from the Corporación Favorita grocery sales dataset.

| Dataset | Description |
|---|---|
| `train.csv` | Historical daily sales records |
| `test.csv` | Store-product combinations for the forecasting period |
| `items.csv` | Product metadata and classifications |
| `stores.csv` | Store metadata and geographic attributes |
| `transactions.csv` | Daily transaction counts by store |
| `oil.csv` | Daily crude oil prices |
| `holidays_events.csv` | Holidays and event information |

### Dataset Statistics

| Metric | Value |
|---|---:|
| Historical sales records | 125,497,040 |
| Historical period | Jan 2013 – Aug 2017 |
| Stores | 54 |
| Catalog products | 4,100 |
| Product families | 33 |
| Store types | 5 |

**Primary Analytical Measure:** `unit_sales` — the number of units sold.

> **Important:** The analysis focuses on sales volume, not revenue or profit.

---

## 🛠️ Technology Stack

| Technology | Purpose |
|---|---|
| Python | Data analysis and processing |
| Pandas | Initial EDA and auxiliary dataset preparation |
| Polars | Large-scale data processing |
| Power BI | Interactive reporting and visualization |
| DAX | Analytical measures and KPIs |
| Git & GitHub | Version control and documentation |

### Why Polars?

The historical sales dataset contains more than **125 million records**. Polars was selected to handle large-scale data processing efficiently.

Its `LazyFrame` API allows query optimization and helps reduce unnecessary memory consumption during data transformations and aggregations.

**Example:**

```python
import polars as pl

sales = pl.scan_csv("train.csv")

schema = sales.collect_schema()

print(schema)
```

---

## ⚙️ Analytical Workflow

The project follows a structured analytical workflow:

### 1. Data Ingestion & Schema Inspection
- Load datasets.
- Inspect structures and data types.
- Examine initial dataset properties.

### 2. Exploratory Data Analysis (EDA)
- Investigate missing values.
- Analyze categorical distributions.
- Identify duplicate records.
- Examine temporal coverage and data consistency.

### 3. Data Cleaning & Transformation
- Standardize date fields.
- Handle categorical attributes.
- Validate numerical data types.
- Prepare relevant analytical indicators.

### 4. Data Quality Decisions
- Evaluate missing values.
- Investigate negative sales.
- Examine incomplete auxiliary records.
- Identify potential data inconsistencies.

### 5. Data Integration
- Validate relationships between sales records and reference datasets.
- Prevent unintended row multiplication during joins.

### 6. Analytical Investigation
- Explore temporal patterns.
- Evaluate store and product performance.
- Analyze promotional activity.
- Investigate potential external relationships.

### 7. Power BI Reporting
- Translate analytical findings into an interactive business report.
- Develop KPIs and visual comparisons.
- Present insights through six analytical pages.

For detailed explanations of processing decisions, validation procedures, formulas, and analytical limitations, see **[Methodology.md](Methodology.md)**.

---

## 📊 Key Business Insights

The following findings are based on the historical sales analysis.

### 1. Store Performance

- Store 44 generated approximately **62.09 million units**, making it the highest-volume store.
- The top five stores contributed approximately **24.35%** of total unit sales.
- The top ten stores accounted for approximately **40.17%** of total unit sales.
- Store Type A recorded the highest aggregate sales volume, with approximately **353.04 million units**, slightly exceeding Type D.
- Store Type D exhibited the highest relative variability, with a coefficient of variation of approximately **51.29%**.

### 2. Product Performance

Sales were concentrated among a small number of product families.

| Product Family | Unit Sales | Contribution |
|---|---:|---:|
| Grocery I | 343.46M | 31.99% |
| Beverages | 216.94M | 20.21% |
| Produce | 122.70M | 11.43% |
| Cleaning | 97.52M | 9.08% |
| Dairy | 64.49M | 6.01% |

**Key Finding:** Grocery I and Beverages together accounted for approximately **52.20%** of total unit sales.

The analysis also identified substantial differences in product-level sales concentration across product families.

### 3. Temporal Patterns

- December recorded the highest normalized monthly sales index across every complete year from 2013 through 2016.
- December's normalized seasonality index reached approximately **134.97**.
- Sunday recorded the highest weekday sales index at approximately **130.83**.
- Thursday recorded the lowest weekday sales index at approximately **79.49**.
- A fixed-store comparison using the same 46 stores showed an approximately **128.3% increase** in average monthly sales per store between January 2013 and July 2017.

These comparisons help distinguish recurring temporal patterns from changes in store coverage.

### 4. Promotion Analysis

Promotion-related comparisons were conducted using observations with recorded promotion indicators.

- In 2015, average unit sales per observed record were approximately **74.9% higher** under promotion.
- In 2016, the observed difference was approximately **60.9%**.
- In 2017, the observed difference was approximately **84.6%**.
- At the product level, **80.3%** of eligible products recorded higher average unit sales during promotional observations.

> **Analytical Note:** These findings represent observational associations, not causal estimates of promotional effectiveness.

### 5. Data Quality & Sensitivity Analysis

Several important data quality decisions were evaluated rather than handled through automatic deletion or imputation.

- Missing promotion indicators were retained as null values.
- Negative sales records were retained.
- Missing oil prices were handled using forward fill and backfill.
- Missing transaction records were not automatically interpreted as zero transactions.
- Reference relationships and sales keys were validated.

A sensitivity analysis showed that excluding negative sales records did not change the ranking of any of the 54 stores.

The difference represented approximately **0.0134% of net sales**.

---

## 📈 Power BI Dashboard

The final report consists of six analytical pages:

| Dashboard Page | Analytical Focus |
|---|---|
| Executive Overview | High-level business performance |
| Time Analysis | Temporal trends and seasonality |
| Store Performance | Store rankings, contributions, and comparisons |
| Product Performance | Product and product-family analysis |
| Geographic Analysis | Geographic and store-level distribution |
| External Factors & Drivers | Promotions and external contextual factors |

### Dashboard Preview

*Replace the image paths below with your actual dashboard screenshots.*

#### 1. Executive Overview

![Executive Overview](Images/executive_overview.png)

#### 2. Time Analysis

![Time Analysis](Images/time_analysis.png)

#### 3. Store Performance

![Store Performance](Images/store_performance.png)

#### 4. Product Performance

![Product Performance](Images/product_performance.png)

#### 5. Geographic Analysis

![Geographic Analysis](Images/geographic_analysis.png)

#### 6. External Factors & Drivers

![External Factors](Images/external_factors.png)

---

## 📁 Repository Structure

```text
Favorita-Grocery-Sales-Analysis/
│
├── README.md
├── Methodology.md
├── Stakeholder_Report.pdf
│
├── notebooks/
│   └── EDA and analytical notebooks
│
├── scripts/
│   └── Data processing scripts
│
├── data/
│   └── Dataset files or data references
│
├── powerbi/
│   └── Power BI report
│
└── images/
    └── Dashboard screenshots
```

> **Note:** The structure above describes the intended organization of the project repository. Update it to match the actual repository contents.

---

## 📚 Project Documentation

For a deeper understanding of the project:

| Document | Description |
|---|---|
| [Methodology.md](Methodology.md) | Detailed data processing, cleaning decisions, analytical methods, validation, and limitations |
| [Stakeholder_Report.pdf](Stakeholder_Report.pdf) | Business-oriented summary of analytical findings and recommendations |

---

## ⚠️ Analytical Limitations

Several considerations are important when interpreting the results:

- The primary metric is unit sales, not revenue or profit.
- The historical dataset ends on **August 15, 2017**, making 2017 a partial year.
- Promotion information is unavailable for all of 2013 and part of 2014.
- Negative sales values may represent returns or business adjustments.
- Some transaction records and oil observations are unavailable.
- Observed relationships between promotions, oil prices, events, and sales do not establish causation.
- Changes in store coverage may influence aggregate sales trends.

These limitations are documented and considered throughout the analytical process.

---

## 📝 Conclusion

This project demonstrates an end-to-end analytical workflow for working with large-scale transactional data.

It combines efficient data processing, exploratory analysis, data quality validation, statistical comparisons, and business intelligence reporting to transform raw historical sales records into meaningful insights.

The central objective is not simply to visualize sales figures, but to understand the patterns behind them, evaluate differences in business performance, and communicate analytical findings with appropriate methodological rigor.

---

<p align="center">
  <strong>Technologies:</strong> Python · Pandas · Polars · Power BI · DAX
  <br>
  <strong>Project Focus:</strong> Data Analysis · Exploratory Data Analysis · Business Intelligence · Large-Scale Data Processing
</p>
