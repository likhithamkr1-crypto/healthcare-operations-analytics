# 🏥 Healthcare Operations & Patient Flow Analytics

> An advanced data analytics pipeline utilizing complex SQL Common Table Expressions (CTEs) and Window Functions to analyze hospital patient flow, department turnaround times, and resource bottlenecks.

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/)
[![Advanced SQL / DuckDB](https://img.shields.io/badge/SQL-CTEs%20%26%20Window%20Functions-orange.svg)](https://duckdb.org/)
[![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-brightgreen.svg)](https://pandas.pydata.org/)

---

## 📊 Executive Summary & Business Objective
Hospital emergency rooms and specialized departments often struggle with unpredictable patient surges, leading to extended triage wait times, operational bottlenecks, and bed capacity strain.
* **The Goal:** Analyze historical admission telemetry and patient stay durations across departments to isolate staffing bottlenecks and high-turnover pinch points.
* **The Impact:** Empowers hospital administrators to optimize shift allocations, reduce emergency wait times, and improve overall operational throughput.

---

## 🏗️ Technical Architecture & Workflow
1. **Data Extraction & Datetime Parsing:** Processed patient admission and discharge logs in Python using Pandas.
2. **Advanced SQL Querying (CTEs):** Built multi-step relational transformations via `DuckDB` to calculate precise stay durations in hours.
3. **Analytical Window Functions:** Utilized `DENSE_RANK()` to rank departments dynamically by patient volume and severity scores without losing row-level context.

---

## 💻 Code Snippet: Advanced SQL CTEs & Window Functions
```sql
WITH PatientStay AS (
    SELECT 
        patient_id,
        department_name,
        severity_score,
        ROUND(EXTRACT(EPOCH FROM (discharge_time - triage_time)) / 3600.0, 2) AS stay_duration_hours
    FROM df_admissions
),
DepartmentMetrics AS (
    SELECT 
        department_name,
        COUNT(patient_id) AS total_patients,
        ROUND(AVG(stay_duration_hours), 2) AS avg_stay_hours,
        MAX(stay_duration_hours) AS max_stay_hours,
        -- Window Function: Rank departments by patient volume
        DENSE_RANK() OVER (ORDER BY COUNT(patient_id) DESC) as volume_rank
    FROM PatientStay
    GROUP BY department_name
)
SELECT * FROM DepartmentMetrics
ORDER BY volume_rank;
