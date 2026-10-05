[README.md](https://github.com/user-attachments/files/33063700/README.md)
# 📦 Vendor Performance Analytics: End-to-End Project

![Python](https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-NumPy-150458?logo=pandas&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-15M%2B%20rows-003B57?logo=sqlite&logoColor=white)
![SQL](https://img.shields.io/badge/SQL-CTEs%20%26%20Indexing-orange)
![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)

> An end-to-end analytics pipeline that loads **15.6M+ raw retail records** into SQLite, optimizes queries with indexes, engineers vendor-level KPIs, and prepares a clean dataset for Power BI reporting.

**Pipeline:** `CSV files → SQLite (inventory.db) → SQL / pandas analysis → vendor_sales_summary → Power BI`

---

## 📑 Table of Contents
1. [Business Problem & Objectives](#-business-problem--objectives)
2. [Dataset](#-dataset)
3. [Tech Stack](#-tech-stack)
4. [Project Pipeline](#-project-pipeline)
5. [Engineered KPIs](#-engineered-kpis)
6. [Key Business Insights](#-key-business-insights)
7. [Power BI Dashboard](#-power-bi-dashboard)
8. [Repository Structure](#-repository-structure)
9. [How to Run](#-how-to-run)
10. [Future Improvements](#-future-improvements)
11. [Author](#-author)

---

## 🎯 Business Problem & Objectives

A retail distributor buys from many vendors and sells thousands of brands across stores, but purchase, sales, inventory and invoice data sit in six separate files. That makes it hard to answer basic procurement questions:

- Which vendors and brands are the most (and least) profitable?
- How much money is tied up in unsold inventory?
- How dependent is the business on a handful of vendors?
- Do freight costs or delivery delays hurt specific vendors' value?

**Objectives**

| # | Objective | Deliverable |
|---|-----------|-------------|
| 1 | Consolidate six raw files into one queryable database | `inventory.db` |
| 2 | Make large-table analysis fast and memory-safe | Chunked loading + SQL indexes |
| 3 | Measure vendor and brand profitability and inventory efficiency | Engineered KPIs |
| 4 | Quantify purchase concentration risk | Top-10 vendor contribution analysis |
| 5 | Provide one clean table for BI tools | `vendor_sales_summary` table and CSV |

---

## 🗂 Dataset

Six CSV files, loaded into SQLite tables of the same name:

| Table | Description | Rows |
|-------|-------------|-----:|
| `sales` | Transaction-level sales | 12,825,363 |
| `purchases` | Purchase order line items | 2,372,474 |
| `end_inventory` | Inventory snapshot at end of period | 224,489 |
| `begin_inventory` | Inventory snapshot at start of period | 206,529 |
| `purchase_prices` | Vendor purchase price and retail price per brand | 12,261 |
| `vendor_invoice` | Vendor invoices, including freight | 5,543 |
| | **Total** | **≈ 15.65M** |

> The raw data is not included in this repository because of its size (GitHub limits files to 100 MB). Dataset source: **[add link or source name here]**. Place the CSVs in a `data/` folder to reproduce the project.

---

## 🛠 Tech Stack

| Layer | Tools |
|-------|-------|
| Language | Python 3.12 |
| Data wrangling | Pandas, NumPy |
| Database | SQLite, SQLAlchemy, SQL (CTEs, joins, aggregations, date functions) |
| Visualization | Matplotlib, Seaborn |
| Statistics | SciPy (Welch's t-test) |
| BI | Power BI, DAX |
| Environment | Jupyter Notebook (VS Code), Git / GitHub |

---

## ⚙️ Project Pipeline

The whole workflow lives in [`project.ipynb`](project.ipynb), organized in five sections.

### 1️⃣ Setup & Ingestion
- Scans the data directory and loads **every CSV** into SQLite automatically.
- **Chunked reads (100,000 rows)** with `pandas` and `to_sql(...)` keep memory low. The 12.8M-row `sales` file loads without crashing.
- **Encoding fallback:** `latin1` → `utf-8` → `cp1252`.
- **Error handling and logging** to console and `ingestion_db.log`; one failed file never stops the rest.
- Full ingestion of all six files took about **5.4 minutes**.

### 2️⃣ SQL Optimization
Indexes on the join and group-by keys, followed by `ANALYZE` to refresh query-planner statistics:

```sql
CREATE INDEX IF NOT EXISTS idx_purchases_vendor_brand ON purchases (VendorNumber, Brand);
CREATE INDEX IF NOT EXISTS idx_sales_vendor_brand     ON sales (VendorNo, Brand);
CREATE INDEX IF NOT EXISTS idx_invoice_vendor         ON vendor_invoice (VendorNumber);
CREATE INDEX IF NOT EXISTS idx_prices_brand           ON purchase_prices (Brand);
```

### 3️⃣ Database Verification
Table listing from `sqlite_master`, row counts, `PRAGMA table_info` schemas and sample rows for every table.

### 4️⃣ Exploratory Data Analysis (SQL + pandas)
- **Data quality:** memory-safe, per-column NULL checks across all tables.
- **Vendor performance:** top 10 vendors by purchase spend, purchase concentration, delivery lead times (PO date → receiving date) and freight as a percentage of invoice value.
- **Purchases vs. sales:** brand-level gross margin (top and loss-making brands) and the monthly sales trend.
- **Inventory turnover:** begin vs. end inventory value per brand and turnover = COGS ÷ average inventory (COGS approximated by purchase dollars).
- **Top performers:** top products, vendors and stores by sales.

### 5️⃣ Feature Engineering: `vendor_sales_summary`
A CTE query joins purchases, purchase prices, sales and freight into **one row per vendor-brand**, then Python adds the KPIs below. The result is saved back to SQLite as `vendor_sales_summary` and exported to `vendor_sales_summary.csv` for Power BI.

Further analysis on the summary table:
- Promotion candidates (bottom 15% sales, top 15% margin)
- Top-10 vendor purchase contribution (pie chart)
- Bulk-purchase effect on unit cost (small / medium / large orders)
- Unsold inventory value by vendor
- Welch's t-test on profit margins of top-quartile vs. bottom-quartile vendors

---

## 📐 Engineered KPIs

| KPI | Formula |
|-----|---------|
| **Gross Profit** | `TotalSalesDollars − TotalPurchaseDollars` |
| **Profit Margin (%)** | `Gross Profit ÷ TotalSalesDollars × 100` |
| **Stock Turnover** | `TotalSalesQuantity ÷ TotalPurchaseQuantity` |
| **Sales-to-Purchase Ratio** | `TotalSalesDollars ÷ TotalPurchaseDollars` |
| **Unsold Inventory Value** | `(TotalPurchaseQuantity − TotalSalesQuantity) × PurchasePrice` |

---

## 💡 Key Business Insights

> ⚠️ **Fill in the bracketed values from your notebook outputs before publishing.** The "Where to find it" column tells you which cell prints each number.

| Insight | Result | Where to find it | Recommended action |
|---------|--------|------------------|--------------------|
| **Purchase concentration risk** | Top 10 vendors = **[X]%** of total purchase spend | Section 3.2, concentration cell | Diversify sourcing; negotiate contingency terms with top vendors |
| **Capital tied up in unsold stock** | **$[X]** total unsold inventory value | Section 4.1, unsold inventory cell | Reduce reorder quantities for the vendors with the largest balances |
| **Slow-moving inventory** | Median turnover **[X]**; slowest brands: **[names]** | Section 3.4, turnover cell | Cut purchasing on low-turnover brands |
| **Loss-making brands** | **[N]** brands with negative gross margin | Section 3.3, bottom-10 output | Renegotiate cost prices or delist |
| **Promotion opportunities** | **[N]** brands combine low sales with high margin | Section 4.1, first cell | Promote or improve shelf placement |
| **Bulk buying** | Avg. unit cost: Small **$[X]**, Medium **$[X]**, Large **$[X]** | Section 4.1, bulk cell | Consolidate orders if large orders are cheaper |
| **Freight impact** | Highest freight share of invoice: **[X]%** (**[vendor]**) | Section 3.2, freight cell | Renegotiate shipping terms |
| **Vendor tier margins** | Top vendors **[X]%** vs. low vendors **[Y]%**, p = **[value]** | Section 4.1, t-test cell | Tier vendor negotiation strategy |
| **Seasonality** | Sales peak in **[month]**, lowest in **[month]** | Section 3.3, monthly trend chart | Align purchasing with demand cycles |

---

## 📊 Power BI Dashboard

The notebook exports `vendor_sales_summary.csv`, which is the single data source for the dashboard.

**Setup:** *Get Data → Text/CSV → `vendor_sales_summary.csv`*. Set IDs to Text, dollars and quantities to Decimal, and `ProfitMargin` to Percentage.

**DAX measures**
```DAX
Total Sales      = SUM(vendor_sales_summary[TotalSalesDollars])
Total Purchases  = SUM(vendor_sales_summary[TotalPurchaseDollars])
Gross Profit     = [Total Sales] - [Total Purchases]
Profit Margin %  = DIVIDE([Gross Profit], [Total Sales])
Stock Turnover   = DIVIDE(SUM(vendor_sales_summary[TotalSalesQuantity]),
                          SUM(vendor_sales_summary[TotalPurchaseQuantity]))
Unsold Inventory = SUM(vendor_sales_summary[UnsoldInventoryValue])
```

**Dashboard layout:** KPI cards (Total Sales, Total Purchases, Gross Profit, Profit Margin %, Unsold Inventory), top-10 vendor and brand bar charts, purchase-contribution donut, margin-vs-sales scatter, low-turnover table and slicers for Vendor, Brand and Description.

<!-- After you build the dashboard, save a screenshot as images/dashboard.png and uncomment the next line -->
<!-- ![Dashboard](images/dashboard.png) -->

---

## 📁 Repository Structure

```
vendor-performance-analytics/
├── project.ipynb        # Full pipeline: ingestion → SQL → EDA → feature engineering
├── README.md
└── .gitignore           # Excludes data/, *.db, *.csv, logs and checkpoints
```

Files generated when you run the notebook (not tracked in Git):

```
data/                        # raw CSVs (you provide)
inventory.db                 # SQLite database
ingestion_db.log             # ingestion audit log
vendor_sales_summary.csv     # export for Power BI
```

---

## 🚀 How to Run

**1. Clone the repo**
```bash
git clone https://github.com/atharvagit22/vendor-performance-analytics.git
cd vendor-performance-analytics
```

**2. Install dependencies** (Python 3.10+)
```bash
pip install pandas numpy matplotlib seaborn scipy sqlalchemy jupyter
```

**3. Add the data**
Create a `data/` folder next to the notebook and put the six CSV files in it.

**4. Run the notebook**
```bash
jupyter notebook project.ipynb
```
Run all cells from top to bottom. Allow roughly **5 to 6 minutes** for ingestion, plus a few more for the larger aggregation queries on `sales`.

**5. Explore the output**
```python
import sqlite3, pandas as pd
conn = sqlite3.connect("inventory.db")
pd.read_sql("SELECT * FROM vendor_sales_summary ORDER BY GrossProfit DESC LIMIT 10", conn)
```

---

## 🔮 Future Improvements

- Incremental loads instead of `if_exists="replace"`
- Demand forecasting (Prophet or SARIMA) on monthly sales
- Move the notebook logic into scheduled scripts (`ingestion_db.py`, `get_vendor_summary.py`)
- Automated data-quality checks for invalid prices, quantities and duplicate join keys
- Migrate from SQLite to PostgreSQL for concurrent access

---

## 👤 Author

**Atharva Lambde**
📧 atharvalambde@gmail.com · 🐙 [GitHub](https://github.com/atharvagit22)

⭐ If you found this project useful, please star the repository.
