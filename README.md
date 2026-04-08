# HEALTHCARE-CLAIMS-FRAUD-COST-ANALYTICS-PLATFORM

Project Overview

This project focuses on analyzing healthcare claims data to identify fraudulent provider behavior and uncover cost patterns in insurance claims. The analysis is based on Medicare datasets, including inpatient, outpatient, beneficiary, and provider-level data.

The core business problem addressed is the rising cost of healthcare due to fraudulent claims. Fraudulent providers inflate claim amounts through overbilling, unnecessary procedures, and misuse of diagnosis codes, leading to increased insurance premiums and operational inefficiencies.

This project simulates a real-world healthcare analytics scenario where insurance companies aim to detect fraud, reduce unnecessary costs, and improve profitability without impacting genuine patient care.

Business Objectives
Identify suspicious providers exhibiting abnormal claim behavior
Analyze claim cost patterns across fraud and non-fraud segments
Detect key drivers of fraud (billing patterns, procedures, diagnosis usage)
Provide actionable strategies to reduce fraud without impacting legitimate operations
Project Structure

The solution includes:

Data ingestion from multiple sources (Inpatient, Outpatient, Beneficiary, Provider)
Data processing and transformation using Snowflake
Data storage using AWS S3
SQL-based analysis and feature engineering
Power BI dashboard for visualization and insights
Business recommendations and fraud detection strategies
1) Data Engineering & Pipeline
Why This Was Necessary

Healthcare datasets are large, complex, and distributed across multiple tables. A scalable data pipeline ensures reliable data integration, transformation, and analysis.

Architecture Overview
AWS S3 used for data storage
Snowflake used for data warehousing and transformation
Power BI used for visualization
Key Steps
Uploaded raw datasets to AWS S3
Created storage integration between AWS and Snowflake
Loaded data into Snowflake raw tables
Created processed tables combining inpatient and outpatient claims
Performed data type standardization and feature engineering

This pipeline ensured scalable and production-ready data processing.

2) Data Preparation & Transformation
Key Actions
Merged inpatient and outpatient data into a unified claims dataset
Joined claims with provider data to identify fraud labels
Created derived features such as:
Procedure Count per claim
Average claim amount
Restructured diagnosis data using transformation techniques for better analysis

This ensured that the dataset was structured for both analytical queries and dashboard reporting.

3) KPI & Performance Analysis
Core KPIs Calculated
Total Claims
Fraud Claims vs Non-Fraud Claims
Average Claim Amount
Average Fraud Claim vs Non-Fraud Claim
Total Providers and Fraud Providers
Fraud Rate (%)
Average Procedures per Claim

These KPIs were designed to quantify fraud impact, operational efficiency, and cost behavior.

4) Dashboard Design & Visualization Rationale

The dashboard was designed to answer critical business questions around fraud detection and cost optimization.

Report Snapshot

1. Average Claim by Fraud Status (Bar Chart)

Why this chart?
Bar charts are effective for comparing categorical groups such as fraud vs non-fraud.

Insight Delivered:

Fraudulent providers submit claims ~84% higher than non-fraud providers

Importance:

Directly highlights cost inflation due to fraud
Helps quantify financial impact
2. Average Procedures per Claim (Bar Chart)

Why this chart?
Used to compare operational behavior across categories.

Insight Delivered:

Fraud providers perform ~2.3x more procedures per claim

Importance:

Indicates overutilization of procedures
Helps detect abnormal medical behavior
3. Diagnosis Distribution Analysis

Why this chart?
Used to analyze frequency and cost across diagnosis categories.

Insight Delivered:

Fraud occurs within common diagnoses, not rare ones
Fraud providers charge 45%–91% higher for similar diagnoses

Importance:

Shows fraud is hidden within normal operations
Helps focus detection on cost patterns rather than rare cases
5) Key Business Insights
1. Fraud Concentration
Only ~9% of providers are fraudulent
But they contribute to ~38% of total claims

This indicates a high-impact small group problem.

2. Cost Inflation as Primary Driver

Fraud is driven by:

Higher claim amounts per case
Not higher number of claims
3. Procedure Inflation

Fraudulent providers:

Perform significantly more procedures per claim
Use procedures to inflate billing
4. Diagnosis-Based Fraud Behavior
Fraud occurs within common diagnoses
No reliance on rare or unusual cases
6) Business Recommendations
Fraud Detection Strategy
Identify providers with:
High average claim amounts
High procedure counts
Use behavioral metrics instead of rule-based filtering
Operational Strategy
Segment providers into:
Low risk
Medium risk
High risk
Focus audits on high-risk providers
Cost Optimization
Reduce unnecessary procedures
Monitor claim inflation patterns
Promote efficient providers
Revenue Protection
Avoid blanket restrictions
Apply targeted controls only on high-risk segments
Estimated Impact
Significant reduction in fraudulent claim costs
Improved operational efficiency
Better allocation of audit resources
Enhanced profitability without affecting genuine claims
Skills Demonstrated
SQL (Joins, Aggregations, Data Transformation)
Data Engineering (AWS S3, Snowflake Integration)
Data Modeling & Feature Engineering
Power BI Dashboard Development
Fraud Analytics & Behavioral Analysis
Business Intelligence & KPI Design
Project Significance

Healthcare fraud is a multi-billion dollar problem globally. This project demonstrates how data analytics, cloud infrastructure, and business intelligence tools can be combined to detect fraud patterns, optimize costs, and support strategic decision-making.

The focus was not just on identifying fraud, but on building a scalable, data-driven framework to monitor and reduce fraud while maintaining operational efficiency.
