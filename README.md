# Marketing & E-Commerce Data Pipeline for MMM & ROAS Analytics

An end-to-end pipeline for preprocessing, cleaning, and analyzing noisy mobile-analytics and advertising campaign data. It serves as the foundational data layer for **Marketing Mix Modeling (MMM)**, attribution analysis, and return-on-ad-spend (ROAS) optimization.

Raw event streams and campaign cost data in multi-channel marketing are often messy, duplicated, and subject to late arrivals or retroactive revenue corrections. The pipeline ingests raw mobile event logs and advertising expenditures, handles these edge cases, and outputs clean, partitioned analytical datasets ready for statistical modeling and machine learning.

---

## Project Architecture

The project has a modular structure:

* **`python/`** — the Python part of the pipeline:
  * **`pipeline/run.py`** — a CLI-driven incremental ETL pipeline with crash safety.
  * **`cleaner.py`** — core data cleaning, anomaly handling, and quarantine logic.
  * **`task_3_3_edired.py`** — an optimized daily revenue aggregation script.
* **`sql/`** — the analytical SQL layer:
  * **`init_db.sql`** — database schema setup and raw data ingestion.
  * **`task_2_1.sql`** — advanced event deduplication and cleaning view.
  * **`task_2_2.sql`** — time-series window functions.
  * **`task_2_3.sql`** — full outer join of revenue and advertising costs for ROAS.
  * **`task_2_4.sql`** — safe incremental loading with `ON CONFLICT` upserts.
  * **`task_2_5.sql`** — automated data quality and integrity checks.

---

## Pipeline Stages

### 1. Data Cleaning and Deduplication
* **Deterministic deduplication:** to avoid random selection on timestamp ties, records are sorted by `ingested_at DESC` and `revenue_usd DESC`.
* **Quarantine pattern:** malformed rows (unparseable revenue, missing timestamps) are isolated into a separate quarantine dataset instead of crashing the pipeline.

### 2. Analytical SQL and Time-Series Metrics
* **Window functions:** cumulative revenue since app launch, 7-day moving averages, and day-over-day percentage changes.
* **ROAS and attribution:** multi-channel campaign costs are merged with event-level revenue using `FULL OUTER JOIN` and `COALESCE`, which safely handles missing revenue and ghost costs.

### 3. Incremental Loading and Idempotency
* **Incremental strategy:** filtering on `ingested_at` with a 7-day lookback window captures late-arriving mobile events and retroactive revenue corrections.
* **Idempotency:** PostgreSQL `ON CONFLICT DO UPDATE` upserts guarantee that re-running the pipeline over overlapping periods creates no duplicates.

### 4. Dimensional Modeling
* **Star schema:** built around the `fact_events` table (grain: one unique deduplicated user event) and the dimensions `dim_app`, `dim_user`, `dim_campaign`, and `dim_date`.
* **Slowly Changing Dimensions (SCD Type 2):** campaign renames mid-flight create new dimension records, which preserves historical metrics accurately.

### 5. Storage
* **Output formats:** cleaned data is stored as Parquet partitioned by date, with atomic file I/O for safe writes.
