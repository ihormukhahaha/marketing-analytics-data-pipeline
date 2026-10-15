# Marketing & E-Commerce Data Pipeline for MMM & ROAS Analytics

An end-to-end data preprocessing, cleaning, and analytics pipeline designed to process noisy mobile-analytics and advertising campaign data. Built as a foundational data layer for **Marketing Mix Modeling (MMM)**, attribution analysis, and return-on-ad-spend (ROAS) optimization.

---

## Project Overview
In modern e-commerce and multi-channel marketing, raw event streams and campaign cost data are often messy, duplicated, and prone to late arrivals or retroactive revenue corrections. 
This project ingests raw mobile event logs and advertising expenditures, handles edge cases (such as corrections, currency/format inconsistencies, and test traffic filtering), and outputs clean, partitioned analytical datasets ready for statistical modeling and machine learning.

---

## Tech Stack & Skills Demonstrated
- **Languages:** Python (Pandas, NumPy), SQL (PostgreSQL, Window Functions, CTEs)
- **Data Processing & Storage:** Parquet (partitioned by date), CSV, Atomic File I/O
- **Data Engineering Concepts:** Idempotency, incremental loading, deduplication strategies, quarantine pattern for malformed data
- **Marketing Analytics:** ROAS calculation, campaign performance aggregation, ad-spend vs. revenue matching

---

## Repository Structure
```text
├── python/
│   ├── pipeline/
│   │   ├── __init__.py
│   │   └── run.py          # CLI-driven incremental ETL pipeline with crash safety
│   ├── cleaner.py          # Core data cleaning, anomaly handling, and quarantine logic
│   └── task_3_3_edired.py  # Optimized daily revenue aggregation script
├── sql/
│   ├── init_db.sql         # Database schema setup and raw data ingestion
│   ├── task_2_1.sql        # Advanced event de-duplication and cleaning view
│   ├── task_2_2.sql        # Time-series window functions (running totals, 7d moving average, DoD %)
│   ├── task_2_3.sql        # Full outer join of revenue and advertising costs for ROAS
│   ├── task_2_4.sql        # Safe incremental loading with ON CONFLICT upserts
│   └── task_2_5.sql        # Automated data quality & integrity checks
├── ANSWERS.md              # Detailed written architectural decisions and trade-offs
└── ASSUMPTIONS.md          # Business and technical assumptions made during development