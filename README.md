# Vendor Sales & Performance Analysis

A SQL- and Python-based analytics project that combines purchase, sales, pricing, and freight data to evaluate vendor and brand performance. The notebooks import CSV files into MySQL, build a consolidated vendor–brand summary table, and use exploratory data analysis and statistical summaries to investigate procurement concentration, product pricing, inventory movement, and profitability.

> **Project format:** The supplied archive contains three Jupyter notebooks. It does not contain the source CSV files, a MySQL database dump, or a standalone Python application.

## Table of Contents

- [Objectives](#objectives)
- [Repository Structure](#repository-structure)
- [Technology Stack](#technology-stack)
- [Data Sources](#data-sources)
- [Workflow](#workflow)
- [Key Metrics](#key-metrics)
- [Analysis and Findings](#analysis-and-findings)
- [Setup and Execution](#setup-and-execution)
- [Important Implementation Notes](#important-implementation-notes)
- [Limitations and Next Steps](#limitations-and-next-steps)

## Objectives

1. Consolidate vendor purchase, product pricing, sales, and freight information into an analysis-ready table.
2. Identify vendors that account for a large share of procurement spending.
3. Compare purchase quantities with sales quantities to identify slow-moving inventory.
4. Examine the relationship between purchase volume and unit purchase cost.
5. Identify brands with relatively low sales but high profit margins for further pricing or promotional review.
6. Compare profit margins across high-sales and low-sales vendor–brand records.

## Repository Structure

```text
Vendor_sales_analysis-main/
├── database_ingestion.ipynb
├── exploratory_data_analysis.ipynb
└── Vendor_performance_Analysis.ipynb
```

### Notebook responsibilities

| Notebook | Purpose |
|---|---|
| `database_ingestion.ipynb` | Creates MySQL tables from CSV headers and bulk-loads CSV records using `LOAD DATA LOCAL INFILE`. |
| `exploratory_data_analysis.ipynb` | Inspects source tables, explores relationships between purchases, prices, sales, and invoices, and constructs `vendor_sales_summary`. |
| `Vendor_performance_Analysis.ipynb` | Performs summary statistics, data filtering, correlation analysis, brand analysis, vendor contribution analysis, bulk-purchase analysis, inventory estimates, and confidence-interval comparisons. |

## Technology Stack

- **Language:** Python
- **Database:** MySQL
- **Data manipulation:** pandas, NumPy
- **Database connectivity:** SQLAlchemy, PyMySQL
- **Visualization:** Matplotlib, Seaborn
- **Statistics:** SciPy (`scipy.stats`)
- **Development environment:** Jupyter Notebook

## Data Sources

The ingestion notebook expects CSV files in a local data directory. The file name (without `.csv`) becomes the MySQL table name.

| CSV file / MySQL table | Role in the analysis |
|---|---|
| `begin_inventory.csv` / `begin_inventory` | Beginning inventory records. It is imported by the ingestion notebook; the supplied analysis flow does not directly use it in the final summary query. |
| `end_inventory.csv` / `end_inventory` | Ending inventory records. It is imported by the ingestion notebook; the supplied analysis flow does not directly use it in the final summary query. |
| `purchases.csv` / `purchases` | Vendor purchase transactions, including vendor, brand, quantity, purchase amount, and purchase price. |
| `purchase_prices.csv` / `purchase_prices` | Brand/vendor price reference data, including purchase price, actual price, and volume. |
| `sales.csv` / `sales` | Sales transactions, including vendor reference, brand, sales quantity, sales dollars, sales price, and excise tax. |
| `vendor_invoice.csv` / `vendor_invoice` | Vendor invoice records, including purchase-order information and freight charges. |

The saved ingestion notebook output reports the following imported row counts from its previous run. These are notebook-run outputs, not a guarantee about a different copy of the source data.

| Table | Rows reported |
|---|---:|
| `begin_inventory` | 206,529 |
| `end_inventory` | 224,489 |
| `purchases` | 2,372,474 |
| `purchase_prices` | 12,261 |
| `sales` | 12,825,363 |
| `vendor_invoice` | 5,543 |

## Workflow

### 1. CSV ingestion

`database_ingestion.ipynb`:

1. Connects to the local MySQL database `vendor_analysis`.
2. Reads a sample of each CSV with pandas to infer column types.
3. Creates or replaces a MySQL table using the CSV's column names.
4. Loads the full CSV through MySQL's `LOAD DATA LOCAL INFILE` command.

### 2. Source-table exploration and aggregation

`exploratory_data_analysis.ipynb` reviews table counts and samples, examines selected vendor records, and calculates purchase, sales, and freight aggregates.

The final summary query uses common table expressions (CTEs):

- **`FreightSummary`:** sums invoice freight by `VendorNumber`.
- **`PurchaseSummary`:** aggregates purchase quantity and purchase dollars by vendor and brand, with product pricing details.
- **`SalesSummary`:** aggregates sales quantity, sales dollars, sales price, and excise tax by vendor and brand.

The CTE outputs are joined to produce `vendor_sales_summary`. The notebook then standardizes `Volume` to a numeric type, fills missing values, trims vendor names, calculates performance metrics, replaces infinite values with nulls, and writes the result to MySQL.

### 3. Exploratory data analysis

`Vendor_performance_Analysis.ipynb` reads `vendor_sales_summary` from MySQL and examines:

- Descriptive statistics and numeric distributions.
- Potential outliers using boxplots.
- Vendor and product frequency counts.
- Correlations between numeric measures.
- Records with positive gross profit, positive profit margin, and positive sales quantity.

### 4. Vendor and brand analysis

The analysis notebook investigates:

- Brands with low sales and high average profit margin.
- Top vendors and brands by total sales dollars.
- Vendor contribution to total purchase dollars.
- Purchase quantity bands and their average unit purchase cost.
- Vendors with low sales-to-purchase quantity ratios.
- Estimated value of purchased quantity not matched by sales quantity.
- Confidence intervals for margins in high-sales and low-sales groups.

## Key Metrics

The notebook calculates the following fields for each vendor–brand record:

| Metric | Calculation in the notebook | Interpretation |
|---|---|---|
| `GrossProfit` | `TotalSalesDollars - TotalPurchaseDollars` | Simplified gross-profit proxy based on sales and purchase dollars. It does not deduct freight, excise tax, or other operating costs. |
| `ProfitMargin` | `(GrossProfit / TotalSalesDollars) * 100` | Simplified margin percentage. It is undefined when sales dollars are zero. |
| `StockTurnover` | `TotalSalesQuantity / TotalPurchaseQuantity` | Sales-to-purchase quantity ratio over the data period. This is a project-defined movement indicator, not the conventional COGS-to-average-inventory turnover formula. |
| `SalesToPurchaseRatio` | `TotalSalesDollars / TotalPurchaseDollars` | Sales dollars relative to purchase dollars. |
| `UnitPurchasePrice` | `TotalPurchaseDollars / TotalPurchaseQuantity` | Average purchase spend per unit for a vendor–brand record. |
| `UnsoldInventoryValue` | `(TotalPurchaseQuantity - TotalSalesQuantity) * PurchasePrice` | Approximation of the value of purchased units not matched by sales quantity. It is not a verified ending-inventory valuation. |
| `PurchaseContribution%` | Vendor purchase dollars divided by total purchase dollars, multiplied by 100 | Vendor share of overall procurement spend. |

## Analysis and Findings

The figures below are taken from outputs saved in the supplied notebooks. They describe the dataset used when those notebooks were last run.

### Data preparation

- The consolidated `vendor_sales_summary` contains **10,692 rows** and **18 columns**.
- The analysis notebook filters to records where `GrossProfit > 0`, `ProfitMargin > 0`, and `TotalSalesQuantity > 0`.
- This filtered dataset contains **8,565 rows**.
- The saved summary statistics show zero sales for some records and very large price/freight outliers. Infinite profit margins occur when sales dollars are zero; the preparation notebook replaces infinite values with nulls before saving the table.

### Brands with low sales and high margins

The notebook groups records by `Description`, sums sales dollars, and averages profit margin. It selects:

- Low sales: at or below the **15th percentile** of brand sales dollars.
- High margin: at or above the **85th percentile** of average brand profit margin.

Saved thresholds:

- Low-sales threshold: **$560.299**
- High-margin threshold: **64.9702%**
- Brands meeting both conditions: **198**

These brands are candidates for further investigation; the analysis does not establish why their sales are low.

### Vendor procurement concentration

The top 10 vendors account for **65.34%** of total purchase dollars in the saved output.

The highest-contributing vendor shown is **DIAGEO NORTH AMERICA INC**, with:

- Purchase dollars: approximately **$50.96 million**
- Share of purchase dollars: **15.83%**
- Sales dollars: approximately **$68.74 million**
- Simplified gross profit: approximately **$17.78 million**

The notebook also produces a Pareto-style chart and a donut chart for vendor procurement contribution.

### Purchase volume and unit purchase price

The notebook divides vendor–brand purchase quantities into three quantile-based groups using `pandas.qcut` and compares average unit purchase price.

| Purchase-size group | Mean unit purchase price in saved output |
|---|---:|
| Small | $43.78 |
| Medium | $17.89 |
| Large | $11.31 |

The output suggests an association between larger aggregated purchase quantities and lower average unit purchase prices. It does not prove that increasing order size alone causes the lower price.

### Inventory movement and estimated unsold value

- The notebook identifies vendors with `StockTurnover < 1` as having sales quantities below purchase quantities in the analysis period.
- The estimated total unsold inventory value is approximately **$8.75 million** using the notebook's quantity-difference formula.
- The ten-vendor breakdown highlights vendors with the largest estimated value under this calculation.

### Profit-margin confidence intervals

The notebook defines high-sales and low-sales groups using the upper and lower quartiles of `TotalSalesDollars`, removes margin outliers with the IQR rule, and calculates 95% t-based confidence intervals.

| Group | Mean profit margin | 95% confidence interval |
|---|---:|---:|
| Top-sales group | 30.82% | 30.49%–31.16% |
| Low-sales group | 10.88% | 8.50%–13.26% |

In this notebook, the grouping is performed on vendor–brand rows using sales-dollar cutoffs. The confidence intervals should therefore not be described as estimates based on a sample of unique vendors.

## Setup and Execution

### Prerequisites

- Python 3.9+ (the notebooks do not pin a specific Python version).
- MySQL server with permission to create tables in a database named `vendor_analysis`.
- The six source CSV files listed in [Data Sources](#data-sources).
- MySQL `LOCAL INFILE` enabled on both the server and the PyMySQL connection for the bulk-load step.

### 1. Install Python dependencies

```bash
python -m venv .venv
```

Activate the environment, then install the libraries used in the notebooks:

```bash
# Windows PowerShell
.venv\Scripts\Activate.ps1

# macOS / Linux
source .venv/bin/activate

python -m pip install pandas numpy matplotlib seaborn scipy sqlalchemy pymysql jupyter
```

### 2. Create the database

Run this in MySQL using an account that has permission to create databases:

```sql
CREATE DATABASE vendor_analysis;
```

### 3. Configure database access and the CSV directory

The supplied notebooks contain local machine paths and a hard-coded database connection. Update those settings for your environment before running them. Do not commit database passwords to Git.

For example, use environment variables in your own copy of the notebook:

```python
import os
from sqlalchemy import create_engine

DB_USER = os.getenv("MYSQL_USER", "root")
DB_PASSWORD = os.environ["MYSQL_PASSWORD"]
DB_HOST = os.getenv("MYSQL_HOST", "localhost")
DB_PORT = os.getenv("MYSQL_PORT", "3306")
DB_NAME = os.getenv("MYSQL_DATABASE", "vendor_analysis")

engine = create_engine(
    f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_NAME}"
)

DATA_DIR = os.getenv("VENDOR_DATA_DIR", r"C:\path\to\vendor_analysis\data")
```

For the ingestion notebook's direct PyMySQL connection, configure the same environment-specific values and keep `local_infile=True` where required by your MySQL setup.

### 4. Place the source files

Set `DATA_DIR` to a directory containing:

```text
begin_inventory.csv
end_inventory.csv
purchases.csv
purchase_prices.csv
sales.csv
vendor_invoice.csv
```

### 5. Run the notebooks in order

Start Jupyter:

```bash
jupyter notebook
```

Run the notebooks in this sequence:

1. `database_ingestion.ipynb`
2. `exploratory_data_analysis.ipynb`
3. `Vendor_performance_Analysis.ipynb`

The first notebook imports the source CSVs. The second builds and stores `vendor_sales_summary`. The third reads that summary table and performs the analysis.

## Important Implementation Notes

The following points are visible in the supplied code and should be considered before relying on the outputs for business decisions.

1. **The CSV files are not included in the supplied ZIP.** You need the original data files and a working MySQL instance to reproduce the notebook outputs.
2. **Database credentials are hard-coded in the original notebooks.** Replace them with environment-based configuration and rotate any credential that has been exposed in a shared copy.
3. **The ingestion step replaces tables.** `create_table_schema()` calls `to_sql(..., if_exists="replace")`; rerunning ingestion recreates the imported tables before loading the CSVs again.
4. **The purchase-price join is inconsistent between notebooks.** In `exploratory_data_analysis.ipynb`, the purchase summary joins `purchases` to `purchase_prices` using both `Brand` and `VendorNumber`. In the final CTE query, the join uses only `Brand`. If a brand appears for multiple vendors in `purchase_prices`, this can create incorrect matches or duplicate purchase aggregation. Validate and, where appropriate, use both keys consistently.
5. **Freight is summarized at vendor level and repeated on brand rows.** `FreightSummary` groups by `VendorNumber` and is then joined to each vendor–brand row. Summing `FreightCost` across the summary table can therefore count the same vendor freight multiple times. The notebook also does not subtract freight from `GrossProfit`.
6. **The profit metrics are simplified.** `GrossProfit` is sales dollars minus purchase dollars; it is not a fully costed accounting profit measure. `ProfitMargin` is undefined for zero-sales rows and can be misleading for very small revenue values.
7. **The stock and unsold-inventory calculations are proxies.** They use quantities purchased and sold over the dataset period. They do not incorporate the supplied beginning/ending inventory tables, stock adjustments, purchase/sales timing, or a full inventory valuation method. The quantity difference can also be negative when sales exceed purchases.
8. **The saved notebook has a bulk-price note that conflicts with its output.** The calculated mean for the Large group is approximately **$11.31**, while a nearby markdown cell states **$10.78**. This README reports the actual displayed calculation output.
9. **The confidence-interval groups are vendor–brand records, not unique vendors.** Multiple rows can belong to the same vendor, so the intervals should be interpreted with that unit of analysis in mind.

## Limitations and Next Steps

- Validate the `purchases`–`purchase_prices` join keys and check row counts before and after joins.
- Allocate freight to brands using a documented rule (for example, purchase value, quantity, or weight) before calculating freight-adjusted profitability.
- Incorporate beginning and ending inventory if the goal is to calculate inventory on hand or conventional inventory turnover.
- Add date-aware analysis to distinguish current stock from products sold or purchased at different points in time.
- Revisit zero-sales, negative-profit, and extreme-price records instead of automatically treating every such record as an error.
- Use vendor-level aggregation when making claims about unique vendors, and account for repeated vendor observations in statistical testing.
- Add a dependency file and a sanitized configuration template to make the project easier to reproduce.

---

**Scope note:** This README documents the three notebooks and their saved outputs in the supplied archive. It does not claim that a dashboard, predictive model, or automated reporting application is part of this project.
