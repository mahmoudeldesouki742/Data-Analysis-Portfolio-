# Methodology — Corporación Favorita Grocery Sales Analysis

## 1. Project Overview

This project analyzes the historical grocery sales data of Corporación Favorita, an Ecuadorian grocery retailer, to understand sales patterns, evaluate store and product performance, and investigate potential relationships between sales activity and external factors.

The project focuses on descriptive and diagnostic analytics, using historical sales data to identify business patterns, measure performance differences, and generate actionable insights.

Rather than building a sales forecasting model, the primary objective is to understand the underlying behavior of the business through large-scale data analysis and interactive reporting.

### Analytical Scope

The analysis covers:

- Historical sales trends and temporal patterns.
- Store-level sales performance and contribution.
- Product and product-family performance.
- Geographic and store-type comparisons.
- Promotion-related sales patterns.
- Potential relationships with transactions, holidays, events, and oil prices.
- Data quality assessment and validation.

### Dataset Coverage

| Attribute | Value |
|---|---|
| Historical period | January 1, 2013 – August 15, 2017 |
| Training records | 125,497,040 |
| Unique stores | 54 |
| Unique products in catalog | 4,100 |
| Product families | 33 |
| Store types | 5 |
| Main analytical measure | "unit_sales" |

> **Important:** The primary sales measure is "unit_sales", representing the number of units sold. It is not revenue or profit. Therefore, conclusions in this project concern sales volume rather than financial profitability.

---

## 2. Data Sources

The project uses multiple datasets from the Corporación Favorita grocery sales dataset.

| Dataset | Description | Main Purpose |
|---|---|---|
| "train.csv" | Historical store-item daily sales records | Primary analytical dataset |
| "test.csv" | Store-item combinations for the forecasting period | Structural validation and reference |
| "items.csv" | Product metadata | Product classification and family analysis |
| "stores.csv" | Store metadata | Geographic and store-type analysis |
| "transactions.csv" | Daily transaction counts by store | Transaction activity analysis |
| "oil.csv" | Daily crude oil prices | External economic factor |
| "holidays_events.csv" | Holidays, transferred holidays, and events | Calendar and event analysis |

The training dataset is the primary source of historical sales information. Auxiliary datasets provide additional attributes that support segmentation, comparisons, and contextual analysis.

---

## 3. Technology Stack

| Technology | Role |
|---|---|
| Python | Data processing and analytical workflows |
| Pandas | Initial EDA and processing of auxiliary datasets |
| Polars | Large-scale processing of the training dataset |
| Power BI | Interactive dashboards and business reporting |
| DAX | Analytical measures and KPI calculations |
| Git & GitHub | Project version control and documentation |

### 3.1 Large-Scale Data Processing

The training dataset contains more than 125 million records. To handle this volume efficiently, Polars was used instead of loading the entire dataset into memory through a conventional Pandas workflow.

The dataset was initially accessed using:

```python
import polars as pl

sales = pl.scan_csv("train.csv")
```

This creates a "LazyFrame", allowing query operations to be optimized and executed when required.

Lazy evaluation helps reduce unnecessary intermediate materialization and makes large-scale filtering, aggregation, and transformation more manageable.

Pandas was used for the initial exploration and preparation of smaller auxiliary datasets, while Polars handled the large historical sales dataset.

---

## 4. Data Ingestion and Initial Exploratory Data Analysis

The initial stage focused on understanding the structure, completeness, and reliability of each dataset before performing analytical transformations.

### 4.1 Schema Inspection

For each dataset, the initial inspection covered:

- Column names and available attributes.
- Data types and structural consistency.
- Initial records and dataset dimensions.
- Basic statistical properties where applicable.
- Compatibility between related identifiers and date fields.

Pandas inspection methods included:

```python
df.info()
df.head()
df.dtypes
```

For the training dataset, the schema was inspected through Polars.

### 4.2 Missing Value Analysis

Missing values were examined across the datasets to identify incomplete records and determine whether missing information represented a genuine data limitation or a value that could reasonably be estimated.

Examples included:

- Missing promotion indicators in the historical sales data.
- Missing daily oil prices.
- Missing transaction records for certain store-date combinations.
- Missing historical sales for some catalog products.

Missing values were not automatically treated as zero or false. Their meaning was evaluated according to the relevant business context.

### 4.3 Duplicate Verification

Potential duplicates were investigated using the logical identifiers of each dataset.

For historical sales, the logical key was:

```text
date + store_nbr + item_nbr
```

For transactions:

```text
date + store_nbr
```

For products and stores, the corresponding identifiers were checked for uniqueness.

The objective was to detect duplicated records that could distort aggregations, joins, and analytical results.

### 4.4 Categorical Integrity

Categorical fields were examined to understand their possible values and identify unexpected or inconsistent categories.

Examples included:

- "family", "class", and "perishable" in product metadata.
- "city", "state", "type", and "cluster" in store metadata.
- "type", "locale", "locale_name", and "transferred" in holiday/event data.

This stage helped establish the classification structure used in subsequent analyses.

---

## 5. Data Cleaning and Transformation

The cleaning process focused on standardizing data types, validating identifiers, addressing missing values, and preparing datasets for integration.

### 5.1 Date Standardization

Date columns were converted from string representations into native date or datetime types.

Pandas:

```python
pd.to_datetime(df["date"])
```

Polars:

```python
pl.col("date").str.to_date()
```

Proper date types were necessary for chronological filtering, time-based aggregation, and joining datasets using calendar dates.

### 5.2 Categorical and Text Standardization

String attributes were examined for formatting inconsistencies.

The standardization approach included:

- Removing leading and trailing whitespace.
- Applying consistent lowercase formatting where appropriate.
- Checking categorical values for unexpected variations.

Typical Pandas operations included:

```python
df["column"] = (
    df["column"]
    .str.strip()
    .str.lower()
)
```

Standardization was applied with consideration for the role of each attribute, especially when matching geographic and categorical values across datasets.

### 5.3 Boolean Standardization

Boolean indicators were examined to ensure consistent representation.

Relevant fields included:

- "onpromotion"
- "perishable"
- "transferred"

The final treatment of missing boolean values depended on their business meaning.

In particular, missing promotion indicators were retained as null values rather than automatically converted to "False".

### 5.4 Numerical Type Optimization

Numerical identifiers and measures were inspected and assigned appropriate numerical types.

Important fields included:

- "store_nbr"
- "item_nbr"
- "class"
- "cluster"
- "unit_sales"
- "transactions"
- "dcoilwtico"

For the large training dataset, type selection also considered memory usage and processing efficiency.

---

## 6. Final Data Quality Decisions

After the initial EDA and cleaning review, several data quality decisions were established. These decisions prioritized preserving valid business information and avoiding unsupported assumptions.

### 6.1 Missing Promotion Data

The training dataset contains:

- Total records: 125,497,040.
- Missing "onpromotion" values: 21,657,651.
- Missing percentage: 17.26%.

The missing values are distributed unevenly across the historical period:

| Year | Missing Promotion Data |
|---|---|
| 2013 | 100% |
| 2014 | 23.95% |
| 2015–2017 | No missing values |

**Decision:** Retain missing values as null.

**Reasoning:**

A missing promotion indicator does not necessarily mean that the product was not promoted. Replacing missing values with "False" would introduce an unsupported assumption and could distort promotion-related comparisons.

Consequently, promotion analysis must account for the limited availability of promotion information, particularly in 2013 and 2014.

### 6.2 Negative Sales Values

The training dataset contains 7,795 negative "unit_sales" records, representing approximately 0.0062% of all observations.

No zero sales values were found.

**Decision:** Retain negative sales values.

**Reasoning:**

Negative values may represent product returns, sales corrections, or other business adjustments.

Automatically removing them could eliminate legitimate business activity and introduce bias into aggregated sales measurements.

Their impact was investigated through a sensitivity analysis.

### 6.3 Missing Oil Prices

The original oil dataset contains:

- 1,218 records.
- 43 missing oil price observations.

Missing values were handled using:

- Forward fill ("ffill").
- Backfill ("bfill").

After processing, the oil dataset contained no missing price values.

**Decision:** Retain the processed dataset and document the imputation method.

**Reasoning:**

The imputation allows oil prices to be used in subsequent analysis. However, imputed values are estimates rather than original observations and must be considered when interpreting relationships between oil prices and sales.

### 6.4 Missing Oil Dates

A total of 481 distinct dates in the historical sales period do not have matching dates in the original oil dataset.

All these dates fall on Saturdays or Sundays.

**Decision:** Do not automatically classify these dates as data errors.

**Reasoning:**

Oil price observations are not necessarily available for every calendar day. Missing dates therefore require explicit handling during integration rather than being treated as evidence of invalid sales records.

### 6.5 Products Without Historical Sales

The item catalog contains 4,100 unique products, while the training dataset contains 4,036 unique products.

Therefore, 64 catalog products have no historical sales records during the training period.

**Decision:** Retain all products in the item catalog.

**Reasoning:**

Products without historical sales may still be valid catalog entries. Their absence from historical transactions does not establish that they are invalid or unnecessary.

Removing them would unnecessarily reduce the completeness of the product reference dataset.

### 6.6 Missing Store-Date Transaction Records

The sales dataset contains 118 distinct store-date combinations without corresponding records in the transactions dataset, distributed across eight dates.

One notable case occurred on January 3, 2016, when all 53 stores represented in the sales data had no matching transaction records.

**Decision:** Retain the sales records and do not invent or impute transaction counts.

**Reasoning:**

The absence of a transaction record does not establish that the transaction count was zero.

Missing transaction information is therefore treated as a data limitation rather than an automatically imputable value.

### 6.7 Duplicate Records and Key Validation

The logical sales key was defined as:

```text
date + store_nbr + item_nbr
```

Validation results:

| Validation | Result |
|---|---|
| Training duplicate keys | None found |
| Test duplicate keys | None found |
| Unique test keys | 3,370,464 |
| Training ID uniqueness | All IDs unique |

**Decision:** No duplicate sales records were removed.

**Reasoning:**

The key validation did not identify duplicated records requiring removal.

### 6.8 Referential Integrity

Relationships between the primary sales dataset and reference datasets were validated.

The validation confirmed that:

- Every sales product exists in the item catalog.
- Every sales store exists in the store catalog.
- No invalid store identifiers were found in the training dataset.
- No invalid product identifiers were found in the training dataset.
- Holiday/event locations were successfully matched to relevant store cities and states.

**Decision:** Retain the reference datasets and use the validated relationships during subsequent integration.

### 6.9 Holiday and Event Data

The original holiday/event dataset contains:

- 350 records.
- No missing values.
- No exact duplicate records.

However, multiple events can occur on the same date, and different events may apply to different stores depending on their geographic scope:

- National.
- Regional.
- Local.

A store-level event mapping was created, and multiple events applying to the same store-date combination were summarized into a single record.

The resulting summary contains 8,109 unique store-date combinations.

Joining this summary to the sales dataset preserved the original 125,497,040 sales records.

**Decision:** Preserve the original event records and use the summarized store-date table when integrating events with sales.

**Reasoning:**

Joining raw event records directly could multiply sales rows whenever multiple events apply to the same date and store.

Summarizing events at the store-date level prevents unintended row multiplication while retaining relevant event information.

Transferred holidays, transfer records, work days, and other event types were retained because they represent distinct calendar information.

### 6.10 Date Validation

The training dataset contains no missing dates.

| Attribute | Result |
|---|---|
| Minimum training date | January 1, 2013 |
| Maximum training date | August 15, 2017 |
| Missing training dates | None |
| Missing test dates | None |
| Missing test promotion indicators | None |

**Decision:** Retain the validated date values for subsequent time-based analysis.

---

## 7. Data Integration and Relationship Management

After the initial cleaning and validation stages, the auxiliary datasets were prepared for integration with the historical sales data.

The main integration dimensions included:

- Product metadata through "item_nbr".
- Store metadata through "store_nbr".
- Calendar information through "date".
- Transaction information through "date" and "store_nbr".
- Oil price information through "date".
- Holiday/event information through the applicable store and date.

### 7.1 Join Validation

Data integration was treated as a data quality concern, not merely a technical operation.

The following conditions were considered:

1. Reference identifiers must correspond to valid catalog entries.
2. Date fields must use compatible data types.
3. Store-date event mappings must be unique before joining.
4. Joins must not unintentionally multiply the original sales records.
5. Missing auxiliary information must remain distinguishable from actual zero values.

The holiday/event integration was specifically validated to ensure that the final join preserved the original sales row count.

### 7.2 Avoiding Unsupported Imputation

Missing auxiliary information was not automatically replaced with zero.

For example:

- Missing transactions were not interpreted as zero transactions.
- Missing promotion indicators were not interpreted as confirmed non-promotional observations.
- Missing oil dates were not treated as invalid sales dates.

This distinction is essential for maintaining the reliability of subsequent analytical conclusions.

---

## 8. Analytical Methodology

The EDA stage focused on understanding the historical behavior of sales through aggregation, segmentation, comparisons, and statistical summaries.

The primary analytical measure was total "unit_sales".

### 8.1 Sales Overview and Temporal Analysis

The objective was to understand how sales volume evolved over time and identify recurring temporal patterns.

The analysis included:

- Daily sales aggregation.
- Monthly and yearly comparisons.
- Average daily sales calculations.
- Monthly seasonality.
- Weekday sales patterns.
- Store coverage changes over time.

#### Monthly Average Daily Sales

To compare months with different numbers of days, average daily sales were calculated:

$$
AverageDailySales_m =
\frac{TotalSales_m}{NumberOfDays_m}
$$

This avoids comparing monthly totals without considering differences in month length.

#### Monthly Seasonality Index

Monthly seasonality was examined using normalized monthly sales indices.

The monthly index represents the average sales level for a particular month relative to the overall average daily sales baseline.

$$
MonthlyIndex_m =
\frac{AverageDailySales_m}
{OverallAverageDailySales}
\times 100
$$

An index above 100 indicates sales above the reference average, while an index below 100 indicates sales below it.

For seasonal comparisons, complete years from 2013 through 2016 were used to avoid introducing bias from the incomplete 2017 period.

#### Weekday Analysis

Daily sales were grouped by weekday and normalized against the overall daily average.

$$
WeekdayIndex_d =
\frac{AverageDailySales_d}
{OverallAverageDailySales}
\times 100
$$

This makes weekday patterns easier to compare independently of the absolute sales scale.

#### Partial-Year Comparisons

The historical dataset ends on August 15, 2017.

Therefore, annual comparisons involving 2017 were restricted to equivalent date ranges where appropriate.

For example, January 1 through August 15 was used when comparing 2017 with previous years.

Monthly averages were preferred over raw monthly totals when the number of observed days differed.

#### Store Coverage Considerations

Changes in the number of observed stores can influence aggregated sales trends.

Therefore, store coverage was examined alongside sales performance.

A fixed-store comparison was also used to investigate long-term changes while holding the selected store population constant.

### 8.2 Store Performance Analysis

The objective was to evaluate differences in store-level sales volume, identify high- and low-performing stores, and understand the distribution of sales across the store network.

The analysis included:

- Total unit sales per store.
- Store contribution to overall sales.
- Store rankings.
- Annual store growth.
- Store-type comparisons.
- Within-type variability.
- Sales concentration.

#### Store Sales Contribution

Each store's contribution to total unit sales was calculated as:

$$
StoreContribution_i =
\frac{StoreSales_i}{TotalSales}
\times 100
$$

This metric measures the percentage of total sales volume generated by each store.

#### Annual Store Growth

Store growth was calculated using equivalent date ranges when comparing different years.

$$
Growth_i =
\frac{Sales_{i,t}-Sales_{i,t-1}}
{Sales_{i,t-1}}
\times 100
$$

Comparisons were restricted to stores with valid baseline sales.

This prevents stores without an earlier observation from being treated as having a valid growth rate.

#### Outlier Investigation

The Interquartile Range method was used to identify unusually high or low store growth values.

$$
IQR = Q3-Q1
$$

The conventional lower and upper boundaries were:

$$
LowerBound = Q1-1.5(IQR)
$$

$$
UpperBound = Q3+1.5(IQR)
$$

Values outside these boundaries were treated as potential statistical outliers requiring further investigation, rather than automatically classified as errors.

#### Store-Type Performance

Stores were grouped by their store type.

The analysis considered:

- Total sales by type.
- Number of stores within each type.
- Average sales per store.
- Median sales per store.
- Minimum and maximum sales.
- Standard deviation.
- Coefficient of variation.

The coefficient of variation was calculated as:

$$
CV =
\frac{StandardDeviation}{Mean}
\times 100
$$

This allows relative variability to be compared across store types with different average sales levels.

#### Sales Concentration

Sales concentration was examined by ranking stores and calculating cumulative contribution.

The objective was to determine whether a relatively small number of stores generated a disproportionately large share of total unit sales.

### 8.3 Product Performance Analysis

The objective was to understand how sales volume was distributed across products and product families.

The analysis included:

- Product-level sales aggregation.
- Product ranking.
- Product-family contribution.
- Sales concentration.
- Distribution differences between product families.
- Product-store observations.

#### Product-Family Contribution

The contribution of each product family was calculated as:

$$
FamilyContribution_f =
\frac{FamilySales_f}{TotalSales}
\times 100
$$

This identifies the product families contributing the largest shares of total unit sales.

#### Product Concentration

Product concentration was examined by ranking products within their respective families and calculating the cumulative share of sales generated by the highest-volume products.

The analysis included the contribution of the top 10% of products within selected families.

This helps distinguish between:

- Families with relatively distributed demand.
- Families where sales are concentrated among a small number of products.

Concentration should be interpreted in the context of family size. A small family may naturally exhibit high concentration because it contains fewer products.

#### Product Availability and Historical Observations

Products without historical sales were retained in the reference catalog.

The absence of observed sales does not necessarily indicate that a product was discontinued or unavailable.

Similarly, the first observed sale date should not automatically be interpreted as the official product launch date.

#### Product-Store Analysis

Selected product-store combinations were examined to investigate differences in observed sales across store attributes.

Relevant attributes included:

- City.
- State.
- Store type.
- Cluster.

Only observed combinations were used in the corresponding analysis.

Missing product-store combinations were not automatically interpreted as zero demand or stockouts because the available data does not establish the reason for their absence.

### 8.4 Geographic and Store-Type Analysis

The project scope included geographic comparisons using store metadata.

The intended analytical dimensions included:

- City.
- State.
- Store type.
- Cluster.

These attributes provide a basis for examining differences in sales volume across locations and store groups.

Geographic comparisons must account for the number of stores represented in each location.

A location with higher total sales may simply contain more stores. Therefore, total sales should be interpreted alongside store counts and, where appropriate, average sales per store.

Geographic performance conclusions should only be reported when supported by the corresponding aggregated results.

### 8.5 Promotion Analysis

The objective was to investigate how observed unit sales differed between promotional and non-promotional records.

The analysis was conducted using the "onpromotion" indicator.

Because promotion information was unavailable for all of 2013 and part of 2014, the comparisons focused on periods with recorded promotion indicators.

Missing promotion values were excluded from comparisons requiring a confirmed promotional or non-promotional status.

#### Average Sales Comparison

Average unit sales per observed record were calculated for promotional and non-promotional observations.

$$
PromotionDifference =
\frac{PromoAverage-NonPromoAverage}
{NonPromoAverage}
\times 100
$$

This provides a relative comparison between the two observed groups.

#### Product-Level Comparison

To reduce unreliable comparisons based on very small observation counts, product-level comparisons required a minimum of 30 observations in each condition.

The analysis measured:

- Number of eligible products.
- Products with higher average sales under promotion.
- Products with lower average sales under promotion.
- Median relative difference.

#### Store-Product Comparison

Promotion behavior was also investigated at the store-product level.

This provides a more granular comparison by considering how the same product behaves within individual stores.

A minimum of 30 observations in each condition was required.

#### Store-Product-Weekday Comparison

A further comparison considered store, product, and weekday together.

This helps control for some recurring differences in weekday sales patterns.

A minimum of 10 observations in each condition was required for this analysis.

#### Interpretation and Limitations

These comparisons are observational.

Higher average sales during promotional periods do not establish that promotions caused the increase.

Potential confounding factors include:

- Product selection for promotions.
- Store-level differences.
- Seasonal demand.
- Product availability.
- Timing and duration of promotions.
- Differences in the eligible observation populations.

Therefore, the results are interpreted as observed associations rather than causal estimates.

Year-to-year comparisons of eligible products or store-product combinations must also account for differences in the populations being analyzed.

### 8.6 Transactions and Sales Relationships

The project scope included investigating the relationship between daily transaction counts and unit sales.

The intended analysis considers:

- Transaction activity across stores.
- Changes in transaction volume over time.
- Comparisons between transaction counts and sales volume.
- Average units sold per transaction where the available data supports a valid calculation.

Missing store-date transaction records are preserved as missing observations.

They are not automatically replaced with zero, since the absence of a record does not establish that no transactions occurred.

Any transaction-related interpretation must therefore consider data coverage and the availability of corresponding store-date records.

### 8.7 Holiday and Event Analysis

Holiday and event data provides calendar context that may help explain changes in sales activity.

The analytical scope includes:

- Sales comparisons on dates with and without relevant events.
- Differences between event categories.
- National, regional, and local event coverage.
- Transferred holidays.
- Work days and other calendar events.

The event data is summarized at the store-date level before integration.

This ensures that multiple events affecting the same store on the same date do not multiply the underlying sales observations.

Observed differences around holidays and events are treated as associations, not proof of causal effects.

### 8.8 Oil Price Analysis

Daily oil prices were included as an external contextual variable.

The analysis scope includes:

- Comparing broad oil price trends with sales activity.
- Investigating potential temporal associations.
- Considering missing oil dates.
- Accounting for previously imputed oil price values.

Oil prices are not assumed to directly explain changes in grocery sales.

Any relationship must be interpreted cautiously because multiple external and internal factors can influence sales at the same time.

Correlation or temporal co-movement does not establish causation.

---

## 9. Sensitivity Analysis and Validation

Sensitivity analysis was used to investigate whether specific data quality decisions materially changed the analytical results.

### 9.1 Negative Sales Sensitivity Analysis

The effect of retaining negative sales values was investigated by comparing two scenarios:

1. All historical sales records included.
2. Historical sales records with negative "unit_sales" excluded.

The comparison examined:

- Total sales differences.
- Store ranking changes.
- The relative impact of negative values.

Results:

| Metric | Result |
|---|---|
| Negative records | 7,795 |
| Approximate difference | 143,580.83 units |
| Difference as a share of net sales | 0.0134% |
| Store ranking changes | 0 |
| Maximum rank shift | 0 |

The results indicate that retaining negative values had a very limited impact on overall store rankings.

Therefore, the primary analysis retained negative sales values rather than removing them.

### 9.2 Join Validation

Data integration was checked to ensure that joins did not unintentionally increase the number of sales records.

The holiday/event integration was validated specifically, with the summarized event table preserving the original 125,497,040 sales rows.

### 9.3 Key Validation

The training and test datasets were checked for uniqueness using their logical sales keys.

No duplicate sales keys were identified.

This supports the reliability of subsequent aggregations at the date-store-product level.

---

## 10. Power BI Reporting Methodology

Power BI was used to translate the analytical results into an interactive business report.

The report is organized into six analytical pages.

### 10.1 Executive Overview

Provides a high-level summary of the historical sales dataset and its most important performance indicators.

Its purpose is to give stakeholders an overview before exploring more detailed analytical dimensions.

### 10.2 Time Analysis

Focuses on temporal patterns, including:

- Sales trends.
- Monthly patterns.
- Weekday behavior.
- Long-term changes.
- Comparisons across equivalent periods.

### 10.3 Store Performance

Examines differences between stores using:

- Total unit sales.
- Store contribution.
- Store rankings.
- Growth comparisons.
- Store-type performance.
- Sales distribution and variability.

### 10.4 Product Performance

Focuses on product-level and family-level performance, including:

- Product rankings.
- Family contribution.
- Sales concentration.
- Differences between product groups.

### 10.5 Geographic Analysis

Provides a geographic perspective using store attributes such as city, state, and cluster.

Its purpose is to support location-based comparisons and identify differences across geographic groups.

### 10.6 External Factors & Drivers

Explores potential relationships between sales and additional contextual variables, including:

- Promotions.
- Holidays and events.
- Oil prices.
- Transactions.

These relationships are presented with appropriate attention to data limitations and the distinction between association and causation.

---

## 11. Analytical Limitations

Several limitations must be considered when interpreting the results.

### 11.1 Sales Volume Versus Financial Performance

The primary measure is "unit_sales".

The analysis does not directly measure revenue, gross margin, operating profit, or profitability.

Higher unit sales do not necessarily indicate higher financial performance.

### 11.2 Incomplete 2017 Coverage

The historical dataset ends on August 15, 2017.

Consequently, 2017 is a partial year and cannot be compared with complete years using unadjusted annual totals.

Equivalent date ranges or daily averages are required for meaningful comparisons.

### 11.3 Missing Promotion Information

Promotion indicators are unavailable for all of 2013 and part of 2014.

Promotion-related analysis is therefore restricted to observations with recorded promotion information.

### 11.4 Negative Sales Values

Negative unit sales were retained because they may represent valid business adjustments.

Although their overall impact on store rankings was negligible in the sensitivity analysis, their business meaning cannot be confirmed from the sales field alone.

### 11.5 Imputed Oil Prices

Missing oil prices were filled using forward fill and backfill.

Imputed values are estimates and may affect the interpretation of relationships involving oil prices.

### 11.6 Missing Transaction Records

Some store-date combinations do not have corresponding transaction records.

These missing observations were not treated as zero transactions.

Consequently, transaction-related analysis may have incomplete coverage.

### 11.7 Store Coverage Changes

The number of stores represented in the dataset changes over time.

Aggregated sales growth may therefore reflect both changes in sales activity and changes in store coverage.

Fixed-store comparisons help address this issue but do not isolate every possible factor affecting sales.

### 11.8 Observational Promotion Analysis

Promotional and non-promotional sales comparisons do not establish causal effects.

Other factors may influence the observed differences.

### 11.9 Product Availability

Missing product-store combinations do not establish that products were unavailable, discontinued, or out of stock.

The available data only confirms whether a combination was observed in the historical records.

### 11.10 Geographic Interpretation

Geographic comparisons should account for the number of stores in each location.

Total sales alone may not represent the relative performance of individual stores within a geographic area.

---

## 12. Reproducibility and Analytical Principles

The project follows several principles to maintain analytical consistency and reliability.

### 12.1 Preserve Valid Business Information

Potentially meaningful records are not removed automatically.

Negative sales, missing promotion indicators, and incomplete transaction records are treated according to their business context.

### 12.2 Avoid Unsupported Assumptions

Missing values are not automatically interpreted as zero or false.

Imputed values are documented, and unavailable information is distinguished from confirmed business observations.

### 12.3 Validate Data Integration

Relationships between sales records and reference datasets are checked before analytical use.

Joins are evaluated for potential row multiplication and identifier mismatches.

### 12.4 Use Appropriate Comparison Periods

Partial periods are not compared directly with complete periods without adjustment.

Equivalent date ranges and normalized measures are used where appropriate.

### 12.5 Distinguish Association from Causation

Observed relationships between sales and external factors are interpreted cautiously.

The analysis identifies patterns and associations without claiming causal effects that the available data cannot establish.

### 12.6 Document Analytical Decisions

Important data quality decisions, assumptions, and limitations are documented so that the reasoning behind the results remains transparent.

---

## 13. Final Methodological Summary

The Corporación Favorita Grocery Sales Analysis project follows a structured analytical workflow:

1. Data Ingestion: Load and inspect the primary and auxiliary datasets.
2. Initial EDA: Examine schemas, missing values, categorical integrity, distributions, and duplicate records.
3. Data Cleaning: Standardize dates, text attributes, numerical types, and relevant indicators.
4. Data Quality Decisions: Evaluate missing values, negative sales, incomplete auxiliary records, and other potential data issues.
5. Data Integration: Validate reference relationships and prepare auxiliary information for analysis without multiplying sales records.
6. Analytical Investigation: Examine temporal patterns, store performance, product contribution, geographic differences, and potential external relationships.
7. Sensitivity Analysis: Evaluate the impact of selected data quality decisions on analytical results.
8. Business Reporting: Present the findings through an interactive Power BI report.

The overall methodology prioritizes data integrity, transparent analytical decisions, appropriate statistical comparisons, and careful interpretation of business patterns.

**Final Objective:** Transform a large and complex historical sales dataset into a structured analytical foundation that supports meaningful business understanding and evidence-based decision-making.
