# Retail Sales & Customer Intelligence Using PostgreSQL

An end-to-end SQL analytics project that turns two years of raw order data into clear business
decisions for **Duka Smart Electronics**, a growing Kenyan electronics retailer operating in ten towns.

## Project Overview

Duka Smart Electronics sells mobile phones, laptops, TVs, accessories and home appliances, and has
recorded every order since January 2025. Its data had never been analysed. As a Junior Data Analyst,
I designed the database, wrote **46 SQL queries**, and translated the results into recommendations
for management.

### Business questions addressed
- Who are the company's most valuable customers?
- Which products and categories generate revenue, and which generate profit?
- How are sales trending month to month?
- Which customers have stopped buying?
- 
## Key Findings

| # | Finding | Business implication |
|---|---|---|
| 1 | Five products bring in **~52% of completed revenue**. The iPhone 13 alone brought in KSh 3.77M. | Stock-outs or supplier price rises on these items would hurt revenue sharply. |
| 2 | Mobile Phones is the most profitable category, but has a **10.5% margin**. | Small increases in discounting would wipe out most of the category's profit. |
| 3 | All five highest-spending customers are in the **Retail** segment. | Segment labels don't reflect actual customer value. |
| 4 | The top 10 customers bring in **29.75% of revenue**. | The business depends heavily on a small number of relationships. |
| 5 | January 2026 was the **lowest-revenue month**, 26% below January 2025. | The slow period after the holidays needs planned promotions. |

Full analysis and recommendations: **[insights.md](insights.md)**

## Dataset

| Table | Rows | Description |
|---|---|---|
| `customers` | 150 | Customer name, gender, city, registration date and segment (Retail, Corporate, VIP) |
| `orders` | 800 | Order date, product, category, quantity, unit price, cost price, discount, payment method and status |

**Relationship:** one customer → many orders, linked by `customer_id` (foreign key).

customers (1) ──────────< orders (many)
```

### Business rules

| Measure | Definition |
|---|---|
| Revenue | `quantity * unit_price * (1 - discount / 100)` |
| Cost | `quantity * cost_price` |
| Profit | `revenue - cost` |
| Profit margin | `profit / NULLIF(revenue, 0) * 100` |

- Financial metrics count **Completed** orders only. Order counts include every order, whatever its status.
- "Today" is the latest `order_date` in the dataset, so results can be reproduced.
- Months are grouped by year and month with `DATE_TRUNC('month', order_date)`.

---

## Getting Started

### Prerequisites
- PostgreSQL 14 or later
- A SQL client: DBeaver, pgAdmin or `psql`

### Setup

```bash
# 1. Create the database
psql -U postgres -d postgres -c "CREATE DATABASE retail_analytics;"

# 2. Create the tables
psql -U postgres -d retail_analytics -f schema.sql

# 3. Load the data (insert scripts are provided on the project page)
psql -U postgres -d retail_analytics -f 03_insert_customers.sql
psql -U postgres -d retail_analytics -f 04_insert_orders.sql

# 4. Verify the load
psql -U postgres -d retail_analytics -f 05_verify_data.sql
```

**Expected verification:** 150 customers, 800 orders, 0 orphaned orders, and order dates from January 2025 to September 2026.

---

## Running the Analysis

Open `queries.sql` while connected to `retail_analytics`, and run each question one at a time or the whole file:

```bash
psql -U postgres -d retail_analytics -f queries.sql
```

> **Note:** run Q43 before Q46. Q43 creates the `monthly_sales_summary` view that Q46 uses.

| Section | Questions | Techniques |
|---|---|---|
| A. Fundamentals | Q1–Q10 | `SELECT`, `WHERE`, `DISTINCT`, `ORDER BY`, `LIKE`, `BETWEEN` |
| B. Aggregation | Q11–Q20 | `COUNT`, `SUM`, `AVG`, `GROUP BY`, `HAVING`, `FILTER` |
| C. Joins | Q21–Q28 | `INNER JOIN`, `LEFT JOIN`, customer-level analysis |
| D. Subqueries & CTEs | Q29–Q34 | Nested aggregation, `WITH` clauses, date arithmetic |
| E. Window Functions | Q35–Q42 | `RANK`, `DENSE_RANK`, `ROW_NUMBER`, `LAG`, `NTILE`, running totals |
| F. Business Intelligence | Q43–Q46 | Views, PL/pgSQL functions, `CASE` segmentation, executive reporting |

### Reusable database objects

```sql
-- Monthly orders, revenue, cost and profit
SELECT * FROM monthly_sales_summary ORDER BY month;

-- Completed-order revenue for any customer (returns 0 for unknown IDs)
SELECT customer_revenue(33);
```

---

## Repository Structure

```
retail-sales-sql/
├── README.md        # Project documentation
├── schema.sql       # Table definitions, keys and constraints
├── queries.sql      # 46 analytical queries (Q1–Q46)
├── insights.md      # Business insights and recommendations
└── screenshots/     # Verification output and key query results
```

---

## Skills Demonstrated

Relational database design · Data validation · Multi-table joins · Aggregation and grouping ·
CTEs and subqueries · Window functions · Views and stored functions · Customer segmentation ·
Translating data into business recommendations

---

## Author

**Hope Gatugi**
