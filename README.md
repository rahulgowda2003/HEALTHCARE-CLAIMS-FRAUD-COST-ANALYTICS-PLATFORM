Healthcare Claims Fraud & Cost Analytics Platform
Project Overview

This project focuses on analyzing healthcare claims data to detect fraudulent provider behavior and identify cost inflation patterns. The analysis is based on Medicare datasets, including inpatient, outpatient, beneficiary, and provider-level data.

Healthcare fraud leads to significant financial losses through overbilling, unnecessary procedures, and misuse of diagnosis codes. This project addresses that challenge by transforming raw claims data into structured insights and identifying high-risk patterns.

Project Objective

The primary objective of this project was to:

Extract and combine multi-source healthcare datasets
Identify suspicious providers based on abnormal claim behavior
Analyze cost differences between fraud and non-fraud claims
Provide actionable strategies to reduce fraud without impacting revenue

This project simulates a real-world scenario where insurance companies rely on data analytics to reduce fraud and improve operational efficiency.

Tools and Technologies
Python (Pandas)
SQL (Snowflake)
AWS S3
Power BI
DAX
Data Modeling & Transformation
Data Collection

The dataset was sourced from Kaggle and consists of multiple tables:

Beneficiary Data – Patient demographics and conditions
Inpatient Claims – Hospital admission claims
Outpatient Claims – Non-admission claims
Provider Data – Fraud labels (Yes/No)

These datasets were integrated to create a unified claims dataset for analysis.

Data Preparation

Since the data was distributed across multiple tables, transformation was required before analysis.

Key Steps
Merged inpatient and outpatient datasets
Joined claims with provider data to include fraud labels
Created derived features:
Procedure count per claim
Average claim amount
Restructured diagnosis columns for aggregation

These steps ensured data consistency and analytical reliability.

Data Engineering

To simulate a real-world pipeline:

Stored data in AWS S3
Integrated with Snowflake using external stages
Loaded raw data into warehouse tables
Created processed datasets for analysis

This ensured scalability and production-level data handling.

SQL Queries
Storage Integration
CREATE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::role'
STORAGE_ALLOWED_LOCATIONS = ('s3://health.care.data/Data/');
Create Stage
CREATE STAGE my_s3_stage
URL = 's3://health.care.data/Data/'
STORAGE_INTEGRATION = s3_int;
Load Data
COPY INTO provider_raw
FROM @my_s3_stage/Train.csv
FILE_FORMAT = my_csv_format;
Combine Claims
CREATE TABLE processed_data.claims_combined AS
SELECT * FROM inpatient_raw
UNION ALL
SELECT * FROM outpatient_raw;
Python Analysis
Load Data
import pandas as pd

beneficiary = pd.read_csv("beneficiary.csv")
inpatient = pd.read_csv("inpatient.csv")
outpatient = pd.read_csv("outpatient.csv")
provider = pd.read_csv("provider.csv")
Merge Data
claims = pd.concat([inpatient, outpatient], axis=0)
claims_provider = claims.merge(provider, on="Provider", how="left")
Feature Engineering
claims_provider['ProcedureCount'] = claims_provider[proc_cols].notna().sum(axis=1)
Diagnosis Transformation
diagnosis_long = claims_provider.melt(
    id_vars=['ClaimID','Provider','PotentialFraud','InscClaimAmtReimbursed'],
    value_vars=diag_cols,
    var_name='Diagnosis_Position',
    value_name='DiagnosisCode'
)
Dashboard Design

The dashboard was designed to provide a clear overview of fraud patterns and cost behavior.

Key Metrics Displayed
Total Claims
Fraud vs Non-Fraud Claims
Average Claim Amount
Fraud Rate (%)
Average Procedures per Claim
Total Providers and Fraud Providers
DAX Queries
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
Visualization Rationale
Average Claim by Fraud Status (Bar Chart)
Why used: Best for categorical comparison
Insight: Fraud claims are ~84% higher
Average Procedures per Claim (Bar Chart)
Why used: Compares behavioral differences
Insight: Fraud providers perform ~2.3x more procedures
Diagnosis Pattern Analysis
Why used: Understand fraud distribution
Insight: Fraud occurs in common diagnoses with inflated costs
Key Insights
~9% providers contribute to ~38% claims
Fraud is driven by:
High claim amounts
More procedures
Cost inflation in common diagnoses
Business Recommendations
Identify high-risk providers using behavioral metrics
Focus audits on high-value, high-procedure providers
Reduce unnecessary procedures
Segment providers instead of applying blanket restrictions
Key Outcomes
Identified high-impact fraud drivers
Enabled targeted fraud detection
Improved cost optimization strategies
Supported data-driven decision-making
Skills Demonstrated
SQL (Joins, Aggregations, Data Modeling)
Python (Pandas, Data Analysis)
Snowflake & AWS (Data Engineering)
Power BI (DAX, Dashboarding)
Business Intelligence & Fraud Analytics
Project Significance

This project demonstrates how data analytics can be used to detect fraud, optimize costs, and support strategic decision-making in healthcare systems.

It highlights the importance of combining data engineering, analytical thinking, and business understanding to solve real-world problems.
