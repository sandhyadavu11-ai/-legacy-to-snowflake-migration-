# Legacy-to-Snowflake Data Migration

![Status](https://img.shields.io/badge/status-production--ready-brightgreen)
![Python](https://img.shields.io/badge/python-3.8%2B-blue)
![Snowflake](https://img.shields.io/badge/snowflake-ready-blue)

Automated, production-grade data migration from a legacy CSV-based system to Snowflake, with comprehensive data validation, reconciliation, and audit trails.

---

## 📋 Project Overview

This project demonstrates an end-to-end data migration pipeline:

- **Extract** legacy data from CSV files
- **Transform** with validation, quality checks, and business logic
- **Load** into Snowflake using star schema (dimensional modeling)
- **Validate** with reconciliation queries and data quality gates

### Key Features

✅ **Star Schema Design** — Dimension tables (CUSTOMER_DIM, PRODUCT_DIM) + Fact table (SALES_FACT)  
✅ **Data Validation** — Null checks, referential integrity, business logic validation  
✅ **Reconciliation** — Row count matching, data quality scoring  
✅ **Audit Trail** — Migration audit table tracks all runs and validation results  
✅ **Error Handling** — Detailed logging, duplicate detection, orphan key identification  
✅ **Performance** — Clustering, indexes, generated columns for query optimization  

---

## 📊 Data Model

### Star Schema

```
                    CUSTOMER_DIM
                    ============
                    customer_key (PK)
                    customer_id
                    customer_name
                    segment
                    region
                    annual_spend_usd
                           |
                           | (FK)
                           |
            SALES_FACT -----+
            ==========
            sales_key (PK)
            sale_id
            customer_key (FK) ---------> CUSTOMER_DIM
            product_key (FK) ---------> PRODUCT_DIM
            sale_date
            quantity
            unit_price_usd
            discount_pct
            status
            sales_amount (calculated)
            discount_amount (calculated)
                           |
                           | (FK)
                           |
                    PRODUCT_DIM
                    ===========
                    product_key (PK)
                    product_id
                    product_name
                    category
                    list_price_usd
                    cost_usd
```

### Tables

| Table | Rows | Purpose |
|-------|------|---------|
| CUSTOMER_DIM | 50 | Customer master (segment, region, spend) |
| PRODUCT_DIM | 25 | Product master (category, pricing) |
| SALES_FACT | 500 | Transaction-level sales data |

---

## 🚀 Quick Start

### 1. Prerequisites

```bash
Python 3.8+
Snowflake account (free trial available at https://signup.snowflake.com)
pip (Python package manager)
```

### 2. Clone the Repository

```bash
git clone https://github.com/sandhyadavu11-ai/legacy-to-snowflake-migration.git
cd legacy-to-snowflake-migration
```

### 3. Install Dependencies

```bash
pip install -r python/requirements.txt --break-system-packages
```

### 4. Generate Legacy Data

```bash
cd data/legacy
python gen_data.py
```

**Output:**
```
✅ Generated 50 customers
✅ Generated 25 products
✅ Generated 500 sales transactions
📊 Legacy Data Summary:
   Customers: 50
   Products: 25
   Sales Transactions: 500
```

### 5. Set Up Snowflake

#### Step A: Create Database & Schema
```sql
-- In Snowflake SQL Editor
CREATE DATABASE legacy_migration;
CREATE SCHEMA legacy_migration.public;
```

#### Step B: Create Tables
```bash
# Run the DDL script in Snowflake
# Copy contents of sql/snowflake_ddl.sql into Snowflake worksheet
```

### 6. Configure Snowflake Connection

Edit `python/migrate.py` and update these credentials:
```python
SNOWFLAKE_USER = 'your_email@company.com'
SNOWFLAKE_PASSWORD = 'your_password'
SNOWFLAKE_ACCOUNT = 'xy12345.us-east-1'  # From Snowflake account URL
SNOWFLAKE_DATABASE = 'legacy_migration'
SNOWFLAKE_SCHEMA = 'public'
SNOWFLAKE_WAREHOUSE = 'compute_wh'
```

**To find your Snowflake account identifier:**
1. Log into Snowflake
2. Click on account name (top-right) → Copy account identifier

### 7. Run the Migration

```bash
cd python
python migrate.py
```

**Expected Output:**
```
╔══════════════════════════════════════════════════════════════════════════════╗
║                  LEGACY-TO-SNOWFLAKE DATA MIGRATION                          ║
╚══════════════════════════════════════════════════════════════════════════════╝

================================================================================
STEP 1: EXTRACT - Reading legacy CSV files
================================================================================
✅ Extracted 50 customers
✅ Extracted 25 products
✅ Extracted 500 sales transactions

================================================================================
STEP 2: TRANSFORM - Validating and preparing data
================================================================================
Validating customer data...
✅ Customer validation complete (50 records)
Validating product data...
✅ Product validation complete (25 records)
Validating sales data...
✅ Sales validation complete (500 records)

...

╔══════════════════════════════════════════════════════════════════════════════╗
║                         ✅ MIGRATION COMPLETE                                ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

### 8. Validate Migration in Snowflake

Run the validation queries from `sql/validation_queries.sql`:

```sql
-- Row count validation
SELECT 'CUSTOMER_DIM' AS table_name, COUNT(*) AS row_count FROM CUSTOMER_DIM;
SELECT 'PRODUCT_DIM' AS table_name, COUNT(*) AS row_count FROM PRODUCT_DIM;
SELECT 'SALES_FACT' AS table_name, COUNT(*) AS row_count FROM SALES_FACT;

-- Data quality checks
SELECT * FROM CUSTOMER_DIM LIMIT 5;
SELECT * FROM SALES_FACT LIMIT 5;

-- Revenue summary
SELECT SUM(sales_amount) AS total_revenue_usd FROM SALES_FACT;
```

---

## 📁 Project Structure

```
legacy-to-snowflake-migration/
├── README.md                      # This file
├── .gitignore                     # Git ignore rules
├── data/
│   └── legacy/
│       ├── gen_data.py           # Generate sample CSV files
│       ├── customer_master.csv   # Sample customer data (50 rows)
│       ├── product_master.csv    # Sample product data (25 rows)
│       └── sales_ledger.csv      # Sample sales data (500 rows)
├── sql/
│   ├── snowflake_ddl.sql         # Create database objects
│   └── validation_queries.sql    # Data quality & reconciliation queries
├── python/
│   ├── migrate.py                # Main ETL script
│   ├── requirements.txt           # Python dependencies
│   └── *.log                      # Migration logs (generated)
└── docs/
    ├── architecture.md            # Data flow diagram
    ├── data_dictionary.md         # Field-level documentation
    └── troubleshooting.md         # Common issues & solutions
```

---

## 🔄 ETL Pipeline Details

### Step 1: EXTRACT
- **Source:** CSV files in `/data/legacy/`
- **Process:** Read using Pandas `read_csv()`
- **Output:** 3 DataFrames (customers, products, sales)

### Step 2: TRANSFORM
- **Validation:**
  - Null checks on required fields
  - Duplicate key detection
  - Value range validation (e.g., discount 0-1)
  - Referential integrity (foreign key existence)
  - Business logic checks (valid status, segment values)

- **Transformation:**
  - Convert sale_date to proper datetime format
  - Calculate derived metrics (discount_amount, sales_amount)
  - Add source system tracking columns

### Step 3: LOAD
- **Target:** Snowflake CUSTOMER_DIM, PRODUCT_DIM, SALES_FACT
- **Method:** 
  - Staging tables for intermediate storage
  - Snowflake `COPY INTO` or `write_pandas()` for bulk load
  - Insert dimension rows first, then fact rows
  - Enforce referential integrity via foreign keys

### Step 4: VALIDATE
- **Reconciliation:**
  - Source row count = Target row count
  - No orphaned keys
  - No duplicates in dimension keys
  - Data quality metrics logged

- **Output:**
  - MIGRATION_AUDIT table with results
  - Migration log file with timestamp

---

## 🛡️ Data Validation Checks

### Implemented Checks

| Check | Purpose |
|-------|---------|
| **Row Count Reconciliation** | Source rows match target rows exactly |
| **Null Detection** | No nulls in required fields (customer_id, product_id, etc.) |
| **Duplicate Keys** | customer_id, product_id, sale_id are unique |
| **Referential Integrity** | All foreign keys exist in parent tables |
| **Value Range** | discount_pct in [0, 1], quantity > 0, prices > 0 |
| **Business Logic** | status in ['Completed', 'Pending', 'Cancelled'] |
| **Data Distribution** | Segment, region, and status breakdowns logged |
| **Orphan Detection** | No sales with non-existent customer or product keys |

### Validation Queries

All validation SQL is in `sql/validation_queries.sql`. Key queries:

```sql
-- Check row counts
SELECT COUNT(*) FROM CUSTOMER_DIM;  -- Expected: 50
SELECT COUNT(*) FROM PRODUCT_DIM;   -- Expected: 25
SELECT COUNT(*) FROM SALES_FACT;    -- Expected: 500

-- Check orphaned keys
SELECT COUNT(*) FROM SALES_FACT sf
WHERE NOT EXISTS (SELECT 1 FROM CUSTOMER_DIM cd WHERE sf.customer_key = cd.customer_key);
-- Expected: 0

-- Revenue by region
SELECT cd.region, SUM(sf.sales_amount) AS total_revenue
FROM SALES_FACT sf
INNER JOIN CUSTOMER_DIM cd ON sf.customer_key = cd.customer_key
GROUP BY cd.region;
```

---

## 📊 Sample Queries for Analysis

### Total Sales Revenue
```sql
SELECT 
    ROUND(SUM(sales_amount), 2) AS total_revenue_usd,
    COUNT(DISTINCT sale_id) AS transaction_count,
    COUNT(DISTINCT customer_key) AS unique_customers
FROM SALES_FACT
WHERE status = 'Completed';
```

### Sales by Customer Segment
```sql
SELECT 
    cd.segment,
    COUNT(sf.sale_id) AS transaction_count,
    ROUND(SUM(sf.sales_amount), 2) AS total_revenue,
    ROUND(AVG(sf.sales_amount), 2) AS avg_transaction_value
FROM SALES_FACT sf
INNER JOIN CUSTOMER_DIM cd ON sf.customer_key = cd.customer_key
GROUP BY cd.segment
ORDER BY total_revenue DESC;
```

### Top 10 Customers by Revenue
```sql
SELECT 
    cd.customer_id,
    cd.customer_name,
    cd.segment,
    COUNT(sf.sale_id) AS transactions,
    ROUND(SUM(sf.sales_amount), 2) AS total_spent_usd
FROM SALES_FACT sf
INNER JOIN CUSTOMER_DIM cd ON sf.customer_key = cd.customer_key
GROUP BY cd.customer_id, cd.customer_name, cd.segment
ORDER BY total_spent_usd DESC
LIMIT 10;
```

### Monthly Sales Trend
```sql
SELECT 
    DATE_TRUNC('month', sale_date) AS month,
    COUNT(sale_id) AS transaction_count,
    ROUND(SUM(sales_amount), 2) AS monthly_revenue
FROM SALES_FACT
WHERE status = 'Completed'
GROUP BY DATE_TRUNC('month', sale_date)
ORDER BY month;
```

---

## 🔧 Troubleshooting

### Issue: "Could not connect to Snowflake"
**Solution:** Verify credentials in `migrate.py`:
```python
# Check account ID format (should be: xy12345.us-east-1)
# Check username is email format
# Check password is correct
# Verify warehouse exists and you have access
```

### Issue: "File not found" error
**Solution:** Ensure you've generated data first:
```bash
cd data/legacy
python gen_data.py
cd ../..
```

### Issue: "Duplicate key" error
**Solution:** Tables may already have data. Truncate before re-running:
```sql
TRUNCATE TABLE CUSTOMER_DIM;
TRUNCATE TABLE PRODUCT_DIM;
TRUNCATE TABLE SALES_FACT;
```

### Issue: "Foreign key constraint violation"
**Solution:** Ensure dimension tables are loaded before fact table. Run DDL in order:
1. Create CUSTOMER_DIM
2. Create PRODUCT_DIM
3. Create SALES_FACT (with references to dimensions)

---

## 📈 Migration Performance

| Metric | Result |
|--------|--------|
| Data Extraction | < 1 sec (500 rows) |
| Validation | < 2 sec (500 rows) |
| Transformation | < 1 sec |
| Snowflake Load | < 5 sec (bulk insert) |
| Total Time | ~10 sec |

---

## 🎯 What This Project Demonstrates

### Skills Showcased

✅ **Snowflake Development**
- Star schema design
- Dimensional modeling (CUSTOMER_DIM, PRODUCT_DIM, SALES_FACT)
- SQL (CTEs, window functions, generated columns)
- Data governance (clustering, indexes, foreign keys)

✅ **ETL/ELT Development**
- Data extraction from CSV (legacy format)
- Transformation with business logic
- Data validation and quality checks
- Reconciliation logic

✅ **Data Warehousing Concepts**
- Fact and dimension tables
- Surrogate keys
- Data lineage and audit trails
- Incremental vs. full loads

✅ **Data Engineering**
- Python for data orchestration
- Pandas for data manipulation
- Error handling and logging
- Migration planning and execution

✅ **Financial Data Experience**
- Budgets vs. actuals (sales revenue)
- Cost tracking (unit price, discounts)
- Multi-dimensional analysis (by customer, region, product)
- Data reconciliation

✅ **Data Migration & Platform Modernization**
- Legacy system to Snowflake
- Source-to-target mappings
- Data validation post-migration
- Audit trail and documentation

---

## 📚 Additional Resources

- [Snowflake Documentation](https://docs.snowflake.com/)
- [Snowflake Star Schema Best Practices](https://docs.snowflake.com/en/user-guide/data-sharing-intro.html)
- [Data Warehousing Fundamentals](https://en.wikipedia.org/wiki/Data_warehouse)
- [SnowPro Core Certification](https://www.snowflake.com/certifications/)

---

## 🤝 Contributing

To improve this project:
1. Test with your own Snowflake account
2. Add advanced validation rules
3. Implement incremental load logic
4. Add Apache Airflow integration
5. Create additional transformation examples

---

## 📄 License

This project is open-source and available under the MIT License.

---

## ✅ Validation Checklist

Before submitting this portfolio project:

- [ ] Clone repo and verify all files present
- [ ] Run `python data/legacy/gen_data.py` — 3 CSV files generated
- [ ] Copy DDL from `sql/snowflake_ddl.sql` into Snowflake
- [ ] Update credentials in `python/migrate.py`
- [ ] Run `python python/migrate.py` — completes without errors
- [ ] Run validation queries in Snowflake — all pass
- [ ] Review generated migration logs
- [ ] README is comprehensive and clear

---

## 📞 Support

For questions or issues, refer to:
- `docs/troubleshooting.md` — Common issues
- Migration logs (`migration_*.log`) — Detailed error messages
- Snowflake support — Connection or permission issues

---

**Built by:** S Deepthi Davu   
**Date:** 2026-09  
**Status:** Production-ready

