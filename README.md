# SQL Data Warehouse Project

A full end-to-end data warehousing solution built on SQL Server, implementing a **Medallion Architecture** (Bronze → Silver → Gold) to ingest, cleanse, transform, and model data from CRM and ERP source systems into an analytics-ready Star Schema.

---

## Architecture Overview

```
Source Systems          Bronze Layer         Silver Layer          Gold Layer
─────────────────       ────────────         ────────────          ──────────
CRM (CSV Files)   →     Raw Ingestion   →    Cleansed &      →    Star Schema
ERP (CSV Files)         (No transforms)      Standardized          (Views)
                                             Data
```

The project follows the **Medallion Architecture** pattern:

| Layer | Purpose |
|---|---|
| **Bronze** | Raw data ingested as-is from CSV files via `BULK INSERT`. No transformations applied. |
| **Silver** | Cleansed, standardized, and deduplicated data. Business rules applied. |
| **Gold** | Analytics-ready Star Schema views — Dimension and Fact tables for reporting. |

---

## Data Sources

| Source | Tables | Description |
|---|---|---|
| **CRM** | `cust_info`, `prd_info`, `sales_details` | Customer, product, and sales transaction data |
| **ERP** | `CUST_AZ12`, `LOC_A101`, `PX_CAT_G1V2` | Customer demographics, locations, and product categories |

---

## Gold Layer — Star Schema

The Gold layer implements a **Star Schema** optimized for analytical queries:

```
                    ┌─────────────────┐
                    │  dim_customers  │
                    │─────────────────│
                    │ customer_key PK │
                    │ customer_id     │
                    │ customer_number │
                    │ first_name      │
                    │ last_name       │
                    │ country         │
                    │ marital_status  │
                    │ gender          │
                    │ birthdate       │
                    └────────┬────────┘
                             │
                             │
┌─────────────────┐   ┌──────┴──────────┐
│  dim_products   │   │   fact_sales    │
│─────────────────│   │─────────────────│
│ product_key PK  ├───│ order_number    │
│ product_id      │   │ product_key FK  │
│ product_number  │   │ customer_key FK │
│ product_name    │   │ order_date      │
│ category        │   │ shipping_date   │
│ subcategory     │   │ due_date        │
│ product_line    │   │ sales_amount    │
│ cost            │   │ quantity        │
└─────────────────┘   │ price           │
                      └─────────────────┘
```

---

## Silver Layer — Data Transformations

Key transformations applied during Bronze → Silver:

| Table | Transformations |
|---|---|
| `crm_cust_info` | Deduplication via `ROW_NUMBER()`, whitespace trimming, gender and marital status normalization |
| `crm_prd_info` | Category ID extraction from product key, product line code mapping, end date derivation using `LEAD()` |
| `crm_sales_details` | Integer-to-date conversion, sales/price/quantity consistency validation and recalculation |
| `erp_cust_az12` | NAS prefix removal from customer IDs, future birthdate nullification, gender normalization |
| `erp_loc_a101` | Hyphen removal from IDs, country code standardization (e.g. `US`/`USA` → `United States`) |
| `erp_px_cat_g1v2` | Loaded as-is, whitespace validated |

---

## Project Structure

```
sql-datawharehouse-project/
│
├── datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   └── source_erp/
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── scripts/
│   ├── init_database.sql          -- Creates DataWarehouse DB and schemas
│   ├── bronze/
│   │   ├── ddl_bronze.sql         -- Bronze table definitions
│   │   └── proc_load_bronze.sql   -- Stored procedure: CSV → Bronze
│   ├── silver/
│   │   ├── ddl_silver.sql         -- Silver table definitions
│   │   └── proc_load_silver.sql   -- Stored procedure: Bronze → Silver (ETL)
│   └── gold/
│       └── ddl_gold.sql           -- Gold views: Star Schema
│
├── tests/
│   └── quality_checks_silver.sql  -- Data quality validation queries
│
└── docs/
    ├── Architecture Diagram.png
    ├── Data_Flow_Diagram.png
    └── Integration_Model_Diagram.png
```

---

## How to Run

> **Prerequisites:** SQL Server running in Docker with datasets copied to `/datasets/`

**Step 1 — Initialize the database:**
```sql
-- Run in SQL Server
scripts/init_database.sql
```

**Step 2 — Create Bronze tables:**
```sql
scripts/bronze/ddl_bronze.sql
```

**Step 3 — Load Bronze layer (CSV ingestion):**
```sql
EXEC bronze.load_bronze;
```

**Step 4 — Create Silver tables:**
```sql
scripts/silver/ddl_silver.sql
```

**Step 5 — Load Silver layer (ETL):**
```sql
EXEC silver.load_silver;
```

**Step 6 — Create Gold layer (Star Schema views):**
```sql
scripts/gold/ddl_gold.sql
```

**Step 7 — Run quality checks:**
```sql
tests/quality_checks_silver.sql
```

---

## Quality Checks

The `quality_checks_silver.sql` script validates:

- No NULL or duplicate primary keys
- No unwanted leading/trailing spaces in string columns
- Data standardization consistency (gender, marital status, country, product line)
- Valid date ranges (no future birthdates, no invalid order/ship/due date sequences)
- Sales consistency: `sales_amount = quantity × price`

---

## Technologies Used

- **SQL Server** (via Docker)
- **T-SQL** — Stored procedures, CTEs, window functions, BULK INSERT
- **Medallion Architecture** — Bronze / Silver / Gold layering
- **Star Schema** — Fact and Dimension modeling for analytics
- **draw.io** — Architecture, Data Flow, and Integration Model diagrams
