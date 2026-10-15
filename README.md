# Marketing & E-Commerce Data Pipeline for MMM & ROAS Analytics

An end-to-end data preprocessing, cleaning, and analytics pipeline designed to process noisy mobile-analytics and advertising campaign data. Built as a foundational data layer for **Marketing Mix Modeling (MMM)**, attribution analysis, and return-on-ad-spend (ROAS) optimization.

---

## 📌 Project Overview
In modern e-commerce and multi-channel marketing, raw event streams and campaign cost data are often messy, duplicated, and prone to late arrivals or retroactive revenue corrections. 
This project ingests raw mobile event logs and advertising expenditures, handles edge cases, and outputs clean, partitioned analytical datasets ready for statistical modeling and machine learning.

---

## 🛠️ Tech Stack & Skills Demonstrated
- **Languages:** Python (Pandas, NumPy), SQL (PostgreSQL, Window Functions, CTEs)
- **Data Processing & Storage:** Parquet (partitioned by date), CSV, Atomic File I/O
- **Data Engineering Concepts:** Idempotency, incremental loading, deduplication strategies, quarantine pattern for malformed data
- **Marketing Analytics:** ROAS calculation, campaign performance aggregation, ad-spend vs. revenue matching

---

## 🏗️ Architecture & Engineering Decisions

### 1. Data Cleaning & Deduplication
- **Deterministic Deduplication:** To prevent random selection during timestamp ties, sorting combines `ingested_at DESC` and `revenue_usd DESC`.
- **Quarantine Pattern:** Malformed rows (unparseable revenue, missing timestamps) are isolated into a separate quarantine dataset rather than crashing the pipeline.

### 2. Analytical SQL & Time-Series Metrics
- **Window Functions:** Computes cumulative revenue since app launch, 7-day moving averages, and day-over-day percentage changes.
- **ROAS & Attribution:** Merges multi-channel campaign costs with event-level revenues using `FULL OUTER JOIN` and `COALESCE` to handle missing revenue or ghost costs safely.

### 3. Incremental Loading & Idempotency
- **Incremental Strategy:** Filters on `ingested_at` using a 7-day lookback window to safely capture late-arriving mobile events and retroactive revenue corrections.
- **Idempotency:** Utilizes PostgreSQL `ON CONFLICT DO UPDATE` upserts to ensure re-running the pipeline over overlapping periods does not create duplicates.

### 4. Dimensional Modeling (Star Schema & SCD)
- **Star Schema Design:** Structured around the `fact_events` table (grain: single unique deduplicated user event) linked with dimensions like `dim_app`, `dim_user`, `dim_campaign`, and `dim_date`.
- **Slowly Changing Dimensions (SCD Type 2):** Handled campaign renames mid-flight by creating new dimension records to preserve historical metrics accurately.

---

## 📂 Repository Structure
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
│   ├── task_2_2.sql        # Time-series window functions
│   ├── task_2_3.sql        # Full outer join of revenue and advertising costs for ROAS
│   ├── task_2_4.sql        # Safe incremental loading with ON CONFLICT upserts
│   └── task_2_5.sql        # Automated data quality & integrity checks