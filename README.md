# Retail Sales & Customer Intelligence Using PostgreSQL

An end-to-end SQL analytics project that turns two years of raw order data into clear business
decisions for **Duka Smart Electronics**, a growing Kenyan electronics retailer operating in ten towns.

---

## Project Overview

Duka Smart Electronics sells mobile phones, laptops, TVs, accessories and home appliances in ten Kenyan
towns, including Nairobi, Mombasa, Meru and Machakos. It has recorded every order since January 2025,
but its data had never been analysed.

As a Junior Data Analyst, I:

1. **Designed and loaded the database**, with primary keys, a foreign key and data-quality constraints.
2. **Wrote 46 SQL queries** that progress from basic filtering to window functions, views and PL/pgSQL functions.
3. **Turned the results into business insights** that management can act on.

### Business questions addressed

| Question | Where it is answered |
|---|---|
| Who are the company's most valuable customers? | Q22, Q24, Q35, Q41, Q45 |
| Which products and categories bring in revenue, and which bring in profit? | Q16, Q33, Q37, Q42, Q46 |
| How are sales trending month to month? | Q31, Q38, Q39, Q40, Q43 |
| Which customers have stopped buying, or never started? | Q23, Q34 |

---

## Key Findings

| # | Finding | Business implication |
|---|---|---|
| 1 | Five products bring in **~52% of completed revenue**. The iPhone 13 alone brought in KSh 3.77M. | A stock-out or supplier price rise on these items would hurt revenue sharply. |
| 2 | Mobile Phones is the most profitable category, at KSh 728,761, but has a **10.5% margin**. | Small increases in discounting would wipe out most of the category's profit. |
| 3 | All five highest-spending customers are in the **Retail** segment, not VIP. | Segment labels don't reflect actual customer value. |
| 4 | The top 10 customers (under 8% of buyers) bring in **29.75% of revenue**. | The business depends heavily on a small number of relationships. |
| 5 | January 2026 was the **lowest-revenue month** (KSh 322,433), 26% below January 2025. | The slow period after the holidays needs planned promotions. |

Full analysis and recommendations: **[insights.md](insights.md)**

---

## Dataset

### Overview

| Table | Rows | Description |
|---|---|---|
| `customers` | 150 | One row per registered customer |
| `orders` | 800 | One row per order, January 2025 – September 2026 |

### Entity relationship

```
customers (1) ──────────< orders (many)
customer_id  (PK)         order_id     (PK)
                          customer_id  (FK → customers.customer_id)
```

One customer can place many orders, and every order belongs to exactly one customer. Some customers
have registered but never ordered, which is why a `LEFT JOIN` is needed to find them.

### `customers`

| Column | Type | Description |
|---|---|---|
| `customer_id` | INTEGER, PK | Unique customer number |
| `customer_name` | VARCHAR(100), NOT NULL | Full name |
| `gender` | VARCHAR(10) | `Male` / `Female` |
| `city` | VARCHAR(50) | One of ten Kenyan towns |
| `registration_date` | DATE | Date the customer joined |
| `customer_segment` | VARCHAR(20) | `Retail`, `Corporate` or `VIP` |

### `orders`

| Column | Type | Description |
|---|---|---|
| `order_id` | INTEGER, PK | Unique order number |
| `customer_id` | INTEGER, FK | Customer who placed the order |
| `order_date` | DATE, NOT NULL | Date of the order |
| `product_name` | VARCHAR(100) | Product purchased |
| `product_category` | VARCHAR(50) | `Electronics`, `Accessories`, `Home Appliances`, `Computing`, `Mobile Phones` |
| `quantity` | INTEGER (> 0) | Units ordered |
| `unit_price` | NUMERIC(12,2) | Selling price per unit in KSh, before discount |
| `cost_price` | NUMERIC(12,2) | Cost per unit to the company, in KSh |
| `discount` | NUMERIC(5,2) (0–100) | Discount percentage (`10` = 10% off) |
| `payment_method` | VARCHAR(30) | `M-Pesa`, `Cash`, `Card`, `Bank Transfer` |
| `order_status` | VARCHAR(20) | `Completed`, `Pending`, `Cancelled` |

---

## Business Rules

| Measure | Definition |
|---|---|
| Revenue | `quantity * unit_price * (1 - discount / 100)` |
| Cost | `quantity * cost_price` |
| Profit | `revenue - cost` |
| Profit margin % | `profit / NULLIF(revenue, 0) * 100` |

1. **Financial metrics** (revenue, cost, profit, spending, averages) count **Completed** orders only. Pending orders are not yet paid, and cancelled orders never will be.
2. **Order counts** include every order, whatever its status.
3. **Customer counts** count distinct customers.
4. **"Today"** is the latest `order_date` in the dataset, not the current date, so results can be reproduced.
5. **Months** are grouped by year and month with `DATE_TRUNC('month', order_date)`, so January 2025 and January 2026 count as different months.
6. **Money** is rounded to 2 decimal places.

---

## Methodology

1. **Setup and validation.** I created the schema, loaded the data and confirmed row counts, that there were no orphaned orders, and the date range.
2. **Exploration (Sections A–B).** I profiled customers, products, prices and order statuses, and calculated headline revenue figures.
3. **Customer analysis (Section C).** I joined customers to orders to measure spending by customer, city and segment, and used a `LEFT JOIN` to find customers who never ordered.
4. **Multi-step analysis (Section D).** I used CTEs to compare customers against the average, calculate category profitability, and flag customers inactive for more than 90 days.
5. **Trend and ranking analysis (Section E).** I used window functions for rankings, running totals, month-over-month growth, spending quartiles and revenue share.
6. **Reusable BI layer (Section F).** I built a monthly summary view, a revenue lookup function, value-based customer segmentation and a management report.
7. **Insight writing.** I turned the query outputs into findings, each with its business impact and a recommendation.

---

## Getting Started

### Prerequisites

- PostgreSQL 14 or later
- A SQL client: DBeaver, pgAdmin or `psql`
- The data insert scripts from the project page (not included in this repository)

### Setup

```bash
# 1. Create the database
psql -U postgres -d postgres -c "CREATE DATABASE retail_analytics;"

# 2. Create the tables
psql -U postgres -d retail_analytics -f schema.sql

# 3. Load the data
psql -U postgres -d retail_analytics -f 03_insert_customers.sql
psql -U postgres -d retail_analytics -f 04_insert_orders.sql

# 4. Verify the load
psql -U postgres -d retail_analytics -f 05_verify_data.sql
```

**Expected verification:**

| Check | Expected |
|---|---|
| Customers | 150 |
| Orders | 800 |
| Orders without a customer | 0 |
| Date range | January 2025 – September 2026 |

**Using DBeaver or pgAdmin?** Create a database named `retail_analytics`, open a SQL editor connected
to it, then run `schema.sql`, the two insert scripts and `05_verify_data.sql` in that order.

---

## Running the Analysis

Open `queries.sql` while connected to `retail_analytics`. In DBeaver, place the cursor inside a query
and press **Ctrl+Enter** to run it on its own. To run the whole file:

```bash
psql -U postgres -d retail_analytics -f queries.sql
```

> **Note:** run Q43 before Q46. Q43 creates the `monthly_sales_summary` view that Q46 uses.

| Section | Questions | Techniques |
|---|---|---|
| A. Fundamentals | Q1–Q10 | `SELECT`, `WHERE`, `DISTINCT`, `ORDER BY`, `LIMIT`, `LIKE`, `BETWEEN` |
| B. Aggregation | Q11–Q20 | `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`, `GROUP BY`, `HAVING`, `FILTER` |
| C. Joins | Q21–Q28 | `INNER JOIN`, `LEFT JOIN`, customer-level aggregation |
| D. Subqueries & CTEs | Q29–Q34 | Nested aggregation, `WITH` clauses, date arithmetic |
| E. Window Functions | Q35–Q42 | `RANK`, `DENSE_RANK`, `ROW_NUMBER`, `LAG`, `NTILE`, running totals |
| F. Business Intelligence | Q43–Q46 | Views, PL/pgSQL functions, `CASE` segmentation, executive reporting |

### Database objects created

| Object | Type | Purpose |
|---|---|---|
| `monthly_sales_summary` | View | Monthly total orders, completed orders, revenue, cost and profit |
| `customer_revenue(p_customer_id)` | Function | Completed-order revenue for a customer, returning 0 for unknown IDs |

```sql
SELECT * FROM monthly_sales_summary ORDER BY month;

SELECT customer_revenue(33);    -- 848,118.75 (highest-spending customer)
SELECT customer_revenue(9999);  -- 0 (ID does not exist)
```

---

## Featured Queries

### Month-over-month revenue growth (Q40)

A CTE builds monthly revenue, then `LAG()` compares each month with the one before it.

```sql
WITH monthly AS (
    SELECT DATE_TRUNC('month', order_date)::DATE AS month,
           SUM(quantity * unit_price * (1 - discount / 100)) AS revenue
    FROM orders
    WHERE order_status = 'Completed'
    GROUP BY DATE_TRUNC('month', order_date)
)
SELECT month,
       ROUND(revenue, 2) AS revenue,
       ROUND((revenue - LAG(revenue) OVER (ORDER BY month))
             / NULLIF(LAG(revenue) OVER (ORDER BY month), 0) * 100, 2) AS mom_growth_pct
FROM monthly
ORDER BY month;
```

### Customers drifting away (Q34)

This query flags customers whose last completed order was more than 90 days before the latest date in the data.

```sql
WITH dataset_today AS (SELECT MAX(order_date) AS today FROM orders),
last_completed AS (
    SELECT customer_id, MAX(order_date) AS last_completed_order
    FROM orders
    WHERE order_status = 'Completed'
    GROUP BY customer_id
)
SELECT c.customer_name, c.customer_segment, lc.last_completed_order,
       d.today - lc.last_completed_order AS days_since_last_order
FROM last_completed lc
JOIN customers c ON c.customer_id = lc.customer_id
CROSS JOIN dataset_today d
WHERE d.today - lc.last_completed_order > 90
ORDER BY days_since_last_order DESC;
```

### Value-based customer segmentation (Q45)

Customers are grouped into value bands using **percentiles** rather than fixed KSh amounts. Spending
varies widely (from KSh 1,899 to KSh 848,119), so fixed cut-offs would put most customers into one
band, while percentiles split them in proportion to the data.

| Band | Rule | Threshold |
|---|---|---|
| High Value | Top 25% of purchasing customers | ≥ KSh 252,835.23 |
| Medium Value | Middle 50% | KSh 35,201.33 – 252,835.23 |
| Low Value | Bottom 25% | < KSh 35,201.33 |
| No Purchases | No completed orders | 18 customers |

---

## Results Snapshot

### Top five products by completed revenue (Q46b)

| Product | Category | Units | Revenue (KSh) |
|---|---|---:|---:|
| iPhone 13 | Mobile Phones | 45 | 3,766,458.15 |
| HP 250 G9 Laptop | Computing | 43 | 2,510,510.15 |
| Hotpoint Refrigerator | Home Appliances | 46 | 2,386,956.60 |
| Lenovo IdeaPad 3 | Computing | 37 | 2,032,875.00 |
| Samsung 55in Smart TV | Electronics | 24 | 1,831,977.10 |

### Top five customers by completed revenue (Q46a)

| Customer | Segment | Revenue (KSh) |
|---|---|---:|
| Brian Chebet | Retail | 848,118.75 |
| Peter Ndungu | Retail | 790,229.50 |
| Lucy Njoroge | Retail | 767,880.80 |
| Nancy Korir | Retail | 702,545.75 |
| Purity Ochieng | Retail | 698,921.60 |

### Customer spending distribution (Q45)

| Statistic | KSh |
|---|---:|
| Paying customers | 132 |
| Minimum | 1,899.00 |
| 25th percentile | 35,201.33 |
| Median | 89,999.00 |
| 75th percentile | 252,835.23 |
| Maximum | 848,118.75 |

### Management report highlights (Q46)

| Metric | Result |
|---|---|
| Most profitable category | Mobile Phones: KSh 728,760.65 profit on KSh 6,942,060.65 revenue (10.5% margin) |
| Lowest-revenue month | January 2026: KSh 322,432.95 from 25 orders |

---

## Recommendations

1. **Protect the top five products.** Set reorder alerts and arrange backup suppliers, and sell accessories alongside each big-ticket sale.
2. **Stop discounting phones.** At a 10.5% margin, attract buyers with low-cost add-ons such as cases and screen protectors instead.
3. **Re-segment customers by spending.** Invite High Value Retail customers into the VIP programme and review segments every quarter.
4. **Manage key accounts.** Give the top 10 customers a named contact and early access to new stock, and help Medium Value customers move up a band.
5. **Plan for January.** Offer back-to-school instalment and M-Pesa payment plans, and set a minimum monthly margin target.

---

## Assumptions & Limitations

- The data is **fictional** and covers only 21 months, so seasonal patterns are based on at most two of each calendar month.
- The **latest month may be incomplete**, so its revenue can look artificially low.
- `cost_price` is treated as the full unit cost. Overheads such as rent, staff and delivery are not included, so profit here is **gross profit**.
- Customer segments (Retail, Corporate, VIP) are taken as recorded. Their criteria are not documented.
- "Popular" category among VIP customers (Q28) is measured by **number of orders**, with units shown as a secondary measure.

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

| Area | Skills |
|---|---|
| Database design | Primary and foreign keys, `CHECK` constraints, data validation |
| Core SQL | Filtering, sorting, aggregation, `GROUP BY` / `HAVING`, conditional counts with `FILTER` |
| Joins | Inner and left joins, finding missing records |
| Advanced SQL | CTEs, subqueries, window functions, percentiles |
| Database programming | Views, PL/pgSQL functions, `NULL` handling with `COALESCE` / `NULLIF` |
| Analytics | Profitability analysis, trend analysis, customer segmentation, churn detection |
| Communication | Turning query results into findings and recommendations for management |

---

## Future Improvements

- Connect the database to **Power BI** or **Tableau** for an interactive sales dashboard.
- Add a **products** table to normalise product and category data.
- Build an **RFM (Recency, Frequency, Monetary)** model for finer customer segmentation.
- Analyse **cancellation patterns** by product, city and payment method.
- Automate monthly reporting with scheduled refreshes of the summary view.

---

## Author

**Hope Gatugi**