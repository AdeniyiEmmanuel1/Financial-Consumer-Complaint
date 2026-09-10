# **# Financial Consumer Complaint Analytics | SQL Server + Tableau Project**

An End-to-end analytics project turning raw CFPB consumer complaint data into a decision-ready Tableau dashboard for Bank of America's financial products, covering **62,516 complaints from 2017–2023**.

------------------------------------------------------------------

## **Dashboard Previews**

![Dashboard Screenshot](https://github.com/AdeniyiEmmanuel1/Financial-Consumer-Complaint/blob/main/Consumer%20Complaint.png)

---------------------------------------------------------------------


## **Business Questions**


1. Do consumer complaints show any seasonal patterns?
2. Which products generate the most complaints, and what are their most common issues?
3. How are complaints typically resolved?
4. What can we learn from complaints with untimely responses?

## **Architecture**

## Built on a **Bronze → Silver → Gold medallion architecture** in SQL Server.

```
CFPB Source CSV
      │
      ▼
┌─────────────┐   BULK INSERT (FORMAT='CSV', FIELDQUOTE='"')
│   Bronze    │   Raw ingestion — schema & completeness checks
│  Layer      │   Bronze.Customer_Complaints_Raw (12 columns, matches source exactly)
└─────────────┘
      │
      ▼
┌─────────────┐   Cleaning, standardization, correctness checks
│   Silver    │   Derived columns: response_lag_days, is_timely_response,
│  Layer      │   product_category, issue_category, resolution_category, us_region
└─────────────┘
      │
      ▼
┌─────────────┐   Business-ready aggregation & integration
│    Gold     │   Gold.Fact_Customer_Complaints — analysis-ready fact table
│  Layer      │   feeding directly into Tableau
```

**Why medallion:** each layer has a single responsibility: Bronze preserves raw fidelity for auditability, Silver isolates all cleaning/business logic in one place, and Gold stays lean and purpose-built for BI consumption, so Tableau never has to guess at data quality.

## **Tableau Dashboard**

Ten visualizations, each mapped to a specific measure/dimension combination and a specific business question:

| # | Visualization | Dimension(s) | Measure(s) | Answers |
|---|---|---|---|---|
| 1 | KPI Tiles | — | `COUNTD(complaint_id)`, `AVG(is_timely_response)`, `AVG(response_lag_days)`, `COUNTD(product)`, % Monetary Relief | Headline performance |
| 2 | Butterfly Chart | `resolution_category` | `Timely Complaints`, `Untimely Complaints` (calculated fields) | How complaints resolve, by timeliness |
| 3 | Running Total | `submission_year_month` | `COUNTD(complaint_id)`, Running Total | Seasonal/cumulative trend |
| 4 | Bar + Line Combo | `submission_month_name` | `COUNTD(complaint_id)`, `AVG(response_lag_days)` | Seasonality of volume vs. lag |
| 5 | Scatter Plot | `product_category`, `issue_category` | `COUNTD(complaint_id)`, `AVG(is_timely_response)` | Volume vs. response quality by product/issue |
| 6 | Bump Chart | `submission_year_month` (Quarter), `issue_category` | `AVG(response_lag_days)`, Rank | Which issues stay chronically slow |
| 7 | Donut Chart | `issue_category` / `resolution_category` | `COUNTD(complaint_id)` | Complaint/resolution mix |
| 8 | KPI Highlight Table | `product_category`, `resolution_category` | `AVG(is_timely_response)` | Timeliness by product × resolution |
| 9 | Global Filters | `product_category`, `us_region` | — | Cross-filtering across all views |
| 10 | Regional Butterfly | `us_region` | `Timely Complaints`, `Untimely Complaints` | Geographic timeliness comparison |

**Calculated fields used:**
```
Number of Complaints  = COUNTD([complaint_id])
Timely Complaints     = SUM(IF [is_timely_response] = 1 THEN 1 ELSE 0 END)
Untimely Complaints   = SUM(IF [is_timely_response] = 0 THEN 1 ELSE 0 END)
% Timely              = AVG([is_timely_response])
% Monetary Relief     = SUM(IF [resolution_category] = 'Monetary Relief' THEN 1 ELSE 0 END) / COUNTD([complaint_id])
```

---

## Key Insights

- **93.8%** of complaints receive a timely response, with an average response lag of **1.2 days** overall.
- **Credit Reporting** and **Checking/Savings Accounts** are the highest-volume product categories, far ahead of other products.
- **65.1%** of complaints close with an explanation only; **23.5%** result in monetary relief; **8.5%** in non-monetary relief.
- Ranking issue categories by response lag on a quarterly basis shows that slow resolution is **not evenly distributed** — a subset of issue categories persistently rank at the bottom, suggesting targeted (not blanket) process fixes would have the most impact.
- The scatter plot of complaint volume against timely-response rate highlights specific product/issue combinations that are both high-volume and below-average on responsiveness — the clearest candidates for operational attention.

---

## Tech Stack

- **SQL Server**; medallion architecture (Bronze/Silver/Gold), data ingestion, cleaning, and business logic
- **Tableau**; dashboard design and interactive visualization
- **Data Source**; [Consumer Financial Protection Bureau (CFPB)](https://www.consumerfinance.gov/data-research/consumer-complaints/) complaint data, 2017-2023

---

## How to Reproduce

1. Restore/create the `Bronze`, `Silver`, and `Gold` schemas in SQL Server.
2. Run the scripts in `sql/` in order (`01` → `04`).
3. Point Tableau at `Gold.Fact_Customer_Complaints` (or export to the provided `Gold_Fact_Customer_Complaints.xlsx`).
4. Open `tableau/Financial_Consumer_Complaint_Dashboard.twbx` to explore the workbook, or rebuild each sheet using the measure/dimension table above
	  
•	Sales & Profit Performance by Sub-Category: dual horizontal bar chart showing sales (dark) vs. profit/loss (blue/orange) per product line; Tables and Bookcases highlighted as loss making

•	Sales and Profit Trends Over Time: dual step-line chart with dashed average reference lines (Avg. $4K sales, Avg. $1K profit) showing above/below average periods


## **Datasets**

	•	**Financial Consumer Complaints.csv:** Complaint ID,	Submitted via,	Date submitted,	Date received,	State,	Product	Sub-product	Issue,	Sub-issue,	Company public response,	Company response to consumer,	Timely response
	
    
## **Acknowledgement**

This project was built following the Tableau Ultimate Course with the mentorship of baraa. The 4-step dashboard design process, container mockup methodology, and project structure are credited to that course.

Author

Created by Aden Emmanuel

LinkedIn: https://www.linkedin.com/in/aden-emmanuel-117440142/?skipRedirect=true



