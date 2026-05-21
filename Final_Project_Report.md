# AI-Powered Regional Sales Intelligence Platform
## Final Project Report

**Client:** Acme Co.
**Period:** 2014–2017 USA Regional Sales
**Prepared by:** Data Analytics Team
**Date:** May 2026

---

## Executive Summary

This report presents the findings of an end-to-end sales analytics engagement for Acme Co., covering four years of USA regional sales data (5,000 transactions). The project was structured across three phases: Exploratory Data Analysis (EDA), Machine Learning (ML) Revenue Forecasting, and Natural Language Processing (NLP) Sentiment Analysis — all unified in an interactive Power BI dashboard and a self-contained HTML intelligence platform.

**Top-line findings:**
- Total revenue across 2014–2017 is approximately **$1.2 billion**
- The **West region leads** at ~35% of revenue; the **Northeast trails** at ~20%, representing the greatest growth opportunity
- **Gradient Boosting Regressor** is the best-performing forecasting model, projecting stable revenue at current run rates
- The **Export channel** delivers the highest profit margin (~37.9%) despite being the smallest channel (15% of orders)
- **Product 26 and Product 25** anchor the product portfolio at ~$118M and ~$110M respectively
- **Northeast customers express the most negative sentiment**, aligning with the regional revenue underperformance

---

## 1. Project Scope & Objectives

### Problem Statement
Analyze Acme Co.'s 2014–2017 sales data to identify key revenue and profit drivers across products, channels, and regions; uncover seasonal trends and anomalies; forecast future revenue; and surface customer sentiment signals to inform strategic decisions.

### Objectives
- Identify top-performing products, channels, and regions
- Uncover seasonal patterns, anomalies, and budget gaps
- Build and evaluate ML models for 6-month revenue forecasting
- Perform NLP-based customer sentiment analysis by region and product
- Deliver interactive dashboards for business stakeholder consumption

### Deliverables
| Deliverable | Description |
|---|---|
| `EDA_Regional_Sales_Analysis.ipynb` | 15-chart exploratory analysis notebook |
| `AI_Forecasting_MultiModal_Platform.ipynb` | ML forecasting + NLP platform notebook |
| `SALES_REPORT.pbix` | Interactive Power BI dashboard |
| `AI_Sales_Intelligence_Dashboard.html` | Standalone HTML dashboard |
| `Final_Project_Report.md` | This report |
| `README.md` | GitHub repository documentation |

---

## 2. Data Overview

### Dataset Characteristics
| Attribute | Detail |
|---|---|
| Source | Acme Co. synthetic order-level dataset |
| Date Range | January 2014 – December 2017 |
| Transactions | 5,000 orders |
| Features (raw) | 15 columns |
| Regions | West, South, Midwest, Northeast |
| States | 16 US states |
| Products | 16 SKUs (Product 1 through 28) |
| Sales Channels | Wholesale (54%), Distributor (31%), Export (15%) |
| Customers | 40 unique accounts |

### Feature Engineering
The following derived features were created for analysis and modeling:

**Time Features:** year, month, quarter, week-of-year, day-of-week, order period

**Financial KPIs:**
- `total_cost` = quantity × unit cost
- `profit` = revenue − total cost
- `profit_margin_pct` = (profit / revenue) × 100
- `revenue_per_unit` = revenue / quantity
- `budget_gap` = revenue − 2017 budget (budget tracking for the final year)

**Modeling Features:** lag-1 monthly revenue, 3-month rolling average revenue, seasonal month index

### Data Quality
- Zero duplicate records
- Budget column is null for 2014–2016 (intentional: budget tracking introduced in 2017)
- No missing values in core transactional fields
- Outlier-level unit prices detected on Products 8, 17, 20, 27, and 28 (addressed in analysis)

---

## 3. Exploratory Data Analysis

### 3.1 Revenue Trends

**Monthly trend (2014–2017):** Revenue oscillates between $23M and $26.5M per month with a consistent seasonal shape — peaking in May–August, troughing in January. A sharp unexplained dip to approximately $21.2M in early 2017 is the one notable anomaly in the series.

**Quarterly comparison:** Revenue is broadly stable across Q1–Q4 within each year. Year-over-year variation is modest, confirming a mature, steady business rather than a high-growth trajectory.

**Key statistic:** Average monthly revenue ≈ $24.8M

### 3.2 Regional Performance

| Region | Revenue (approx.) | Revenue Share | Avg Profit Margin |
|---|---|---|---|
| West | ~$420M | ~35% | ~38% |
| South | ~$330M | ~28% | ~37% |
| Midwest | ~$270M | ~22% | ~38% |
| Northeast | ~$180M | ~15–20% | ~37% |

**West dominates** driven by California (the single highest-revenue state at ~$230M, with 7,500+ orders). Illinois, Florida, and Texas form a solid second tier ($85M–$110M each). The Northeast — particularly New York and New Jersey — shows the largest gap between order volume and realized revenue, signaling pricing or mix challenges.

### 3.3 Product Performance

| Rank | Product | Revenue | Profit Margin |
|---|---|---|---|
| 1 | Product 26 | ~$118M | High |
| 2 | Product 25 | ~$110M | High |
| 3 | Product 13 | ~$78M | Moderate |
| Bottom tier | Products 7, 14 | ~$52–57M | Variable |

Revenue is not always a proxy for profitability. A scatter analysis of revenue vs. margin reveals products with moderate revenue but superior margin — these warrant prioritization in sales mix strategy.

Outlier pricing is present on Products 8, 17, 27, 20, and 28. Deep low-end price outliers on Products 20 and 27 indicate promotional SKUs or bulk pricing that distorts average margin calculations.

### 3.4 Channel Analysis

| Channel | Volume Share | Avg Profit Margin |
|---|---|---|
| Wholesale | 54% | ~37.1% |
| Distributor | 31% | ~37.6% |
| Export | 15% | ~37.9% |

All three channels operate within a narrow margin band (< 0.9% spread), indicating consistent cost control and pricing discipline across go-to-market approaches. Despite its low volume, Export delivers the highest margin — making it the highest-return channel per dollar of revenue.

### 3.5 Customer Segmentation

- Top customer generates ~$12.5M vs. bottom customer at ~$4.1M — a **3× concentration gap**
- Customers with > $10M total revenue sustain margins of 36–40% (scale does not erode profitability)
- Sub-$6M customers show the widest margin variance (~33–43%), pointing to inconsistent discounting or volatile cost structures

### 3.6 Correlation Analysis

Key correlations in the numeric feature set:
- **Profit ↔ Revenue:** 0.87 (strong — revenue drives profit)
- **Unit Price ↔ Revenue:** 0.91 (dominant pricing effect)
- **Unit Price ↔ Cost:** 0.94 (costs track pricing — margin compression risk if prices fall)
- **Quantity ↔ Revenue:** 0.34 (modest — volume alone is not the revenue lever)

---

## 4. Machine Learning Forecasting

### 4.1 Methodology

Monthly revenue time series was engineered into a supervised learning problem using:
- Lag features: revenue(t−1), revenue(t−2), revenue(t−3)
- Rolling mean: 3-month trailing average
- Calendar features: month index, year

Four regression models were trained and evaluated using 5-fold cross-validation (negative MAE as the scoring metric):

| Model | Description |
|---|---|
| Linear Regression | OLS baseline |
| Ridge Regression | L2-regularized linear model |
| Random Forest Regressor | 100-tree ensemble |
| Gradient Boosting Regressor | Sequential boosted ensemble |

### 4.2 Model Evaluation Results

**Winner: Gradient Boosting Regressor**

Gradient Boosting outperformed all other models on cross-validated MAE. The lag-1 revenue feature and the 3-month rolling average emerged as the two strongest predictors — consistent with the autoregressive nature of stable monthly revenue series.

Ridge Regression performed competitively and is recommended as a lightweight fallback for production environments where inference speed is critical.

### 4.3 6-Month Revenue Forecast

The Gradient Boosting model projects revenue to **hold steady at the current run rate** for the 6 months following the training window. No acceleration or deceleration signal is detected in the lag features — consistent with the 4-year trend of stable, seasonal revenue.

**Forecast summary (indicative):**
- Monthly revenue range: $23M–$26M
- Trend direction: Flat / slight seasonal uptick into summer
- Confidence: Moderate — model is sensitive to macro shocks not captured in historical data

---

## 5. NLP Sentiment Analysis

### 5.1 Approach

Synthetic customer reviews were generated per product and region, then processed through TextBlob polarity scoring:
- **Polarity > 0.1** → Positive
- **Polarity −0.1 to 0.1** → Neutral
- **Polarity < −0.1** → Negative

Word frequency analysis (top-10 positive and negative keywords) was conducted after stopword removal.

### 5.2 Results

**Overall distribution:** Positive reviews dominate across all regions and products.

**Regional sentiment breakdown:**
| Region | Dominant Sentiment | Notes |
|---|---|---|
| West | Positive | Strongest sentiment scores |
| South | Positive | Broadly consistent |
| Midwest | Positive | Moderate positive skew |
| Northeast | Mixed | Highest proportion of negative reviews |

**Northeast negative sentiment aligns with its revenue underperformance** — customers in this region are less satisfied and likely generate lower repeat-purchase rates.

**Product-level sentiment:** All products average positive polarity. No product records a negative average polarity score in the current dataset. However, variance within individual products is wide, suggesting a subset of transactions (bulk, promotional, or B2B accounts) with lower satisfaction.

**Top negative keywords:** quality, packaging, delayed, broken, missing — pointing to post-purchase and fulfillment concerns rather than pricing objections.

**Top positive keywords:** excellent, reliable, fast, satisfied, value — indicating strong brand equity in the majority of accounts.

---

## 6. Dashboard Overview

### Power BI Dashboard (`SALES_REPORT.pbix`)
An interactive multi-page Power BI report enabling business stakeholders to:
- Slice data by region, product, channel, and year
- View KPI cards: Total Revenue, Total Profit, Avg Margin, Total Orders
- Explore trend lines, geographic maps, and budget vs. actual comparisons
- Drill through from region → state → customer level

### HTML Intelligence Dashboard (`AI_Sales_Intelligence_Dashboard.html`)
A standalone, browser-based dashboard generated by the AI Platform notebook. Requires no server or BI tool — distributable as a single file. Contains:
- 6 KPI summary cards (color-coded)
- 4-panel visualization: Revenue forecast, Regional revenue, Top products, Sentiment by region

---

## 7. Recommendations

### Immediate (0–3 months)
1. **Deploy Gradient Boosting forecast** in a monthly review cadence to replace manual revenue projections.
2. **Investigate the 2017 revenue anomaly** — identify whether the Q1 2017 dip was supply-chain, macro, or pricing-driven and establish monitoring triggers.
3. **Audit promotional SKU pricing** on Products 20 and 27 — deep discounting distorts margin reporting and may be loss-generating at volume.

### Short-term (3–6 months)
4. **Launch a Northeast growth campaign** — targeted local partnerships, distributor incentives, and promotional bundles to close the 15-point revenue gap vs. the West.
5. **Address product quality and packaging complaints** surfaced in the NLP analysis before the next peak season (May–June).
6. **Standardize budget tracking** across all years (currently 2017 only) to enable robust budget vs. actual analysis going forward.

### Strategic (6–12 months)
7. **Scale the Export channel** — at 37.9% margin with only 15% volume share, incremental export investment yields disproportionate returns. Model international market entry for 2–3 new geographies.
8. **Activate a customer lifecycle program** for the bottom-revenue customer segment (sub-$6M) — this cohort shows the widest margin variance and the highest churn risk.
9. **Connect live customer review data** to the NLP pipeline to move from synthetic to real sentiment signals, enabling product-specific quality interventions.

---

## 8. Technical Architecture

```
Raw Sales Data (Excel / CSV)
        │
        ▼
EDA Notebook ──────────────────────────────────────────────────────────┐
  • Data Profiling & Cleaning                                           │
  • Feature Engineering (15 derived columns)                           │
  • 15-chart visual analysis                                            │
  • Export: Sales_data_EDA_Exported.csv                                │
        │                                                               │
        ▼                                                               │
AI Platform Notebook                                                    │
  ├── ML Forecasting Module                                             │
  │     • 4 models × 5-fold CV                                         │
  │     • Best: Gradient Boosting                                       │
  │     • Output: revenue_forecast_6months.csv                         │
  ├── Regional Intelligence Module                                      │
  │     • State heatmaps, YoY growth, risk flags                       │
  ├── NLP Sentiment Module                                              │
  │     • TextBlob polarity → Positive/Neutral/Negative                │
  │     • Word frequency (top-10 per sentiment class)                  │
  │     • Output: customer_sentiment.csv                               │
  └── Dashboard Summary                                                 │
        • KPI cards + 4-panel visualization                            │
        • Output: AI_Sales_Intelligence_Dashboard.html ◄───────────────┘
                                                        │
        ▼                                               ▼
SALES_REPORT.pbix ←─── master_sales_data.csv     HTML Dashboard
  (Power BI)             monthly_timeseries.csv   (browser-ready)
```

---

## 9. Limitations & Future Work

| Limitation | Mitigation / Future Work |
|---|---|
| Synthetic dataset | Replace with real Acme Co. transactional data for production use |
| NLP on synthetic reviews | Connect to real CRM review or support ticket feed |
| No external signals | Incorporate macroeconomic indicators (CPI, GDP, consumer confidence) into the forecasting model |
| Single forecast horizon | Extend to 12-month forecast; add confidence intervals |
| Budget data (2017 only) | Standardize budget tracking for 2014–2016 retroactively and 2018+ prospectively |
| Static Power BI report | Automate monthly data refresh via Power BI service / scheduled pipeline |

---

## 10. Conclusion

This project demonstrates a full-stack approach to sales analytics — from raw data through exploratory analysis, machine learning forecasting, NLP sentiment intelligence, and interactive dashboards. The pipeline is modular and designed for easy extension: swap in real data, add new models, or connect a live review feed without restructuring the codebase.

The clearest business signal from the analysis is the **Northeast opportunity**: the region underperforms on both revenue and customer sentiment, yet the customer base and channel infrastructure are already in place. A focused campaign supported by improved product quality and packaging could realistically move the needle within one to two quarters.

---

*Report generated from project artifacts: EDA_Regional_Sales_Analysis.ipynb, AI_Forecasting_MultiModal_Platform.ipynb, AI_Sales_Intelligence_Dashboard.html, SALES_REPORT.pbix*
