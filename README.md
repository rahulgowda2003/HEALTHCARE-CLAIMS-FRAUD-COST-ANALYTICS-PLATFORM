Healthcare Claims Fraud & Cost Analytics Platform
Project Overview

Healthcare fraud is a multi-billion dollar problem, driven by overbilling, unnecessary procedures, and misuse of diagnosis codes. This project analyzes large-scale Medicare claims data to identify fraudulent provider behavior and uncover cost inflation patterns.

The objective was not just to detect fraud, but to build a scalable analytics pipeline that combines data engineering, statistical analysis, and business intelligence to support decision-making.

Business Problem

Insurance companies face three critical challenges:

Which providers are likely committing fraud?
What behaviors drive inflated healthcare costs?
How can fraud be reduced without impacting genuine claims or revenue?

This project answers these questions using structured data analysis and dashboard-driven insights.

Dataset Overview
558,211 claims
138,556 patients
5,410 providers
~4 claims per patient

Fraud distribution:

9% providers → 38% total claims (high concentration risk)
Architecture & Data Flow
Kaggle Dataset
     ↓
AWS S3 (Data Storage)
     ↓
Snowflake (Data Warehouse)
     ↓
SQL Transformations
     ↓
Power BI (Dashboard & Insights)
1. Data Engineering Pipeline
AWS + Snowflake Integration
CREATE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::role'
STORAGE_ALLOWED_LOCATIONS = ('s3://health.care.data/Data/');
Create External Stage
CREATE STAGE my_s3_stage
URL = 's3://health.care.data/Data/'
STORAGE_INTEGRATION = s3_int;
Load Data
COPY INTO provider_raw
FROM @my_s3_stage/Train.csv
FILE_FORMAT = my_csv_format;
2. Data Preparation (Python)
import pandas as pd

beneficiary = pd.read_csv("beneficiary.csv")
inpatient = pd.read_csv("inpatient.csv")
outpatient = pd.read_csv("outpatient.csv")
provider = pd.read_csv("provider.csv")
Merge Claims
claims = pd.concat([inpatient, outpatient], axis=0)
claims_provider = claims.merge(provider, on="Provider", how="left")
3. Feature Engineering
Procedure Count
claims_provider['ProcedureCount'] = claims_provider[proc_cols].notna().sum(axis=1)
Diagnosis Reshaping
diagnosis_long = claims_provider.melt(
    id_vars=['ClaimID','Provider','PotentialFraud','InscClaimAmtReimbursed'],
    value_vars=diag_cols,
    var_name='Diagnosis_Position',
    value_name='DiagnosisCode'
)
4. Fraud Analysis
Claim Amount Comparison
claims_provider.groupby("PotentialFraud")["InscClaimAmtReimbursed"].describe()
Insight
Fraud claims are ~84% higher
Claims per Provider
providers_count = claims_provider.groupby("PotentialFraud")["Provider"].nunique()
claims_count = claims_provider.groupby("PotentialFraud").size()
claims_per_provider = claims_count / providers_count
Insight
Fraud providers: ~420 claims/provider
Non-fraud: ~70 claims/provider
Procedure Analysis
claims_provider.groupby('PotentialFraud')['ProcedureCount'].mean()
Insight
Fraud providers perform 2.3x more procedures
Diagnosis Analysis
diag_summary = diagnosis_long.groupby(
    ['PotentialFraud','DiagnosisCode']
).agg(
    ClaimCount=('DiagnosisCode','count'),
    AvgClaimAmount=('InscClaimAmtReimbursed','mean')
).reset_index()
Insight
Fraud occurs in common diagnoses
Charges are 45%–91% higher
5. SQL Data Modeling (Snowflake)
Combine Claims
CREATE TABLE processed_data.claims_combined AS
SELECT * FROM inpatient_raw
UNION ALL
SELECT * FROM outpatient_raw;
Final Dataset
CREATE TABLE processed_data.final_claims AS
SELECT
    BeneID,
    ClaimID,
    Provider,
    PotentialFraud,
    TRY_TO_NUMBER(InscClaimAmtReimbursed) AS ClaimAmount
FROM processed_data.claims_with_fraud;
6. Power BI Dashboard (DAX)
Total Claims = DISTINCTCOUNT(FINAL_CLAIMS[CLAIMID])
Fraud Claims =
CALCULATE(
    DISTINCTCOUNT(FINAL_CLAIMS[CLAIMID]),
    FINAL_CLAIMS[POTENTIALFRAUD] = "yes"
)
Fraud % =
DIVIDE([Fraud Claims], [Total Claims]) * 100
Average Claim =
AVERAGE(FINAL_CLAIMS[CLAIMAMOUNT])
7. Dashboard Insights
Key Findings
9% providers → 38% claims
Fraud claims are ~84% higher
Fraud driven by:
High claim value
More procedures
Not rare diagnoses
8. Business Recommendations
Fraud Detection Strategy
Identify providers with:
High claim amount
High procedure count
Operational Strategy
Segment providers:
Low risk
Medium risk
High risk
Cost Optimization
Reduce unnecessary procedures
Monitor abnormal billing patterns
Revenue Protection
Avoid blanket restrictions
Target only high-risk providers
9. Business Impact
Reduced fraudulent claim exposure
Improved audit efficiency
Better cost control
Maintained revenue from legitimate providers
Skills Demonstrated
SQL (Joins, Aggregations, Data Modeling)
Python (Pandas, Data Analysis)
Snowflake (Data Warehousing)
AWS S3 (Data Pipeline)
Power BI (DAX, Dashboarding)
Fraud Detection Analytics
Business Intelligence & Decision-Making
