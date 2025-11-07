[README (1).md](https://github.com/user-attachments/files/23427178/README.1.md)
# HR Data Migration Pipeline: PeopleSoft to Redshift

An Apache Airflow learning project demonstrating data pipeline orchestration using Medallion Architecture for HR data migration.

## 📋 Project Overview

This pipeline orchestrates a large-scale HR data migration that extracts employee, department, and job code data from PeopleSoft, applies multi-layer transformations, and loads analytics-ready tables into Amazon Redshift.

**Key Metrics:**
- **Data Volume:** 10 years of historical HR data (~100K employee records)
- **Architecture:** 3-layer Medallion (Bronze → Silver → Gold)
- **Total Tasks:** 17 tasks across 8 phases
- **Execution Time:** ~4 hours

**Tech Stack:** Apache Airflow, AWS S3, Amazon Redshift, Parquet, Tableau

## 🏗️ Medallion Architecture

```
PeopleSoft → Bronze Layer → Silver Layer → Gold Layer → Tableau
              (Raw Data)   (Cleaned)      (Analytics)    (BI)
```

**Bronze Layer (Raw Data)**
- Direct extract from source systems
- No transformations applied
- Stored in S3 as Parquet files
- Preserves data lineage

**Silver Layer (Cleaned & Standardized)**
- Standardized column names and data types
- Null handling and deduplication
- Business rule validation
- Indexed for performance

**Gold Layer (Business-Ready Analytics)**
- Pre-aggregated metrics and KPIs
- Department-level summaries
- Optimized for BI tools

## 🔄 Pipeline Workflow

### Phase 1: Extract (3 parallel tasks)
```
extract_employees (100K records)
extract_departments (50 depts)
extract_job_codes (250 codes)
    ↓
Output: Parquet files in S3
```

### Phase 2: Validate Bronze (1 task)
```
validate_bronze
    ↓
Checks: nulls, duplicates, date logic, salary ranges
```

### Phase 3: Load Bronze to Redshift (3 parallel tasks)
```
load_bronze_employees
load_bronze_departments
load_bronze_jobs
    ↓
Output: bronze.raw_* tables
```

### Phase 4: Create Silver Layer (3 parallel tasks)
```
create_silver_employees (clean & standardize)
create_silver_departments (dedupe)
create_silver_jobs (filter active)
    ↓
Output: silver.* tables with indexes
```

### Phase 5: Create Gold Layer (2 parallel tasks)
```
create_gold_employee_summary (tenure, status)
create_gold_department_metrics (headcount, salary, turnover)
    ↓
Output: gold.* aggregated tables
```

### Phase 6: Validate Gold (1 task)
```
validate_gold
    ↓
Check: record count reconciliation across layers
```

### Phase 7: Tableau Integration (2 sequential tasks)
```
create_tableau_views (materialize views)
    ↓
refresh_tableau (update dashboards)
```

### Phase 8: Reporting (2 sequential tasks)
```
generate_report (collect XCom stats)
    ↓
send_notification (email to HR team)
```

## 🎓 Key Learning Points

**Airflow Concepts Demonstrated:**
- ✅ Parallel task execution using lists in `chain()`
- ✅ XCom for passing data between tasks
- ✅ Multiple operators: Python, Postgres, S3ToRedshift, Bash, Email
- ✅ Error handling with retry logic and timeouts
- ✅ Data quality validation checkpoints
- ✅ Production-ready DAG structure

**Medallion Architecture Benefits:**
- Bronze = immutable raw data for audit trail
- Silver = cleaned data for analytics
- Gold = business-specific aggregations for BI

---

**Created by:** Wendy | Learning Project
