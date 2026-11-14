# Retail Customer & Sales Analytics
### An end-to-end analytics project using Python, PostgreSQL, and Power BI

An analysis of historical retail transactions designed to answer three business questions:

1. What drives sales performance across products, countries, and time?
2. Which customers show strong engagement or signs of inactivity?
3. How does repeat purchasing differ across customer cohorts?

The project connects data preparation, SQL-based business analysis, and interactive reporting in a reproducible workflow.

> Project status: Core cleaning and SQL analysis documented. The first version of the customer-segmentation dashboard is complete; sales-overview and cohort-retention pages remain in development.
>
> Portfolio context: This is a historical-data portfolio project, not a production deployment or a commissioned business engagement.

---

## Dashboard Preview

![Customer segmentation dashboard](05_power_bi_dashboard/rfm_page.png)

*Customer-segmentation page. Sales-overview and cohort-retention pages are under development.*

---

## Business Context

Sales totals show overall performance, but they do not explain whether customers return, which customer groups contribute value, or where engagement is weakening.

This project brings those questions together through:

- Sales and order-performance analysis.
- Customer segmentation using Recency, Frequency, and Monetary value.
- Monthly cohort analysis of repeat purchasing.
- Interactive reporting for reviewing patterns and potential actions.

The goal is to turn transaction records into transparent, explainable business metrics—not simply produce charts.

---

## Dataset

| Attribute | Details |
|---|---|
| Dataset | UCI Online Retail II |
| Source | [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii) |
| Coverage | 1 December 2009–9 December 2011 |
| Raw size | 1,067,371 transaction-line records |
| Business | UK-based non-store retailer |
| Currency | GBP |
| Important fields | Invoice, product, quantity, invoice date, unit price, customer ID, country |

Each row represents a product line associated with an invoice. Row counts should not be interpreted as distinct order counts.

The source includes missing values and cancellation invoices. Invoice numbers beginning with `C` identify cancellations.

### Data Provenance

Use the original UCI workbook to reproduce the cleaning workflow described here.

A third-party cleaned dataset may apply different filters or transformations and should not be assumed to reproduce the same row counts, cancellation metrics, or customer totals.

---

## Analytical Workflow

```text
Raw Excel workbook
        ↓
Python: combine sheets, inspect data, flag cancellations
        ↓
Prepared CSV
        ↓
PostgreSQL: analytical views, RFM scoring, cohort analysis
        ↓
Power BI: business metrics and customer reporting
        ↓
Documented findings, limitations, and potential actions
```

---

## Technology Stack

| Tool | Role |
|---|---|
| Python | Data preparation and validation |
| pandas / NumPy | Tabular processing and transformations |
| SQLAlchemy | Loading prepared data into PostgreSQL |
| PostgreSQL | Business metrics, analytical views, and customer analysis |
| Jupyter Notebook | Analysis walkthrough and documentation |
| Power BI | Interactive dashboards and reporting |

---

## 1. Data Preparation

The documented preparation pipeline:

- Combines the two annual Excel sheets.
- Standardizes column names.
- Flags cancellation invoices using an `is_cancelled` field.
- Removes records with missing product descriptions.
- Creates a line-value field using quantity multiplied by unit price.
- Exports the prepared data for PostgreSQL ingestion.

### Documented Record Counts

| Stage | Records |
|---|---:|
| Combined raw data | 1,067,371 |
| Records removed for missing descriptions | 4,382 |
| Prepared output | 1,062,989 |

These counts should be reproduced by running the notebook against the original workbook.

“Prepared” does not mean every record is suitable for every metric. Customer analysis, sales reporting, and cancellation reporting require different eligibility rules.

---

## 2. Sales and Business Metrics

The SQL layer organizes recurring business questions into reusable views.

| Analysis | Business question |
|---|---|
| Overall performance | What are the recorded sales value, distinct invoice count, and customer count? |
| Yearly and monthly trends | How does performance change over time? |
| Product performance | Which products contribute the most units and sales value? |
| Country performance | How is sales value distributed geographically? |
| Customer value | Which identified customers contribute the most recorded spend? |
| New vs. returning customers | How does the observed customer mix change each month? |
| Segment contribution | How much sales value is associated with each RFM segment? |
| Cancellations | How does recorded cancellation activity change over time? |

### Metric Interpretation

- Line value is calculated as `quantity × unit_price`.
- Order counts should use distinct eligible invoice identifiers, not row counts.
- Average order value requires the same invoice-eligibility rules in both its numerator and denominator.
- Customer metrics require identifiable customers.
- Sales value is not profit; product-cost data is not included.
- Cancellation treatment must be checked in each SQL view before interpreting its output.

The SQL definitions are the source of truth for the implemented filters.

---

## 3. RFM Customer Segmentation

RFM summarizes observed purchasing behavior using three dimensions:

| Dimension | Meaning |
|---|---|
| Recency | Days between the analysis reference date and the customer's latest eligible purchase |
| Frequency | Number of distinct eligible purchase invoices |
| Monetary | Total eligible customer spend |

The project uses quartile-based scoring with `NTILE(4)` and maps the resulting scores to business-oriented customer segments.

### Interpretation Checks

Before using the segment outputs:

- Confirm the reference date used for recency.
- Confirm that more recent purchasing receives a better recency score.
- Check that frequency counts invoices rather than product rows.
- Confirm how cancellations and missing customer IDs are handled.
- Read the SQL rules behind each segment label.

### Business Use

The resulting groups can support hypotheses such as:

- Prioritizing retention activity for previously valuable customers who have become inactive.
- Reviewing engagement opportunities for recent customers.
- Understanding the contribution of frequent, high-spending customers.

These are potential uses—not measured campaign outcomes.

### Important Limitation

RFM is a rules-based summary of historical behavior, not a trained churn-prediction model.

Labels such as “Potential Churners” or “Churned Customer” are analytical categories. They do not establish that a customer has permanently left or quantify the probability of future churn.

---

## 4. Cohort Retention Analysis

Customers are grouped by the month of their first observed eligible purchase.

The analysis then tracks:

- Distinct customers purchasing again in subsequent months.
- Sales value generated by each cohort over time.

### Outputs

| Output | Purpose |
|---|---|
| Customer-level cohort table | Compare repeat-purchase activity across cohorts |
| Revenue-level cohort table | Track recorded sales contribution as cohorts age |

For percentage-based retention:

`Retention in month n = returning cohort customers in month n ÷ original cohort size`

### Interpretation Notes

- “First purchase” means first observed purchase within this dataset, not necessarily the customer's first-ever purchase.
- Newer cohorts have shorter observation periods.
- A month that has not yet been observed should not be treated as zero retention.
- Cohort comparisons should use equivalent follow-up periods.
- Cohort patterns alone do not establish the cause of customer behavior.

---

## 5. Power BI Dashboard

| Dashboard page | Scope | Status |
|---|---|---|
| Sales Overview | Sales value, orders, customers, average order value, and monthly trends | In progress |
| Customer Segmentation | RFM distribution and customer-segment metrics | First version complete |
| Cohort Retention | Monthly retention heatmap and cohort comparisons | Planned |

### Reporting Priorities

- Clear metric names and currency labels.
- Consistent filtering between SQL outputs and visuals.
- Visible definitions for customer segments.
- Distinction between sales performance and cancellation activity.
- Transparent reporting of unfinished features.

---

## Findings and Recommendations

Numerical findings should be added only after reproducing and validating the analysis.

For each finding, document:

| Element | What to record |
|---|---|
| Observation | The measured result |
| Evidence | Supporting SQL view, output, or dashboard visual |
| Interpretation | What the pattern may indicate |
| Potential action | A practical response worth testing |
| Limitation | What the analysis cannot establish |

Suggested reporting format:

> Observation: [Verified result and relevant period]  
> Evidence: [Query or dashboard reference]  
> Interpretation: [Reasonable explanation]  
> Potential action: [Proposed next step]  
> Limitation: [Uncertainty or missing evidence]

No increase in revenue, retention, or campaign performance is claimed without a measured intervention.

---

## Repository Structure

```text
.
├── 01_data_cleaning_with_Python/
│   ├── retail_data_cleaning_and_preparation.ipynb
│   └── Queries.ipynb
│
├── 02_sql_scripts_in_PostgreSQL/
│   ├── kpi_test_queries.sql
│   ├── RFM_Segmentation.sql
│   └── Cohort_Analysis.sql
│
├── 03_cohort_analysis/
│   ├── README.md
│   ├── Cohort_Analysis.sql
│   ├── Cohort_Analysis_on_Revenue.csv
│   └── Cohort_analysis_on_Customer_Level.csv
│
├── 04_RFM_segmentation/
│   ├── README.md
│   ├── RFM_Segmentation.sql
│   └── rfm_final_score.csv
│
├── 05_power_bi_dashboard/
│   ├── Retail_analysis.pbix
│   └── rfm_page.png
│
└── README.md
```

---

## Run Locally

### Prerequisites

- Python and Jupyter Notebook or JupyterLab.
- A running PostgreSQL instance.
- Power BI Desktop for opening the dashboard.
- The original UCI Online Retail II workbook.

### 1. Clone This Repository

```bash
git clone [https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git](https://github.com/YOUR_USERNAME/YOUR_REPOSITORY.git)
cd YOUR_REPOSITORY
```

Replace the placeholders with this repository's actual URL and folder name.

### 2. Install Dependencies

```bash
pip install pandas numpy openpyxl sqlalchemy psycopg2-binary jupyter
```

### 3. Download the Source Data

Download the original workbook from:

https://archive.ics.uci.edu/dataset/502/online+retail+ii

Update the input path in the cleaning notebook to match your local file location.

### 4. Run the Preparation Notebook

```text
01_data_cleaning_with_Python/retail_data_cleaning_and_preparation.ipynb
```

Review the row counts and generated fields before exporting the prepared CSV.

### 5. Load Data into PostgreSQL

Open:

```text
01_data_cleaning_with_Python/Queries.ipynb
```

Configure the local database connection and load the prepared data into `retail_data`.

Do not commit database passwords or other credentials.

### 6. Create the Analytical Views

Create the RFM and cohort views, then run dependent KPI views.

Review dependencies in the scripts: KPI views referencing RFM outputs require those outputs to exist first.

```text
02_sql_scripts_in_PostgreSQL/RFM_Segmentation.sql
02_sql_scripts_in_PostgreSQL/Cohort_Analysis.sql
02_sql_scripts_in_PostgreSQL/kpi_test_queries.sql
```

Adjust the execution order if your implementation uses different dependencies.

### 7. Open Power BI

Open:

```text
05_power_bi_dashboard/Retail_analysis.pbix
```

Update database connections and local CSV paths, then refresh the report.

---

## Validation Checklist

The following checks should be completed before treating the outputs as final:

- [ ] Reproduce the documented input and output row counts.
- [ ] Confirm cancellation-flag logic.
- [ ] Review missing customer IDs and their treatment in customer metrics.
- [ ] Check nonpositive quantities and prices.
- [ ] Investigate repeated rows before removing them.
- [ ] Reconcile sales totals across Python, SQL, and Power BI using identical filters.
- [ ] Verify distinct invoice and customer counts.
- [ ] Confirm that RFM scoring directions match the intended interpretation.
- [ ] Validate cohort denominators and observation windows.
- [ ] Check that incomplete months are not compared with full months without a warning.

---

## Limitations

- The dataset is historical and does not represent current retail conditions.
- It represents one retailer and should not be generalized to all businesses.
- Missing identifiers limit customer-level analysis.
- Dataset boundaries limit what can be concluded about acquisition and inactivity.
- RFM segments describe observed behavior; they are not validated churn predictions.
- No campaign, experiment, or production deployment is included.
- The full dashboard is not yet complete.

---

## My Contribution

This implementation is adapted from an existing open-source reference project.

My verified contributions:

- [Describe a cleaning or validation improvement you implemented.]
- [Describe a SQL query or analytical improvement you implemented.]
- [Describe a dashboard page or visual you built or modified.]
- [Describe findings you independently reproduced and checked.]

Replace these placeholders with completed work before publishing. If you have only reproduced the original workflow, state that clearly.

---

## Next Improvements

- Complete the sales-overview dashboard.
- Add the cohort-retention dashboard.
- Publish a data-quality summary.
- Add automated metric-reconciliation checks.
- Document a fixed RFM reference date.
- Pin dependency versions after testing the environment.

These items are planned work, not completed features.

---

## Credits and Attribution

### Reference Implementation

This project adapts work by Md Shah Nawaj:

https://github.com/Md-ShahNawaj/online-retail-data-analysis-python-sql-powerbi

Retain applicable copyright notices and follow the original repository's licensing terms.

### Dataset

Chen, D. (2012). Online Retail II [Dataset]. UCI Machine Learning Repository.

https://doi.org/10.24432/C5CG6D

The UCI dataset is distributed under CC BY 4.0. Dataset licensing is separate from the licensing of repository code.
