# E-Commerce Sales EDA: Database & Business Metrics Exploration
### Exploratory Data Analysis Using SQL

---

## 📋 Abstract

Before you can extract insight from data, you need to understand
what you're working with.

This project is the foundation of a two-part SQL analytics series
built on a real-world e-commerce dataset. Using a structured,
systematic approach to Exploratory Data Analysis, it investigates
the database architecture, dimensions, time boundaries, and core
business metrics of an e-commerce operation — laying the groundwork
for the deeper analytical work that follows in Part 2.

The goal here is simple but critical: know your data before you
draw conclusions from it.

---

## Section 1 — Introduction & Problem Statement

### 1.1 Background

In any data analytics project, the quality of your conclusions
depends entirely on how well you understand your data. Jumping
straight into advanced analysis without first exploring the
dataset — its structure, its range, its quirks — is one of the
most common mistakes analysts make.

This project takes the opposite approach.

Before any business question gets answered, the data gets
interrogated. What tables exist? What are their relationships?
What time period does the data cover? What are the key business
metrics at a high level? Only after answering these questions
can meaningful analysis begin.

### 1.2 Problem Statement

Working with an e-commerce dataset containing customer, product,
and sales records, the core challenge is establishing a reliable
analytical foundation by answering:

- What is the structure and schema of the database?
- What are the unique dimensions — countries, categories,
  products — that define this business?
- What time period does the data cover, and what is the
  demographic range of the customer base?
- What are the headline business metrics — total sales, orders,
  customers, and products?
- How does performance distribute across countries, genders,
  and product categories?
- Which products and customers are the top and bottom performers?

### 1.3 Research Objectives

This analysis is organized around six progressive exploration
questions:

1. What does the database structure look like?
2. What are the unique dimensions of the business?
3. What is the temporal and demographic boundary of the data?
4. What are the key business metrics at a glance?
5. How do metrics distribute across categories and geographies?
6. Who and what are the top and bottom performers?

---

## Section 2 — Methodology & Tools

### 2.1 Dataset

The dataset follows a **Gold Layer architecture** — pre-cleaned,
business-ready tables optimized for analysis:

| File | Description |
|---|---|
| `gold.dim_customers.csv` | Customer demographics — names, countries, birthdates |
| `gold.dim_products.csv` | Product catalog — categories, subcategories, costs |
| `gold.fact_sales.csv` | Sales transactions — orders, quantities, revenue |

> The Gold Layer naming convention signals that these tables have
> already passed through data cleaning and transformation stages —
> making them reliable and analysis-ready. However, as this EDA
> reveals, even Gold Layer data can contain quality issues worth
> flagging and documenting.

### 2.2 Analytical Approach

| Layer | Focus |
|---|---|
| Database Exploration | Schema, tables, column metadata |
| Dimensions Exploration | Unique values across key dimensions |
| Date Range Exploration | Temporal boundaries and customer age range |
| Measures Exploration | Headline KPIs and business metrics |
| Magnitude Analysis | Distribution across categories and geographies |
| Ranking Analysis | Top and bottom performers by revenue and orders |

### 2.3 Project Scripts

| Script | Purpose |
|---|---|
| `01_database_exploration.sql` | Inspect database structure and column metadata |
| `02_dimensions_exploration.sql` | Explore unique countries, categories, and products |
| `03_date_range_exploration.sql` | Determine data time range and customer age boundaries |
| `04_measures_exploration.sql` | Calculate headline business KPIs |
| `05_magnitude_analysis.sql` | Aggregate metrics by country, gender, and category |
| `06_ranking_analysis.sql` | Rank top and bottom products and customers |

### 2.4 Tools Used

| Tool | Purpose |
|---|---|
| SQL Server (T-SQL) | Core query language |
| SSMS | Query execution and result inspection |
| INFORMATION_SCHEMA | Database metadata exploration |
| Window Functions | Flexible ranking and comparative analysis |
| Git & GitHub | Version control and portfolio sharing |

---

## Section 3 — Analysis & Results

### 3.1 Database Exploration
📄 *Script: `01_database_exploration.sql`*

The first step was understanding what the database contains —
inspecting all available tables and their column structures
using system metadata views.

```sql
-- Database Tables
SELECT
    TABLE_CATALOG,
    TABLE_SCHEMA,
    TABLE_NAME,
    TABLE_TYPE
FROM INFORMATION_SCHEMA.TABLES;

-- Column Structure for dim_customers
SELECT
    COLUMN_NAME,
    DATA_TYPE,
    IS_NULLABLE,
    CHARACTER_MAXIMUM_LENGTH
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME = 'dim_customers';
```

#### Result:
![Database Exploration Result](/images/01_database_exploration_result.PNG)

**What this revealed:**
The database contains **4 objects** in the `gold` schema —
three core tables (`dim_customers`, `dim_products`,
`fact_sales`) and one VIEW (`report_products`), confirming
a clean dimensional model architecture. The `dim_customers`
table structure reveals key fields including `customer_key`,
`customer_id`, `customer_number`, `first_name`, `last_name`,
and `country` — all essential dimensions for customer-level
analysis.

---

### 3.2 Dimensions Exploration
📄 *Script: `02_dimensions_exploration.sql`*

With the structure confirmed, the next step was understanding
the unique values within key dimensions — the building blocks
of any segmentation or filtering analysis.

```sql
-- Unique Countries
SELECT DISTINCT country
FROM gold.dim_customers
ORDER BY country;

-- Unique Categories, Subcategories and Products
SELECT DISTINCT
    category,
    subcategory,
    product_name
FROM gold.dim_products
ORDER BY category, subcategory, product_name;
```

#### Result:
![Dimensions Exploration Result](/images/02_dimensions_exploration_result.PNG)

**What this revealed:**
The customer base spans **7 geographic entries** — Australia,
Canada, France, Germany, United Kingdom, and United States —
with one entry recorded as `n/a`, flagged for data quality
review.

> ⚠️ **Data Quality Observation:** The `category` and
> `subcategory` columns returned **NULL values** for a
> subset of product entries. Upon investigation, this appears
> to be either incomplete data entry at the source or products
> that were not yet assigned to a category at the time of
> recording. This was documented and flagged for cleaning
> before any product category-level analysis is conducted.
> This is a normal and expected finding in real-world datasets
> — and identifying it early is precisely the purpose of EDA.

---

### 3.3 Date Range Exploration
📄 *Script: `03_date_range_exploration.sql`*

Understanding the temporal boundaries of the data is essential
before any trend or time-series analysis can be trusted.

```sql
-- First and Last Order Date
SELECT
    MIN(order_date) AS first_order_date,
    MAX(order_date) AS last_order_date,
    DATEDIFF(MONTH, MIN(order_date), MAX(order_date))
        AS order_range_months
FROM gold.fact_sales;

-- Youngest and Oldest Customer
SELECT
    MIN(birthdate) AS oldest_birthdate,
    DATEDIFF(YEAR, MIN(birthdate), GETDATE()) AS oldest_age,
    MAX(birthdate) AS youngest_birthdate,
    DATEDIFF(YEAR, MAX(birthdate), GETDATE()) AS youngest_age
FROM gold.dim_customers;
```

**Result:**
![Date Range Exploration Result](/images/03_date_range_result.PNG)

**What this revealed:**
The dataset covers **37 months of transaction history**,
spanning from **December 29, 2010** to **January 28, 2014**
— a solid analytical window for trend and seasonality analysis.

The customer age analysis reveals a range from **40 years
old** (youngest, born June 25, 1986) to **110 years old**
(oldest, born February 10, 1916).

> ⚠️ **Data Quality Observation:** The oldest customer age
> of **110 years** is a likely data entry anomaly. A birthdate
> of 1916 in an active e-commerce customer dataset warrants
> investigation — this record should be reviewed and either
> corrected or excluded from any age-based demographic analysis.

---

### 3.4 Measures Exploration — Key Business Metrics
📄 *Script: `04_measures_exploration.sql`*

With the structure and boundaries understood, the headline
business metrics were calculated to establish a single source
of truth for the e-commerce operation's overall performance.

```sql
SELECT 'Total Sales' AS measure_name,
    SUM(sales_amount) AS measure_value FROM gold.fact_sales
UNION ALL
SELECT 'Total Quantity', SUM(quantity) FROM gold.fact_sales
UNION ALL
SELECT 'Average Price', AVG(price) FROM gold.fact_sales
UNION ALL
SELECT 'Total Orders',
    COUNT(DISTINCT order_number) FROM gold.fact_sales
UNION ALL
SELECT 'Total Products',
    COUNT(DISTINCT product_name) FROM gold.dim_products
UNION ALL
SELECT 'Total Customers',
    COUNT(customer_key) FROM gold.dim_customers;
```

#### Result:
![Key Business Metrics Result](/images/04_measures_result.PNG)

**What this revealed:**
A single consolidated view of the entire business:

| Metric | Value |
|---|---|
| Total Sales | **$29,356,250** |
| Total Quantity Sold | **60,423 units** |
| Average Price | **$486** |
| Total Orders | **27,659** |
| Total Products | **295** |
| Total Customers | **18,484** |

These headline numbers serve as the baseline benchmark
against which all deeper analysis is measured — any
segment, category, or ranking finding must be interpreted
relative to these totals.

---

### 3.5 Magnitude Analysis
📄 *Script: `05_magnitude_analysis.sql`*

With headline metrics established, the analysis shifted to
understanding how performance distributes across key
dimensions — countries, genders, and product categories.

```sql
SELECT
    p.category,
    SUM(f.sales_amount) AS total_revenue
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p
    ON p.product_key = f.product_key
GROUP BY p.category
ORDER BY total_revenue DESC;
```

#### Result:
![Magnitude Analysis Result](/images/05_magnitude_result.PNG)

**What this revealed:**
The revenue distribution across product categories is
dramatically skewed:

| Category | Total Revenue |
|---|---|
| Bikes | **$28,316,272** |
| Clothing | **$339,716** |
| Accessories | **$700,262** |

**Bikes account for approximately 97% of all revenue** —
an extraordinary concentration that has major implications
for inventory strategy, marketing investment, and business
risk management.

The top customers by revenue are tightly clustered —
the highest spender (**Nichole Nara, $13,294**) is
separated from the 7th-ranked customer by less than
$100 — suggesting a relatively even distribution among
top buyers rather than a single dominant customer.

---

### 3.6 Ranking Analysis
📄 *Script: `06_ranking_analysis.sql`*

The final exploration layer identified the specific products
at the top and bottom of performance — using both simple
`TOP` queries and advanced Window Functions for flexible,
reusable ranking logic.

```sql
-- Top 5 Products by Revenue
SELECT * FROM (
    SELECT
        p.product_name,
        SUM(f.sales_amount) AS total_revenue,
        RANK() OVER (ORDER BY SUM(f.sales_amount) DESC)
            AS rank_products
    FROM gold.fact_sales f
    LEFT JOIN gold.dim_products p
        ON p.product_key = f.product_key
    GROUP BY p.product_name
) AS ranked_products
WHERE rank_products <= 5;

-- Bottom 5 Products by Revenue
SELECT TOP 5
    p.product_name,
    SUM(f.sales_amount) AS total_revenue
FROM gold.fact_sales f
LEFT JOIN gold.dim_products p
    ON p.product_key = f.product_key
GROUP BY p.product_name
ORDER BY total_revenue;
```

**Result:**
![Ranking Analysis Result](/images/06_ranking_result.PNG)

**What this revealed:**

**Top 5 Products by Revenue:**

| Product | Total Revenue |
|---|---|
| Mountain-200 Black-46 | **$1,373,454** |
| Mountain-200 Black-42 | **$1,363,128** |
| Mountain-200 Silver-38 | **$1,339,394** |
| Mountain-200 Silver-46 | **$1,301,029** |
| Mountain-200 Black-38 | **$1,294,854** |

**Bottom 5 Products by Revenue:**

| Product | Total Revenue |
|---|---|
| Racing Socks-L | **$2,430** |
| Racing Socks-M | **$2,682** |
| Patch Kit/8 Patches | **$6,382** |
| Bike Wash-Dissolv... | **$7,272** |
| Touring Tire Tube | **$7,440** |

The top 5 products are **all Mountain-200 variants** —
confirming that a single product line drives an outsized
share of revenue. The bottom 5 are all low-cost accessories,
generating less than $7,500 each across the entire
37-month dataset.

---

## Section 4 — Key Findings

- The database contains **4 objects** in the Gold schema —
  3 tables and 1 VIEW — organized in a clean dimensional
  model architecture
- The customer base spans **6 countries** plus one
  unclassified `n/a` entry flagged for data quality review
- **Data quality issues were identified** — NULL category
  values in the products dimension and a likely erroneous
  customer birthdate of 1916 — both documented and flagged
  for remediation
- The dataset covers **37 months** of transaction history
  from December 2010 to January 2014
- Total revenue across the period reached **$29,356,250**
  across **27,659 orders** from **18,484 customers**
- **Bikes account for ~97% of all revenue ($28.3M)** —
  an extraordinary concentration that defines the business's
  commercial profile
- The **Mountain-200 product line dominates** top performance,
  with all 5 highest-revenue products belonging to this
  single product family
- Bottom performers are exclusively **low-cost accessories**
  generating under $7,500 each across the full 37 months

---

## Section 5 — Recommendations

**1. Investigate and Resolve Data Quality Issues**
Two data quality flags were raised during this EDA — NULL
product categories and a likely erroneous customer birthdate.
These should be investigated at the source system level
before any production reporting is built on this dataset.
Clean data is the foundation of trustworthy analysis.

**2. Reduce Concentration Risk in the Bike Category**
With 97% of revenue coming from Bikes, this business is
heavily exposed to any disruption in that category —
supply chain issues, demand shifts, or competitive pressure
could be catastrophic. A deliberate strategy to grow
Accessories and Clothing revenue would reduce this risk.

**3. Invest Heavily in the Mountain-200 Product Line**
The top 5 revenue-generating products are all Mountain-200
variants. This product line deserves priority treatment —
in stock levels, in marketing, and in supplier negotiations.
Running out of Mountain-200 stock is not an inventory
problem, it's a revenue problem.

**4. Review and Discontinue Bottom-Performing Accessories**
Products generating under $7,500 over 37 months —
like Racing Socks and Patch Kits — are consuming catalog
space and operational overhead for minimal return. Each
should be evaluated for discontinuation, repricing,
or bundling with higher-value products.

**5. Investigate the n/a Customer Country Entry**
A customer country recorded as `n/a` is a data integrity
issue. Understanding whether this represents a specific
geographic region, a system default, or a data entry
error will determine whether these customers can be
properly segmented in future geographic analysis.

---

## Section 6 — Conclusion

Good analysis starts with good questions. And good questions
start with understanding your data.

This project deliberately slows down before speeding up —
taking the time to inspect the database structure, understand
the dimensions, establish the time boundaries, and calculate
the headline metrics before drawing any conclusions.

What emerged was more than just numbers. The EDA surfaced
two data quality issues that would have silently corrupted
downstream analysis if left undetected. It revealed a
business almost entirely dependent on a single product
category and a single product line. And it established
the baseline metrics — $29.4M in revenue, 27,659 orders,
18,484 customers — against which all future performance
will be measured.

This is what EDA is for. Not just to describe the data —
but to understand it well enough to trust it.

---

## 📁 Project Structure
EDA_SQL_Project/
├── datasets/
│   ├── gold.dim_customers.csv
│   ├── gold.dim_products.csv
│   └── gold.fact_sales.csv
├── scripts/
│   ├── 01_database_exploration.sql
│   ├── 02_dimensions_exploration.sql
│   ├── 03_date_range_exploration.sql
│   ├── 04_measures_exploration.sql
│   ├── 05_magnitude_analysis.sql
│   └── 06_ranking_analysis.sql
├── images/
└── README.md

> 🔗 This is **Part 1** of a two-part SQL analytics series.
> For advanced analytics including segmentation, performance
> analysis, and business reporting, see
> [Part 2 — Advanced Sales Analytics](../Advanced_SQL_Analytics./)