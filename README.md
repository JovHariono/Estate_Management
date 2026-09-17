# SML Estate Management — IPL Collection & Utility Billing Audit

Capstone Project, Module 2 (Purwadhika) — a data cleaning, EDA, and business
analytics audit of a Sinarmas Land-style township management portfolio: IPL
(environmental maintenance fee) collection, vacant-property risk, and utility
(water) billing integrity, across 300,000 monthly invoices.

## Background

SML operates several self-contained townships (BSD City, Kota Wisata, Grand
Wisata, Deltamas, NavaPark), earning recurring revenue from IPL, water
utility billing, and commercial leasing. Two structural problems threaten
that revenue: units bought purely for capital gain and left vacant for years
while owners ignore IPL bills, and a billing/ERP system prone to human error
in meter readings and inconsistent payment-method logging. This project
quantifies both, plus a logical-impossibility bug in the billing system
(invoices marked "Paid" before the unit was even handed over).

## Source data

`sml_clusters.csv` 100 one row per cluster
`sml_units.csv` 25,000 one row per unit (house/shophouse)
`sml_ipl_billings.csv` 300,000 one row per monthly invoice

## What the notebook does

The notebook (`Estate_Management.ipynb`)
runs end to end, section by section:

1. **Problem Statement, Goals, Dataset Dictionary** — the business framing.
2. **Data Loading** — reads the three source CSVs; `contact_number` is loaded
   as `string` explicitly to avoid pandas coercing it to a float and dropping
   the leading `0`.
3. **Data Understanding** — shape, dtypes, initial quality checkpoints.
4. **Data Cleaning & Wrangling**
   - 6.1 Duplicate checks (row-level and primary-key)
   - 6.2 Payment method standardization (9 raw labels → 4 categories)
   - 6.3 Water usage cleaning — separates **negative readings** (physically
     impossible) from **sentinel readings** (`9999`, a meter-read-failure
     error code) using an exact-frequency scan before treating either as
     invalid; outlier detection is done **per `cluster_category`**, not
     globally, since Commercial units legitimately use more water than
     Residential
   - 6.4 Missing-value profile
   - 6.5 Referential integrity (every invoice ↔ valid unit ↔ valid cluster)
   - 6.6 Duplicate removal
   - 6.7 Contact number normalization (`+62`/dash/space handling, format
     validation against a real Indonesian mobile pattern, not just
     null-checking)
5. **Feature Engineering** — merges all three tables into `df`; derives
   `is_unpaid`, `aging_bucket`, `pre_handover_payment`,
   `water_outlier` (aliased to the segmented flag from step 6.3),
   `contact_missing` (based on format validity, not just nulls).
6. **EDA & Visualisasi** — outstanding by township, vacant-vs-occupied,
   aging, payment-method quality, missing contacts, pre-handover anomalies,
   water outliers.
7. **Statistical Analysis** — Chi-square test of independence between
   vacancy status and payment status.
8. **Business Summary / KPI, Key Findings, Recommended Actions**
9. **Export Clean Dataset** — writes all four CSVs to `output_capstone/`.

## Key findings

 1.  Largest outstanding by township  **BSD City** — Rp 39,04 M / 29.808 invoice 
 2.  Vacant vs Occupied unpaid rate  **60,10%** vs 20,18%
 3.  Outstanding aged > 6 months  **Rp 87,45 M** (74,6% of total outstanding), 67.645 invoice 
 4.  Payment method cleansing  9 raw labels → 4 categories; 30,09% "Unknown" is structural (unpaid invoices have no payment record yet) 
 5.  Invalid/missing contact numbers  **20,0%** of units (5.000 / 25.000) → 60.046 invoices affected 
 6.  Pre-handover payment bug  **6.291 invoices** marked Paid before `handover_date` 
 7.  Water usage sentinel vs real outliers  1.481 invoices at exactly `9999 m³` (meter-read failures) + 1.519 negative readings nulled; **zero** genuine statistical outliers remain after cleaning (segmented IQR, per category) 
 8.  **Billing integrity (found while building the dashboard)**  The `9999` sentinel value was billed through at face value: raw total billed **Rp 400,78 M** overstates the true figure by **Rp 119,44 M** (9.225 anomalous invoices); true outstanding is **Rp 81,94 M**, not Rp 117,26 M 

## How to reproduce

1. Place `sml_clusters.csv`, `sml_units.csv`, `sml_ipl_billings.csv` in the
   same directory as the notebook.
2. Run all cells top to bottom in Jupyter (`pandas`, `numpy`, `matplotlib`,
   `seaborn`, `scipy` required).
3. writes the four output CSVs to `output_capstone/`.