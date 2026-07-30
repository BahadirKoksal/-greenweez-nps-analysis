# 🌿 Greenweez NPS Analysis — BigQuery SQL

## 🎯 Objective

Greenweez is a French organic e-commerce company. This project analyzes customer satisfaction (NPS) data from April to August 2021 to measure overall performance, track monthly trends, and identify the root cause of a score decline.

**Key Research Questions:**
- What is Greenweez's overall NPS score and how does it compare to industry benchmarks?
- Why did NPS decline between June and August 2021?
- Which transporter is most responsible for customer dissatisfaction?
- Which customer segment was most impacted, and what action should be taken?

---

## 📁 Dataset


**Main Table:** `gwz_nps`

| Column | Type | Description |
|---|---|---|
| `date_date` | DATE | Order date |
| `orders_id` | INTEGER | Unique order identifier |
| `global_note` | INTEGER | Overall satisfaction score (0–10) |
| `csat_website` | INTEGER | Website satisfaction score (1–5) |
| `csat_product` | INTEGER | Product satisfaction score (1–5) |
| `csat_price` | INTEGER | Price satisfaction score (1–5) |
| `csat_delivery` | INTEGER | Delivery satisfaction score (1–5) |
| `transporter` | STRING | Delivery company (Chrono Pickup, Chrono Home, DPD Pickup, DPD Home) |
| `sgt` | STRING | Customer segment (new, occasional, frequent) |

**Dataset size:** 6,295 raw rows → 6,294 after deduplication  
**Date range:** April 2021 – August 2021  
**NPS response rate:** 6,253 / 6,294 (99.3%) ✅

---

## 🗂️ Data Architecture

This project follows the **Bronze → Silver → Gold** layered data architecture pattern.

| Table | Layer | Description |
|---|---|---|
| `gwz_nps` | 🥉 Bronze | Raw data — never modified |
| `gwz_nps_deduplicated` | staging | Working copy for deduplication |
| `gwz_nps_clean` | 🥈 Silver | Cleaned, deduplicated data |
| `gwz_nps_calculated` | 🥇 Gold | Enriched with NPS category column — used for all analysis |

---

## 🔍 Analysis Steps

**1. Data Exploration**  
Explored table structure, column types, and row counts. Identified that `orders_id` is not a true primary key due to one duplicate entry (order `970720` appeared twice).

**2. Data Cleaning**  
Created a deduplicated copy using `SELECT DISTINCT *`. Validated the result: `total_rows = distinct_orders = 6,294`.

**3. NPS Score Calculation**  
Converted `global_note` (0–10) into NPS categories using `CASE WHEN`:

| global_note | NPS Value | Category |
|---|---|---|
| 9 – 10 | +1 | Promoter |
| 7 – 8 | 0 | Passive |
| 0 – 6 | -1 | Detractor |
| null | null | No response |

**4. Overall NPS**  
Counted promoters (4,524), detractors (459), and total responses (6,253).  
`NPS = ROUND((4524 - 459) / 6253 * 100, 1) = 65.0`

**5. Monthly Trend Analysis**  
Calculated NPS separately for June and August 2021 to identify trends.

**6. Transporter Breakdown**  
Drilled down by transporter to find which delivery company was responsible for the August decline.

**7. Action List**  
Identified the 5 most critical detractors (Chrono Home + August + low delivery score) sorted by customer value for the customer service team to follow up.

---

## 📊 Key Findings

| Metric | Value |
|---|---|
| Total orders analyzed | 6,294 |
| NPS response rate | 99.3% |
| Overall NPS (Apr–Aug 2021) | **65.0** ✅ |
| June 2021 NPS | **67.0** |
| August 2021 NPS | **62.9** ⚠️ |
| Chrono Home NPS — June | **62.1** |
| Chrono Home NPS — August | **35.1** 🚨 |

**Root cause:** Chrono Home's NPS dropped 27 points in a single month (June → August 2021), driven by poor delivery experiences — primarily among frequent (loyal) customers.

**Recommended action:** The customer service team should prioritize contacting the 5 highest-value detractors who experienced poor delivery via Chrono Home in August 2021.

---

## 🛠️ SQL Techniques Used

| Technique | Purpose |
|---|---|
| `SELECT DISTINCT` | Duplicate detection and removal |
| `CREATE OR REPLACE TABLE ... AS SELECT` | Safe working copy creation without modifying raw data |
| `CASE WHEN` | Converting raw scores (0–10) into NPS categories |
| `WHERE ... IS NOT NULL` | Excluding null responses from NPS calculations |
| `BETWEEN` | Monthly date range filtering for trend analysis |
| `COUNT(*) vs COUNT(DISTINCT ...)` | Data quality validation — primary key check |
| `ROUND()` | Clean score formatting for reporting |
| `ORDER BY ... DESC` | Prioritizing action list by customer segment value |

---

## 🔧 Tools & Environment

- **Google BigQuery** — SQL query execution and table management
- **Standard SQL** — All queries written in BigQuery-compatible SQL
- **Google Cloud Console** — Dataset and project management

---

