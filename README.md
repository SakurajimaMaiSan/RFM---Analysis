# Customer Segmentation with RFM Analysis & K-Means

![Python](https://img.shields.io/badge/Python-3.10%2B-blue)
![Jupyter](https://img.shields.io/badge/Notebook-Jupyter-orange)
![scikit-learn](https://img.shields.io/badge/ML-scikit--learn-f7931e)

Segment e-commerce customers by **Recency, Frequency and Monetary value (RFM)**, validate the segments with **K-Means clustering**, and predict **days until a customer's next purchase** with a Random Forest regressor.

Everything lives in one notebook: [`RFM-Analysis.ipynb`](RFM-Analysis.ipynb).

---

## Table of Contents

- [What's Inside](#whats-inside)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Segments & Suggested Actions](#segments--suggested-actions)
- [Getting Started](#getting-started)
- [Project Structure](#project-structure)
- [Known Limitations](#known-limitations)

---

## What's Inside

| Stage | What it does |
|-------|--------------|
| **Data cleaning** | Drops rows with no Customer ID, cancelled invoices and non-positive quantities; builds `TotalPrice` |
| **RFM metrics** | Computes Recency, Frequency and Monetary value per customer |
| **Rule-based segmentation** | Scores each metric 1–4 and maps the total score to five segments |
| **K-Means clustering** | Log-transforms and scales RFM, picks *k* with the elbow method, profiles the clusters |
| **Segment vs. cluster comparison** | Heatmap, bar charts, radar charts and aggregate statistics |
| **Geography** | Segment × country breakdown with an interactive treemap |
| **Product preferences** | Top 5 products and cost per item for each segment |
| **Next-purchase model** | Random Forest regression on inter-purchase behaviour |

---

## Dataset

The notebook expects **`online_retail_II.csv`** (transactional data from a UK online retailer) in the same folder as the notebook. The file is not included in this repo.

| Column | Description |
|--------|-------------|
| `Invoice` | Invoice number (prefix `C` = cancellation) |
| `StockCode` | Product code |
| `Description` | Product name |
| `Quantity` | Units per line item |
| `InvoiceDate` | Timestamp, format `m/d/yy H:MM` |
| `Price` | Unit price |
| `Customer ID` | Customer identifier (missing for guest checkouts) |
| `Country` | Customer country |

The file is read with `encoding="ISO-8859-1"` (it contains non-UTF-8 characters such as `£`).

Source: the UCI *Online Retail* / *Online Retail II* datasets (also mirrored on Kaggle).

---

## Methodology

### 1. Cleaning
- Drop rows with a missing `Customer ID`
- Remove cancelled invoices (`Invoice` starting with `C`)
- Keep only `Quantity > 0`
- Cast `Customer ID` to `int` and add `TotalPrice = Quantity × Price`

### 2. RFM metrics
The snapshot date is **one day after the last transaction**.

| Metric | Definition |
|--------|------------|
| **Recency** | Days between the snapshot date and the customer's last purchase |
| **Frequency** | Number of unique invoices |
| **Monetary** | Sum of `TotalPrice` |

### 3. Scoring and segments
- **R score:** quartiles of Recency, reversed so recent customers score 4
- **F score:** fixed bins `[0, 1, 3, 7, max]` → scores 1–4
- **M score:** quartiles of Monetary value
- **RFM score** = R + F + M (range 3–12)

| RFM score | Segment |
|-----------|---------|
| 11–12 | Champions |
| 9–10 | Loyal Customers |
| 6–8 | Potential Loyalists |
| 5 | At-Risk Customers |
| 3–4 | Can't Lose Them |

### 4. K-Means clustering
1. Apply `log1p` to reduce skew in R, F and M
2. Standardise with `StandardScaler`
3. Choose *k* with the elbow method (k = 1–10)
4. Fit K-Means with **k = 4** (`random_state=42`, `n_init=10`)
5. Profile each cluster and compare it to the rule-based segments

### 5. Next-purchase prediction
- **Target:** days between a customer's last two purchases
- **Features:** Recency, Frequency, MonetaryValue, average inter-purchase time
- **Model:** `RandomForestRegressor(n_estimators=100)` with a 70/30 train/test split
- **Metrics:** MAE (days) and R²
- **Output:** predicted days until next purchase for Champions and Loyal Customers

---

## Segments & Suggested Actions

| Segment | Suggested action |
|---------|------------------|
| **Champions** | Exclusive offers, early access, loyalty rewards; cross-sell premium items |
| **Loyal Customers** | Upsell larger sets and premium finishes; keep revenue stable |
| **Potential Loyalists** | Frequent low-ticket promotions to grow purchase frequency |
| **At-Risk Customers** | Seasonal campaigns and discounts |
| **Can't Lose Them** | Win-back campaigns, seasonal and holiday reminders |

For the next-purchase model, a short predicted gap (0–5 days) marks a customer who is already active. Skip "we miss you" discounts for them. A long predicted gap (over 90 days) marks a lapsed customer who needs a win-back offer.

---

## Getting Started

### Requirements
- Python 3.10+
- Jupyter Notebook or JupyterLab

Developed with Python 3.13, pandas 2.3.3, NumPy 2.3.5, Matplotlib 3.10.6 and Seaborn 0.13.2.

### Install

```bash
pip install pandas numpy matplotlib seaborn scikit-learn openpyxl plotly
```

### Run

1. Put `online_retail_II.csv` next to the notebook.
2. Launch Jupyter:
```bash
   jupyter notebook RFM-Analysis.ipynb
```
3. Choose **Kernel → Restart & Run All**.

The notebook is meant to run top to bottom. The cleaning cell must run **before** the RFM cell, otherwise `TotalPrice` will not exist.

---

## Project Structure

```
.
├── RFM-Analysis.ipynb                     # Full analysis
├── online_retail_II.csv                   # Input data (download separately)
└── top_5_products_by_segment_plot.png     # Generated by the notebook
```

---

## Known Limitations

- **Snapshot in time:** Recency is measured against the last date in the file, so results shift if the data changes.
- **Segment labels:** names are assigned by RFM score band. Check the aggregate statistics before acting on a label.
- **Purchase gaps:** inter-purchase times are computed from line-item timestamps. Items on the same invoice produce 0-day gaps, which pulls many predictions towards zero.
- **Model scope:** the regressor uses a random train/test split, not a time-based one, and has no seasonality features.

---

## Possible Extensions

- Time-based cross-validation (`TimeSeriesSplit`) for the regression model
- Gradient boosting or XGBoost as alternative regressors
- Customer lifetime value (CLV) estimates per segment
- Silhouette scores to support the choice of *k*
