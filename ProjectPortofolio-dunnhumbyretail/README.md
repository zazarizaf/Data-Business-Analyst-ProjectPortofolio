# 🛒 Dunnhumby Retail Analysis

> Decoding 117 weeks of retail transactions — from seasonal spikes to new customer acquisition.

---

## 📌 Table of Contents

- [Overview](#-overview)
- [Business Questions](#-business-questions)
- [Dataset](#-dataset)
- [Tools](#-tools)
- [Data Cleaning & Transformation](#-data-cleaning--transformation)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [Files](#-files)
- [Author](#-author)

---

## 📖 Overview

This project analyzes **31 million retail transactions** from the Dunnhumby dataset, covering **117 weeks** of shopping activity between **April 2006 and July 2008**. The goal is to uncover behavioral patterns across time, product categories, and customer segments — and translate them into actionable business recommendations.

The analysis is split into two parts:

1. **Data Cleaning & Transformation** — performed in `DataAnalysisProject_dunnhumby_retail.ipynb`
2. **Business Insight Report** — summarized in `Decoding Retail Transactions.pdf`

---

## 🎯 Business Questions

1. How does shopping behavior vary across the year, especially around seasonal peaks?
2. What are the peak hours for purchasing, and how does product mix shift throughout the day?
3. How does the balance between **customer retention** and **acquisition** evolve over time?

---

## 📂 Dataset

| Info | Detail |
|------|--------|
| Source | Dunnhumby Retail Dataset |
| Rows | 31,057,860 (after cleaning) |
| Columns | 23 (after cleaning) |
| Period | April 2006 – July 2008 (117 weeks) |
| Raw Format | 118 CSV files (one per week) |
| Fields | Transaction, basket, store, customer segment, product hierarchy |

Key columns include:
- `SHOP_DATE`, `SHOP_WEEK`, `SHOP_WEEKDAY`, `SHOP_HOUR`
- `PROD_CODE` and its hierarchy (`PROD_CODE_10` to `PROD_CODE_40`)
- `CUST_CODE`, `seg_1`, `seg_2`
- `BASKET_ID`, `BASKET_SIZE`, `BASKET_TYPE`, `BASKET_DOMINANT_MISSION`
- `STORE_CODE`, `STORE_FORMAT`, `STORE_REGION`
- `SPEND`, `QUANTITY`

---

## 🛠 Tools

- **Python 3.13**
- **Pandas**, **NumPy** — data manipulation
- **PyArrow** — Parquet storage
- **Jupyter Notebook** — analysis environment
- **Matplotlib**, **Seaborn** — visualization

---

## 🧹 Data Cleaning & Transformation

All cleaning steps were performed in `DataAnalysisProject_dunnhumby_retail.ipynb`. Below is a summary of what was done.

### 1. Data Loading & Consolidation
- Retrieved **118 CSV files** automatically using `glob.glob()`.
- Looped `pd.read_csv()` over each file and concatenated them with `pd.concat()`.
- **Result:** 31,057,875 rows × 24 columns (8.2 GB memory).

### 2. Storage Optimization
- Converted the dataset to **Parquet** with `snappy` compression for faster downstream loading.
- Re-loaded the file using `pd.read_parquet()`.

### 3. Missing Values — `SPEND`
- `SPEND` and `SFNND` are complementary columns (never populated at the same time — overlap = 0).
- Filled missing `SPEND` values with `SFNND`, then dropped `SFNND`.

### 4. Missing Values — `BASKET_TYPE`
- Same complementary pattern with `BASKET_TYFN`.
- Filled missing `BASKET_TYPE` values with `BASKET_TYFN`, then dropped `BASKET_TYFN`.
- **Result:** Columns reduced from **24 → 22**.

### 5. Type Conversion — `SHOP_WEEK` & `SHOP_DATE`
- `SHOP_WEEK` → `string` (identifier, not a number).
- `SHOP_DATE` → `datetime` (format `%Y%m%d`) for time-series analysis.

### 6. Duplicate Removal
- Detected **15 duplicate rows** and removed them via `drop_duplicates()`.
- **Result:** 31,057,875 → **31,057,860 rows**.

### 7. Type Conversion — `BASKET_ID`
- Converted from `float64` (scientific notation `9.941000e+14`) to `int64` (`994100000000000`).

### 8. Transformation — `SHOP_WEEKDAY`
- Mapped numeric values `1–7` to day names (`Sunday` – `Saturday`).

### 9. Feature Engineering — `SHOP_HOUR_GROUP`
- Grouped `SHOP_HOUR` into 4 time buckets:
  - **Morning** (1–6)
  - **Afternoon** (7–12)
  - **Evening** (13–18)
  - **Night** (19–24)

### 10. Clean Data Export
- Saved the cleaned dataset to `cleaned_dunnhumby.csv` with separator `;` and decimal `,` (Excel-friendly for EU/Indonesia locale).

### 📊 Summary of Changes

| # | Stage | Action | Result |
|---|-------|--------|--------|
| 1 | Data Loading | Merged 118 CSVs via `glob` + `concat` | 31,057,875 rows × 24 cols |
| 2 | Storage | Saved to Parquet (snappy) | Efficient `.parquet` file |
| 3 | Missing Values | Filled `SPEND` from `SFNND` | `SPEND` complete |
| 4 | Missing Values | Filled `BASKET_TYPE` from `BASKET_TYFN` | Consistent columns |
| 5 | Type Conversion | `SHOP_WEEK` → str, `SHOP_DATE` → datetime | Time-series ready |
| 6 | Duplicates | Removed 15 duplicates | 31,057,860 rows |
| 7 | Type Conversion | `BASKET_ID` → int64 | No scientific notation |
| 8 | Transformation | `SHOP_WEEKDAY` → day names | `Tuesday`, `Friday`, etc. |
| 9 | Feature Engineering | Created `SHOP_HOUR_GROUP` | +1 new column |
| 10 | Export | Saved CSV (`;`, `,`) | Ready for BI tools |

**Total columns: 24 → 23** (dropped `SFNND` & `BASKET_TYFN`, added `SHOP_HOUR_GROUP`).

---

## 🔍 Key Insights

### 💡 Insight 1 — Seasonality is Sharp and Predictable

Grocery and Mixed-mission baskets **spike sharply every year in the run-up to Christmas** (both 2006 and 2007 show the pattern). Fresh produce does the opposite — it **drops steadily through December**, then **rebounds hard every January**.

### 💡 Insight 2 — A Sharp Buying Volume Spike at 9 PM (21:00)

Grocery and Mixed categories **nearly doubled** the volume sold at 9 PM compared to the previous hour. From 14:00 onward, sales in every category show a continuous drop before a sudden spike at 9 PM.

### 💡 Insight 3 — Strong at Retention, Incredibly Weak at Acquisition

**Returning customers** stay steady and continue to increase across the two-year period. However, **new customer acquisition saw a huge decline** over the same period — in May 2008, the company only acquired **2 new customers**.

---

## 🎯 Recommendations

**Based on Insight 1 — Seasonality**
- Begin building **Grocery and Mixed stock roughly 6 weeks ahead of Christmas** — not at the last minute.
- Hold back on Fresh over-ordering before December — demand dips well before December.
- Ramp up Fresh supply and run **"New Year, Fresh Start"** promotions in early January to meet the rebound.

**Based on Insight 2 — 9 PM Peak**
- Ensure **staffing and product availability during night hours**, especially for the **Fresh** category, since those items are more likely to run out or degrade in quality compared to earlier in the day.

**Based on Insight 3 — Acquisition Gap**
- Continue the **Retention Programme**, but adjust budgeting more towards a **New Customer Acquisition Programme**.
- Target the new acquisition programme toward **BG and CT customer segment lookalikes**, rather than a generic approach.

---

## 📁 Files

| File | Description |
|------|-------------|
| `DataAnalysisProject_dunnhumby_retail.ipynb` | Full data cleaning & transformation notebook |
| `DataAnalysisProject_dunnhumby_retail_visualization.pdf` | Business insight report |
| `DataAnalysisProject_dunnhumby_retail_SummaryofChanges.xlsx` | Summary of changes made with data in Python |
| `DataAnalysisProject_dunnhumby_retail.csv` | Clean sample data (only sample due to data size) |
| `README.md` | Project documentation |

*This project is part of a Data Analyst Portfolio. For other projects, see the [main repository](../README.md).*
