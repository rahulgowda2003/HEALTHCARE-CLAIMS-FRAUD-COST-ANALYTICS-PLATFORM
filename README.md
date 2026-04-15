# Healthcare Claims Fraud & Cost Analytics Platform

## Project Overview

This project analyzes healthcare claims data to detect fraudulent provider behavior and identify cost inflation patterns. Fraud in healthcare leads to significant financial losses due to overbilling, unnecessary procedures, and misuse of diagnosis codes.

The objective is to identify suspicious providers, understand fraud drivers, and provide data-driven strategies to reduce fraud while maintaining operational efficiency.

---

## Tools and Technologies

* **Python (Pandas)**
* **SQL (Snowflake)**
* **AWS S3**
* **Power BI**
* **DAX**
* **Data Modeling**
* **Data Collection**
  
---

## Dataset Overview

The dataset consists of:

* 558,211 claims (Inpatient + Outpatient)
* 138,556 beneficiaries (patients)
* 5,410 providers (doctors/hospitals)

---

### The dataset consists of:

* Beneficiary Data – Patient demographics
* Inpatient Claims – Hospital admissions
* Outpatient Claims – Non-admission claims
* Provider Data – Fraud labels
* Data Preparation

## Key Steps

* Merged inpatient and outpatient datasets
* Joined claims with provider data
* Created features:
* Procedure count
* Average claim amount
* Restructured diagnosis columns
* Data Engineering
* Stored data in AWS S3
* Integrated with Snowflake
* Loaded into warehouse tables
* Created processed datasets

---

## SQL Queries

1. Data Engineering (AWS + Snowflake)

Snowflake Integration

* CREATE STORAGE INTEGRATION s3_int
TYPE = EXTERNAL_STAGE
STORAGE_PROVIDER = S3
ENABLED = TRUE
STORAGE_AWS_ROLE_ARN = 'arn:aws:iam::role'
STORAGE_ALLOWED_LOCATIONS = ('s3://health.care.data/Data/');

Create Stage

* CREATE STAGE my_s3_stage
URL = 's3://health.care.data/Data/'
STORAGE_INTEGRATION = s3_int;

Load Data

* COPY INTO provider_raw
FROM @my_s3_stage/Train.csv
FILE_FORMAT = my_csv_format;

Combine Claims

*CREATE TABLE processed_data.claims_combined AS
SELECT * FROM inpatient_raw
UNION ALL
SELECT * FROM outpatient_raw;

Final Dataset

* CREATE TABLE processed_data.final_claims AS
SELECT
    BeneID,
    ClaimID,
    Provider,
    PotentialFraud,
    TRY_TO_NUMBER(InscClaimAmtReimbursed) AS ClaimAmount
FROM processed_data.claims_with_fraud;

---

## Python Analysis

1. Data Loading (Python)
* import pandas as pd
beneficiary = pd.read_csv("Train_Beneficiarydata.csv")
inpatient = pd.read_csv("Train_Inpatientdata.csv")
outpatient = pd.read_csv("Train_Outpatientdata.csv")
provider = pd.read_csv("Train_provider.csv")

2. Dataset Understanding
* print("beneficiary:", beneficiary.shape)
print("inpatient:", inpatient.shape)
print("outpatient:", outpatient.shape)
print("provider:", provider.shape)

Insights:

- 558k total claims
= ~4 claims per patient
= Fraud is concentrated but high impact

3. Data Preparation
* Merge Claims
claims = pd.concat([inpatient, outpatient], axis=0)
Merge with Provider (Fraud Label)
claims_provider = claims.merge(provider, on="Provider", how="left")

4. Fraud Analysis (Python)

Claim Comparison

* claims_provider.groupby("PotentialFraud")["InscClaimAmtReimbursed"].describe()

Insight:

* Fraud claims are ~84% higher than normal claims

Claims Distribution

* claims_provider.groupby("PotentialFraud").size()

Insight

* Only 9% providers → 38% claims (fraud)

Claims per Provider

* providers_count = claims_provider.groupby("PotentialFraud")["Provider"].nunique()
* claims_count = claims_provider.groupby("PotentialFraud").size()
* claims_per_provider = claims_count / providers_count
  
Insight

* Fraud providers: ~420 claims/provider
* Non-fraud: ~70 claims/provider

5. Diagnosis Analysis

Reshape Data

*diagnosis_long = claims_provider.melt(
    id_vars=['ClaimID','Provider','PotentialFraud','InscClaimAmtReimbursed'],
    value_vars=diag_cols,
    var_name='Diagnosis_Position',
    value_name='DiagnosisCode'
)
* diagnosis_long = diagnosis_long.dropna(subset=['DiagnosisCode'])
Diagnosis Summary
diag_summary = diagnosis_long.groupby(
    ['PotentialFraud','DiagnosisCode']
).agg(
    ClaimCount=('DiagnosisCode','count'),
    AvgClaimAmount=('InscClaimAmtReimbursed','mean')
).reset_index()

Insights

* Fraud happens in common diagnoses
* Fraud providers charge 45%–91% higher

6. Procedure Analysis
* claims_provider['ProcedureCount'] = claims_provider[proc_cols].notna().sum(axis=1)
claims_provider.groupby('PotentialFraud')['ProcedureCount'].mean()

Insight

* Fraud providers perform 2.3x more procedures

7. Suspicious Provider Identification
* provider_summary = claims_provider.groupby(['Provider','PotentialFraud']).agg(
    TotalClaims=('ClaimID','count'),
    AvgClaimAmount=('InscClaimAmtReimbursed','mean'),
    AvgProcedures=('ProcedureCount','mean')
).reset_index()

---

## DAX Queries

* Average Claim = "$ " & 
ROUND(
    AVERAGE(FINAL_CLAIMS[CLAIMAMOUNT]),1)

* Average Fraud Claim = "$ " &
CALCULATE(
    ROUND(
    AVERAGE(FINAL_CLAIMS[CLAIMAMOUNT]),1),
    FINAL_CLAIMS[POTENTIALFRAUD] = "yes"
)

* Average Non Fraud Claim = "$ " &
CALCULATE(
    ROUND(
    AVERAGE(FINAL_CLAIMS[CLAIMAMOUNT]),1),
    FINAL_CLAIMS[POTENTIALFRAUD] = "no"
)

* Average Procedure = AVERAGE(FINAL_CLAIMS[PROCEDURECOUNT])

* Fraud % = CALCULATE(
    ROUND(
    DIVIDE(
    [Fraud Claims],
    [Total Claims],
    0),3
)*100)& " %"

* Fraud Claims = CALCULATE(
    DISTINCTCOUNT(FINAL_CLAIMS[CLAIMID]),
    FINAL_CLAIMS[POTENTIALFRAUD] = "yes"
)

* Fraud Providers = CALCULATE(
                            DISTINCTCOUNT(PROVIDER_RAW[PROVIDER]),
                            PROVIDER_RAW[POTENTIALFRAUD] = "yes"
)

* Non Fraud Claims = CALCULATE(
    DISTINCTCOUNT(FINAL_CLAIMS[CLAIMID]),
    FINAL_CLAIMS[POTENTIALFRAUD] = "no"
)

* Non Fraud Providers = CALCULATE(
                            DISTINCTCOUNT(PROVIDER_RAW[PROVIDER]),
                            PROVIDER_RAW[POTENTIALFRAUD] = "no"
)

* Total Claims = DISTINCTCOUNT(FINAL_CLAIMS[CLAIMID])

* Total Providers = DISTINCTCOUNT(FINAL_CLAIMS[PROVIDER])

- created a table named diagnosis_table combining all the diagnosis with potential fraud

* Diagnosis_Table = 
UNION(
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_1],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_2],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_3],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_4],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_5],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_6],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_7],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_8],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_9],
        "PotentialFraud", final_claims[PotentialFraud]
    ),
    SELECTCOLUMNS(final_claims,
        "DiagnosisCode", final_claims[ClmDiagnosisCode_10],
        "PotentialFraud", final_claims[PotentialFraud]
    )
)



---

## Visualization Rationale

![Dashboard_upload](https://github.com/rahulgowda2003/HEALTHCARE-CLAIMS-FRAUD-COST-ANALYTICS-PLATFORM/blob/main/Healthcare%20Claim%20Fraud%20%26%20Cost%20Analysis%20Overview%20Screenshot.png)

![Dashboard_upload](https://github.com/rahulgowda2003/HEALTHCARE-CLAIMS-FRAUD-COST-ANALYTICS-PLATFORM/blob/main/Chart%20Screenshot.png)

![Dashboard_upload](https://github.com/rahulgowda2003/HEALTHCARE-CLAIMS-FRAUD-COST-ANALYTICS-PLATFORM/blob/main/Solutions%20Screenshot.png)

---

Dashboard Insights

Key Findings

* 9% providers → 38% claims
* Fraud claims are ~84% higher

Fraud driven by:

* High claim value
* More procedures
* Not rare diagnoses

Business Recommendations

* Fraud Detection Strategy
* Identify providers with:
* High claim amount
* High procedure count
* Operational Strategy

Segment providers:

* Low risk
* Medium risk
* High risk
* Cost Optimization
* Reduce unnecessary procedures
* Monitor abnormal billing patterns
* Revenue Protection
* Avoid blanket restrictions
* Target only high-risk providers

Business Impact

* Reduced fraudulent claim exposure
* Improved audit efficiency
* Better cost control
* Maintained revenue from legitimate providers

Skills Demonstrated

* SQL (Joins, Aggregations, Data Modeling)
* Python (Pandas, Data Analysis)
* Snowflake (Data Warehousing)
* AWS S3 (Data Pipeline)
* Power BI (DAX, Dashboarding)
* Fraud Detection Analytics
* Business Intelligence & Decision-Making

This project demonstrates how data analytics can detect fraud, reduce costs, and support decision-making in healthcare systems.
