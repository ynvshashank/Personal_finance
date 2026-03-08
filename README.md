# Personal Finance

A personal finance project to track expenses, built for practicing and learning Azure Cloud (ADLS Gen2) for storage, Databricks for end-to-end data flow, and visualization.

---

## Version 1 — Personal Finance ETL Pipeline

### Overview

An end-to-end ETL pipeline following the **Medallion Architecture** (Bronze → Silver → Gold) to ingest, clean, and transform personal financial transaction data for analysis.

### Data Source

- **Format**: Excel (`.xlsx`)
- **Location**: Azure Data Lake Storage Gen2  
  `abfss://finance@mypersonalfinance09.dfs.core.windows.net/raw/`
- **Fields**: `transaction_id`, `date`, `Type` (Income / Expenses / Investment), `category`, `payment_mode`, `amount`, `Notes`

### Architecture

```
Raw (Excel) → Bronze (Delta) → Silver (Delta, deduplicated) → Gold (Delta, analytical views)
```

#### Bronze Layer
- Reads raw Excel file using `pandas` + `openpyxl`, converts to Spark DataFrame.
- Adds `ingestion_timestamp` column for audit tracking.
- Writes to Delta format at `/bronze/bronze_transactions`.

#### Silver Layer
- Deduplicates records using a window function (`ROW_NUMBER` partitioned by `transaction_id`, ordered by `ingestion_timestamp DESC`).
- Retains only the latest record per transaction.
- Writes to Delta format at `/silver/silver_transactions`.

#### Gold Layer
- Creates analytical views and persists them as Delta tables under `/gold/`:

| Gold Table | Description |
| --- | --- |
| `gold_monthly_summary` | Monthly totals for income, expenses, net cashflow, and transaction count |
| `gold_category_spending` | Total spending and transaction count grouped by category |
| `gold_payment_mode_spending` | Total spending and transaction count grouped by payment mode |
| `gold_daily_spending` | Daily spending totals |
| `gold_top_expenses` | Top 20 highest expense transactions |
| `gold_running_balance` | Cumulative running balance across all transactions (income, expenses, investments) |

### Storage Layout

```
abfss://finance@mypersonalfinance09.dfs.core.windows.net/
├── raw/
│   └── 2026-02/transactions.xlsx
├── bronze/
│   └── bronze_transactions/        (Delta)
├── silver/
│   └── silver_transactions/        (Delta)
└── gold/
    ├── gold_monthly_summary/       (Delta)
    ├── gold_category_spending/      (Delta)
    ├── gold_payment_mode_spending/  (Delta)
    ├── gold_daily_spending/         (Delta)
    ├── gold_top_expenses/           (Delta)
    └── gold_running_balance/        (Delta)
```

### Tech Stack

- **Cloud Storage**: Azure Data Lake Storage Gen2
- **Compute**: Databricks (PySpark + Spark SQL)
- **Data Format**: Delta Lake
- **Libraries**: `pandas`, `openpyxl`, `pyspark`

### Notebook

- `notebooks/Personal Finance ETL Pipeline` — contains the full pipeline from ingestion to gold table creation and visualization queries.
