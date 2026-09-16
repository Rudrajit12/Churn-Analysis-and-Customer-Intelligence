# Customer Churn Intelligence & Retention Analytics

## 1. Problem Statement

Customer churn is a critical business problem for subscription-based businesses such as OTT platforms. When customers cancel their subscriptions, the business loses recurring revenue and potentially significant customer lifetime value (CLTV).

The business therefore needs to understand:

- **Who** is churning or represents elevated churn risk?
- **Why** are customers leaving?
- **When** does churn occur in the customer lifecycle?
- **What** retention actions should be investigated or prioritized?

The project uses a relational SQLite database containing customer, subscription, and customer-support information to build an end-to-end **Customer Churn Intelligence** solution.

The objective is not simply to calculate a churn percentage. The analysis connects customer behavior and attributes with subscription structure, support interactions, churn risk, and financial value to produce actionable business insights.

---

# 2. Business Objective

Build a portfolio-grade churn analytics solution that transforms raw relational customer data into decision-ready retention intelligence.

The project aims to:

1. Identify customer segments with elevated churn rates.
2. Understand factors associated with customer churn.
3. Identify potential churn danger zones across the customer lifecycle.
4. Analyze churn across contract, plan, acquisition, geography, and support dimensions.
5. Quantify monthly-charge and CLTV exposure associated with churn.
6. Identify high-risk and high-value customers for retention investigation.
7. Translate analytical findings into practical retention hypotheses.
8. Establish a foundation for future predictive churn modeling and retention decision systems.

---

# 3. Dataset & Data Model

The database is `customer_churn.db`.

It contains three primary relational tables.

## 3.1 Customer Table — `db_customer`

Contains customer profile information:

| Column | Description |
|---|---|
| `customerid` | Unique customer identifier |
| `name` | Customer name |
| `country` | Customer country |
| `state` | Customer state |
| `gender` | Customer gender |
| `dob` | Date of birth |
| `interests` | Customer interests |
| `pincode` | Postal code |

## 3.2 Subscription Table — `db_subscription`

Contains subscription and customer-value information:

| Column | Description |
|---|---|
| `customerid` | Customer identifier |
| `subscription_start_date` | Subscription start date |
| `subscription_type` | Acquisition/source type |
| `renewal_date` | Renewal date |
| `plan_type` | Basic / Standard / Premium |
| `contract_type` | Monthly / Annual |
| `cancellation_date` | Cancellation date |
| `cancellation_reason` | Reason for cancellation |
| `monthly_charges` | Monthly subscription charge |
| `cltv` | Customer lifetime value |
| `churn_score` | Source-provided churn score |

## 3.3 Support Table — `db_support`

Contains customer-support interactions:

| Column | Description |
|---|---|
| `customerid` | Customer identifier |
| `complaint_date` | Complaint date |
| `escalations` | Escalation indicator |
| `csat_score` | Customer satisfaction score |
| `comment` | Support comment |

The project reference defines these three tables as the primary data sources for customer, subscription, and support analysis.

---

# 4. Analytical Approach

The analysis follows an end-to-end analytics pipeline:

```text
SQLite Database
       │
       ▼
Database & Schema Inspection
       │
       ▼
SQL Data Extraction
       │
       ▼
Data Quality Audit
       │
       ▼
Data Cleaning
       │
       ▼
Customer-Level Data Integration
       │
       ▼
Feature Engineering
       │
       ▼
Exploratory Data Analysis
       │
       ├───────────────┐
       ▼               ▼
Churn Drivers     Customer Risk
       │               │
       └───────┬───────┘
               ▼
       Revenue & CLTV Impact
               │
               ▼
      Retention Prioritization
               │
               ▼
      Business Recommendations
```

---

# 5. Database & SQL Analysis

The first stage connects Python to SQLite using `sqlite3`.

The database schema is inspected before data extraction to understand:

- Available tables
- Column names
- Data types
- Primary-key information
- Row counts
- Customer-level cardinality

Data is then imported into pandas using SQL queries.

Example:

```python
import sqlite3
import pandas as pd

conn = sqlite3.connect("customer_churn.db")

customer = pd.read_sql_query(
    "SELECT * FROM db_customer",
    conn
)

subscription = pd.read_sql_query(
    "SELECT * FROM db_subscription",
    conn
)

support = pd.read_sql_query(
    "SELECT * FROM db_support",
    conn
)
```

SQL is also used for grouped business analysis such as churn by contract type and cancellation reason.

---

# 6. Data Quality & Cleaning

Before performing analysis, the datasets are audited for quality issues.

The checks include:

- Missing values
- Duplicate customer IDs
- Duplicate support records
- Referential integrity
- Date parsing problems
- Categorical inconsistencies
- Invalid or unexpected values

## Customer Data Cleaning

Actions include:

- Renaming `name` to `customer_name`
- Removing fields outside the analytical scope
- Converting `dob` to datetime
- Standardizing gender categories
- Filling missing country values using observed state-country relationships

## Subscription Data Cleaning

Actions include:

- Converting subscription dates to datetime
- Standardizing acquisition/source categories
- Creating a binary churn flag
- Validating cancellation information

## Support Data Cleaning

Actions include:

- Converting complaint dates to datetime
- Standardizing escalation indicators
- Removing unused fields
- Aggregating support activity at customer level

---

# 7. Customer-Level Data Integration

A key analytical consideration is the different grain of the three tables.

The customer and subscription tables represent customer-level information, while the support table can contain multiple records for the same customer.

A direct join of raw support records could duplicate customer rows and inflate churn, revenue, complaint, and CLTV calculations.

Therefore, support interactions are aggregated first.

For each customer we calculate:

- Complaint count
- Escalation count
- Whether the customer has experienced an escalation
- Average CSAT
- Latest complaint date
- Latest escalation status
- Latest CSAT

The resulting dataset contains one analytical row per customer.

---

# 8. Feature Engineering

Several derived variables are created to support the analysis.

## Churn Flag

A customer is classified as churned when `cancellation_date` is populated.

```python
subscription["churn_flag"] = (
    subscription["cancellation_date"].notna().astype(int)
)
```

## Tenure

Tenure is calculated as:

```text
Cancellation Date - Subscription Start Date
```

for churned customers.

For active customers, tenure is calculated using a fixed analysis cutoff date.

A fixed cutoff is used instead of the current date to make the analysis reproducible.

## Customer Age

Customer age is calculated from date of birth and the analysis cutoff date.

## Cancellation Month

The cancellation date is converted into a monthly period to analyze when churn occurs.

## Churn Risk

The source `churn_score` is converted into three analytical tiers:

| Score | Risk |
|---:|---|
| < 50 | Low |
| 50–69 | Medium |
| ≥ 70 | High |

This is a segmentation of the supplied score rather than a newly trained predictive model.

---

# 9. KPI Framework

The project calculates a comprehensive KPI layer.

## Churn KPIs

### Churn Rate

```text
Churn Rate =
Churned Customers / Total Customers
```

### Retention Rate

```text
Retention Rate =
1 - Churn Rate
```

## Customer Value

### ARPU

The reference framework defines ARPU as:

```text
ARPU =
Sum of Monthly Charges / Active Customers
```

The notebook also shows the overall mean monthly charge to make denominator differences transparent.

### Average Tenure

```text
Average Tenure =
Average customer tenure in days
```

## Financial Exposure

### Churned Monthly-Charge Exposure

```text
Sum of monthly charges for churned customers
```

This is treated as an MRR/monthly-charge proxy rather than accounting revenue loss.

### CLTV Exposure

```text
Sum of CLTV associated with churned customers
```

## Support KPIs

### Average Complaints per Customer

```text
Total Complaints / Total Customers
```

### Escalation Rate

```text
Escalated Support Events / Total Support Complaints
```

### Escalation → Churn

Churn rates are compared between customers with and without support escalations.

---

# 10. Exploratory Analysis

The EDA is organized around the four business questions:

```text
WHO?
│
├── Contract
├── Plan
├── Acquisition
├── Geography
└── Risk Segment

WHY?
│
├── Cancellation Reason
├── Support Escalations
├── CSAT
└── Subscription Structure

WHEN?
│
├── Cancellation Month
└── Customer Tenure

WHAT IS THE IMPACT?
│
├── Monthly Charges
├── Revenue Exposure
└── CLTV
```

---

# 11. Churn by Contract Type

Contract structure is one of the most important dimensions in the analysis.

Observed churn rates:

| Contract Type | Churn Rate |
|---|---:|
| Monthly | **55.56%** |
| Annual | **8.33%** |

The difference is:

```text
55.56% - 8.33%
= 47.22 percentage points
```

The monthly-contract churn rate is approximately:

```text
55.56 / 8.33
≈ 6.67×
```

the annual-contract churn rate.

## Interpretation

Monthly subscribers represent a substantially higher observed churn-risk segment in this dataset.

A potential business hypothesis is to investigate whether suitable monthly subscribers can be encouraged to migrate to annual contracts through:

- Annual-plan incentives
- Loyalty benefits
- Bundled features
- Personalized retention offers

This should ultimately be validated using experimentation rather than assuming that contract migration itself causes retention.

---

# 12. Churn by Plan Type

Observed churn rates:

| Plan | Churn Rate |
|---|---:|
| Basic | **60.00%** |
| Standard | **22.22%** |
| Premium | **14.29%** |

## Interpretation

Basic has the highest observed churn rate.

Potential areas for investigation include:

- Pricing
- Content availability
- Feature limitations
- Perceived value
- Upgrade/downgrade behavior
- Competitor alternatives

The analysis does not establish that the Basic plan causes churn.

---

# 13. Churn by Acquisition Type

Observed churn rates include:

| Acquisition Type | Churn Rate |
|---|---:|
| Referral | **83.33%** |
| Paid | **16.67%** |
| Organic | **0.00%** |

Referral customers represent 5 of the 6 churned customers in the sample.

## Interpretation

The observed referral-channel churn rate is high and deserves investigation.

Potential questions include:

- Is the referral channel attracting lower-fit customers?
- Are referred customers receiving different expectations?
- Is onboarding different for referred customers?
- Are referral incentives affecting customer quality?
- Does the referral source itself require segmentation?

Because the dataset contains only 21 customers, this result should be treated as a hypothesis rather than a generalized channel-performance conclusion.

---

# 14. Geographic Churn

Churn is analyzed across:

- Country
- State

The geographic analysis compares:

- Customer count
- Churned customer count
- Churn rate
- Monthly-charge exposure

The reference project identifies Karnataka as the most affected state.

In this dataset, Karnataka has a 100% observed churn rate, but this represents only **2 customers**.

Therefore:

> The geographic signal is useful for investigation but should not be interpreted as representative of the broader market without a larger sample.

Potential investigation areas include:

- Regional pricing
- Service availability
- Customer-support performance
- Regional product issues
- Competitive conditions

---

# 15. Churn Timing

The project analyzes cancellation month to identify potential churn concentration.

The highest cancellation month in the dataset is:

**September 2024**

with:

**2 of the 6 churn events**

This creates an opportunity to investigate what happened around that period.

Potential questions:

- Were subscription prices changed?
- Were there product or technical issues?
- Were there content changes?
- Did support complaints increase?
- Did competitors introduce new offers?

Temporal concentration alone does not identify the cause.

---

# 16. Customer Tenure Analysis

Tenure is compared between retained and churned customers.

The reference project reports an average tenure of approximately:

**1,451 days**

The detailed notebook recalculates tenure using a reproducible analysis cutoff.

The purpose is to identify whether churn appears concentrated among newer or longer-tenured customers.

A shorter churned-customer tenure can support further investigation of:

- Onboarding
- First-value realization
- First renewal
- Early customer engagement
- Initial product experience

---

# 17. Cancellation Reason Analysis

Cancellation reasons provide direct descriptive evidence for the customer's stated reason for leaving.

The analysis calculates:

- Number of churned customers by reason
- Customer share
- Monthly-charge exposure
- CLTV associated with each reason

This allows the business to distinguish between:

```text
Most common reason
        vs.
Highest financial impact reason
```

Potential cancellation themes include:

- Competitor switching
- Pricing sensitivity
- Content dissatisfaction

The business can use this information to prioritize further root-cause analysis.

---

# 18. Customer Support Analysis

Support data is analyzed at customer level.

Metrics include:

- Complaint count
- Escalation count
- Escalation presence
- Average CSAT
- Latest CSAT

The analysis compares churn among customers:

```text
Customers with escalation
vs.
Customers without escalation
```

Observed result:

- 5 customers had at least one escalation
- All 5 of those customers churned
- 1 of 16 customers without an escalation churned

This represents a strong sample-level association.

## Analytical caution

This does **not prove that escalations caused churn**.

Possible explanations include:

```text
Poor service
    ↓
Customer dissatisfaction
    ↓
Escalation
    ↓
Cancellation
```

but also:

```text
Underlying dissatisfaction
       ↓
Higher churn risk
       ↓
More support interactions
       ↓
Escalation + cancellation
```

Additional timestamped behavioral data would be required to establish directionality.

---

# 19. Churn Risk Segmentation

The source dataset provides a `churn_score`.

The project converts this score into:

- Low Risk
- Medium Risk
- High Risk

The risk segmentation is combined with:

- Customer value
- Contract
- Plan
- Monthly charges
- CLTV
- Complaints
- Escalations

This produces a practical retention-prioritization view.

## Example retention logic

```text
High Churn Score
       +
High CLTV
       +
Support Escalation
       ↓
High Retention Priority
```

The purpose is to move from:

> "Who has high churn risk?"

toward:

> "Which high-risk customers deserve attention first?"

---

# 20. Revenue & CLTV Impact

Churn is translated into financial exposure.

The dataset contains:

- Monthly charges
- CLTV

The six churned customers account for approximately:

### Monthly-charge exposure

**73.94**

### Share of total monthly charges

**18.68%**

### Associated CLTV

**2,047**

These figures quantify the economic importance of churn.

## Interpretation

A business should not necessarily treat every churned customer equally.

Retention prioritization should consider:

```text
Churn Risk
     +
Customer Value
     +
Potential Recoverability
     +
Intervention Cost
```

This provides a foundation for a future retention decision engine.

---

# 21. Executive Insights

## Insight 1 — Overall churn is material

The dataset contains:

- 21 customers
- 6 churned customers
- 28.57% churn rate
- 71.43% retention rate

---

## Insight 2 — Monthly contracts show substantially higher churn

Monthly:

**55.56%**

Annual:

**8.33%**

This represents a **47.22 percentage-point difference**.

Contract structure should therefore be investigated as a major retention dimension.

---

## Insight 3 — Basic customers have elevated churn

Basic-plan churn is:

**60%**

compared with:

- Standard: 22.22%
- Premium: 14.29%

The business should investigate whether the Basic proposition is sufficiently competitive in price, content and features.

---

## Insight 4 — Referral customers show unusually high churn

Referral churn is:

**83.33%**

This is a strong signal in the sample but should be validated with a larger customer population.

---

## Insight 5 — Support escalation is strongly associated with churn

All five customers with escalations churned.

This makes support recovery a potentially important retention area.

However, the result should be treated as an association rather than a causal conclusion.

---

## Insight 6 — Churn has measurable financial exposure

The six churned customers account for:

- 73.94 monthly-charge exposure
- 18.68% of total monthly charges
- 2,047 associated CLTV

This demonstrates why churn analysis should be connected to customer value rather than focusing only on customer counts.

---

## Insight 7 — September 2024 represents a churn concentration

Two of the six churn events occurred in September 2024.

This should trigger investigation into product, pricing, support, content and competitive events during that period.

---

# 22. Business Recommendations

## 1. Investigate Monthly → Annual Migration

Analyze monthly customers with elevated churn scores and determine whether annual-plan incentives could improve retention.

Possible test:

```text
Control
Monthly Plan

vs.

Treatment
Annual Migration Offer
```

Measure:

- Conversion to annual
- Churn reduction
- Revenue impact
- CLTV impact

---

## 2. Build a Support-Recovery Workflow

Customers with:

- High churn score
- High CLTV
- Support escalation

can be routed to a priority recovery process.

```text
Escalation
    ↓
Risk + CLTV Evaluation
    ↓
Priority Customer
    ↓
Root-Cause Resolution
    ↓
Retention Follow-up
```

---

## 3. Investigate Basic-Plan Churn

Conduct deeper analysis of:

- Price/value perception
- Content consumption
- Feature usage
- Upgrade/downgrade behavior
- Competitor offerings

---

## 4. Audit Referral Customer Quality

Investigate whether referral acquisition is producing customers with different:

- Engagement
- Expectations
- Retention behavior
- Customer value

---

## 5. Investigate September 2024

Conduct a historical event analysis around September 2024.

Look for:

- Pricing changes
- Product releases
- Technical incidents
- Content changes
- Support spikes
- Competitor campaigns

---

## 6. Strengthen Early-Lifecycle Retention

Future analysis should incorporate:

- Login frequency
- Watch time
- Content consumption
- Session recency
- First-week engagement
- First-renewal behavior

This would allow the business to identify churn signals before cancellation occurs.

---

# 23. Limitations

The dataset is intentionally small and therefore requires careful interpretation.

### Sample Size

There are only **21 customers**.

Segment-level percentages can therefore be highly unstable.

### Predictive Modeling

The source database already contains `churn_score`.

This project segments the supplied score but does not claim to have built a new predictive model.

### Causality

Observed relationships such as escalation → churn are associations.

They do not prove that one variable caused the other.

### Financial Interpretation

Monthly-charge exposure is used as an MRR/monthly-charge proxy.

It should not automatically be interpreted as recognized accounting revenue loss.

### Missing Behavioral Data

The dataset does not contain rich OTT engagement data such as:

- Watch time
- Sessions
- Content consumption
- Login frequency
- Recency
- Engagement depth

These would materially improve churn diagnosis and prediction.

---

# 24. Future Enhancements

The current project establishes the foundation for a more advanced **Churn Prediction & Retention Decision System**.

## Phase 1 — Current Analytics

```text
SQL
 ↓
Data Quality
 ↓
Cleaning
 ↓
Feature Engineering
 ↓
EDA
 ↓
Segmentation
 ↓
Business Insights
```

## Phase 2 — Predictive Modeling

Potential models:

- Logistic Regression
- Decision Tree
- Random Forest
- XGBoost
- LightGBM

Evaluation:

- Precision
- Recall
- F1
- ROC-AUC
- PR-AUC
- Confusion Matrix

---

## Phase 3 — Explainable Churn

Use SHAP and feature importance to answer:

> Why is this customer considered high risk?

Example:

```text
Customer
   │
   ▼
High Churn Risk
   │
   ├── Monthly Contract
   ├── Low CSAT
   ├── Recent Escalation
   ├── Short Tenure
   └── Basic Plan
```

---

## Phase 4 — Retention Decision Engine

Move from:

> **Who will churn?**

to:

> **Who should we intervene with, why, and what action should we take?**

A decision engine could combine:

```text
Churn Probability
        +
Customer CLTV
        +
Churn Driver
        +
Intervention Cost
        +
Expected Retention Value
        ↓
Retention Priority
        ↓
Recommended Action
```

---

## Phase 5 — BI Dashboard

Potential dashboard implementation:

- Tableau
- Power BI
- Streamlit

Dashboard sections:

1. Executive Overview
2. Churn Trends
3. Contract Analysis
4. Plan Analysis
5. Acquisition Analysis
6. Geographic Analysis
7. Support Intelligence
8. Revenue at Risk
9. CLTV Exposure
10. High-Risk Customer Queue

---

# 25. Portfolio Value

This project demonstrates an end-to-end analytics workflow rather than isolated technical exercises.

### Technical Skills

- SQL
- SQLite
- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- Data Cleaning
- Feature Engineering
- Relational Data Integration
- Exploratory Data Analysis
- Customer Segmentation

### Business Analytics Skills

- Churn Analysis
- Retention Analytics
- Customer Intelligence
- Revenue-at-Risk Analysis
- CLTV Analysis
- Customer Support Analytics
- Risk Segmentation
- Business KPI Design
- Decision Support
- Business Recommendations

---

# 26. Final Business Takeaway

The analysis demonstrates that churn should not be viewed as a single KPI.

A useful customer-retention framework connects:

```text
WHO
Customer & Segment Risk
        │
        ▼
WHY
Churn Drivers & Support Signals
        │
        ▼
WHEN
Lifecycle & Timing
        │
        ▼
VALUE
Revenue & CLTV Exposure
        │
        ▼
WHAT
Retention Action & Prioritization
```

The most important observed signals in this dataset are the substantial difference between monthly and annual contract churn, elevated Basic-plan churn, high observed referral churn, the strong association between support escalation and churn, and the financial exposure associated with the six churned customers.

The next analytical step is to introduce richer behavioral data and build a validated predictive model and retention decision layer.
