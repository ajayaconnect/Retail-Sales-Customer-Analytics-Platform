# 🛒 Retail Sales & Customer Analytics Platform

![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-336791?logo=postgresql)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow?logo=powerbi)
![RFM](https://img.shields.io/badge/Analysis-RFM%20Segmentation-purple)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

---

## 📌 Business Problem

An e-commerce company wants to understand its customer base, product performance, and delivery operations — but all data is siloed across separate CSV files with no unified reporting layer.

**This project builds a retail analytics platform** that:
- Unifies 5 operational data tables in PostgreSQL
- Segments all customers using RFM (Recency, Frequency, Monetary) analysis in SQL
- Identifies top revenue categories and underperforming SKUs
- Analyses delivery performance and its impact on customer satisfaction
- Delivers a 4-page Power BI dashboard for business decision-making

---

## 📊 Dataset

| Attribute | Detail |
|-----------|--------|
| Source | [Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce) |
| Total Orders | ~100,000 real orders (2016–2018) |
| Tables | 5 CSV files (orders, items, customers, products, reviews) |
| Geography | Brazil — 27 states |
| Data Type | Real anonymised data from Olist (Brazilian marketplace) |

> ⚠️ Raw data is not included in this repository. Download from the Kaggle link above.

---

## 🛠️ Tools & Technologies

| Layer | Tools |
|-------|-------|
| Data Storage | PostgreSQL 15, pgAdmin 4 |
| Data Analysis | Python, Pandas, NumPy, SQLAlchemy |
| Visualisation | Matplotlib, Seaborn |
| Dashboard | Microsoft Power BI Desktop |
| Version Control | Git, GitHub |

---

## 📁 Project Structure

```
retail-sales-customer-analytics/
├── data/
│   ├── load_data.py               # Load all 5 CSVs → PostgreSQL
│   └── processed/
│       └── rfm_segments.csv       # RFM output exported from SQL
├── notebooks/
│   └── 01_eda.ipynb               # EDA + 5 visualisations
├── sql/
│   └── retail_queries.sql         # 7 business SQL queries
├── reports/
│   ├── 01_monthly_revenue_trend.png
│   ├── 02_top_categories.png
│   ├── 03_review_distribution.png
│   ├── 04_delivery_analysis.png
│   └── 05_state_performance.png
├── .gitignore
└── README.md
```

---

## 🗄️ Database Schema

5 tables loaded into PostgreSQL `retail_db`:

```
orders          — order_id, customer_id, order_status, timestamps, delivery dates
items           — order_id, product_id, price, freight_value
customers       — customer_id, customer_city, customer_state
products        — product_id, product_category_name
reviews         — order_id, review_score, review_comment_title
```

**Relationships:**
- `orders` → `items` via `order_id`
- `orders` → `reviews` via `order_id`
- `orders` → `customers` via `customer_id`
- `items` → `products` via `product_id`

---

## 🔍 SQL Analysis Highlights

7 business queries written in PostgreSQL — see full file: [`sql/retail_queries.sql`](sql/retail_queries.sql)

**Most important: Full RFM Segmentation Query**

```sql
WITH last_date AS (
    SELECT MAX(order_purchase_timestamp::date) AS max_date FROM orders
),
rfm_raw AS (
    SELECT
        o.customer_id,
        (SELECT max_date FROM last_date) - MAX(o.order_purchase_timestamp::date) AS recency_days,
        COUNT(DISTINCT o.order_id) AS frequency,
        ROUND(SUM(i.price)::numeric, 2) AS monetary
    FROM orders o
    JOIN items i ON o.order_id = i.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY o.customer_id
),
rfm_scored AS (
    SELECT *,
        NTILE(5) OVER (ORDER BY recency_days ASC)  AS r_score,
        NTILE(5) OVER (ORDER BY frequency DESC)    AS f_score,
        NTILE(5) OVER (ORDER BY monetary DESC)     AS m_score
    FROM rfm_raw
)
SELECT *,
    CASE
        WHEN r_score + f_score + m_score >= 13 THEN 'Champion'
        WHEN r_score + f_score + m_score >= 10 THEN 'Loyal'
        WHEN r_score >= 4 AND f_score <= 2      THEN 'New Customer'
        WHEN r_score <= 2 AND f_score >= 3      THEN 'At Risk'
        ELSE 'Need Attention'
    END AS segment
FROM rfm_scored;
```

---

## 📈 Key Business Findings

| Finding | Insight |
|---------|---------|
| **Total Revenue** | *(fill in after running analysis)* |
| **Avg Review Score** | *(fill in after running analysis)* |
| **Late Delivery Rate** | *(fill in after running analysis)* |
| **Top Revenue Category** | *(fill in after running analysis)* |
| **Largest RFM Segment** | *(fill in after running analysis)* |
| **Best State by Revenue** | São Paulo leads by significant margin |

> Fill in these values with your actual query results before publishing.

---

## 👥 RFM Customer Segmentation

RFM (Recency, Frequency, Monetary) is an industry-standard method for customer segmentation.

| Segment | Definition | Business Action |
|---------|-----------|-----------------|
| **Champion** | Bought recently, frequently, high spend | Reward and upsell |
| **Loyal** | Regular buyers, good spend | Retention offers |
| **New Customer** | Recent first purchase | Onboarding campaign |
| **At Risk** | Used to buy frequently, gone quiet | Win-back campaign |
| **Need Attention** | Low score across all 3 | Low priority / re-engage |

Scoring method: `NTILE(5)` in PostgreSQL — each customer ranked 1–5 on Recency, Frequency, and Monetary independently, then summed.

---

## 📊 Power BI Dashboard

4-page dashboard connected to PostgreSQL with table relationships set in Model View:

| Page | Content |
|------|---------|
| **Executive Dashboard** | Total orders, revenue, avg order value, review score KPIs, monthly trend |
| **Category Analysis** | Top 10 categories by revenue, treemap, avg price vs review scatter |
| **Customer Intelligence** | RFM segment distribution, avg spend by segment, top Champions |
| **Geography & Delivery** | Revenue by state, delivery delay map, on-time vs late donut |

**DAX Measures created:** Total Revenue, Total Orders, Avg Order Value, Avg Review Score, Avg Delivery Days, Late Delivery Rate %

---

## ▶️ How to Run

### 1. Install Libraries
```bash
pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary
```

### 2. Create Database
- pgAdmin → New database: `retail_db`

### 3. Load Data
```bash
# Edit YOUR_PASSWORD first
python data/load_data.py
```

### 4. Run SQL Queries
- Open pgAdmin → Query Tool for `retail_db`
- Run queries from `sql/retail_queries.sql`
- Export RFM output as `data/processed/rfm_segments.csv`

### 5. Run EDA Notebook
```
notebooks/01_eda.ipynb
```

### 6. Open Dashboard
- Open `powerbi/retail_dashboard.pbix` in Power BI Desktop
- Update PostgreSQL connection credentials

---

## 💬 Interview Talking Points

**Q: What is RFM and why did you use it?**
> RFM stands for Recency, Frequency, Monetary. It is a proven customer segmentation technique used by retail and e-commerce companies worldwide. I used `NTILE(5)` in PostgreSQL to rank all customers 1–5 on each dimension, then combined the scores to classify customers into actionable segments like Champions and At Risk.

**Q: Why did you use real Olist data instead of a synthetic dataset?**
> The Olist dataset is real, anonymised transaction data from a live Brazilian marketplace. This makes the findings credible — the seasonal revenue spikes, review score distributions, and state-level delivery patterns reflect actual business operations. It also demonstrates my ability to work with messy, multi-table real-world data.

**Q: What was the most interesting SQL challenge?**
> The RFM query uses a Common Table Expression (CTE) chain with window functions (`NTILE(5)`) and a multi-condition CASE statement. Writing it correctly required understanding that `NTILE` ordering direction matters — Recency should be ordered `ASC` (lower recency days = more recent = better), while Frequency and Monetary use `DESC`.

---

## 👤 Author

**Ajaya Kumar Pradhan**
- 📧 ajayapradhan210@gmail.com
- 💼 [LinkedIn](https://linkedin.com/in/ajaya-pradhan-1945341b0)
- 🐙 [GitHub](https://github.com/Ajaya210)

---

*Part of a 3-project Data Analyst portfolio. See also: [Credit Risk Analytics](https://github.com/Ajaya210/credit-risk-loan-default) | [Supply Chain Analytics](https://github.com/Ajaya210/supply-chain-vendor-performance)*
