# 📊 Spring Financial - Compliance & Marketing Analysis

## 🏢 Company Context

**Spring Financial** is a Canadian financial services company offering personal loans and credit-building products. To support **regulatory compliance** and **targeted marketing efforts**, the data team was tasked with preparing a clean, reliable dataset from scattered sources.

This repository contains an **Alteryx workflow** that ingests, cleans, integrates, and transforms raw customer data from multiple sources into a **final analytical dataset** ready for use by the compliance and marketing teams.

---

## 🎯 Project Objective

✅ Build an Alteryx workflow that:

- Loads and processes 3 source datasets:
  - `Loan_Applications.csv`
  - `Payment_History.csv`
  - `Contact_Preferences.csv`
- Cleans and deduplicates each dataset.
- Joins data into a unified analytical view.
- Derives new fields for compliance flagging and marketing scoring.
- Outputs the dataset to `Compliance_Marketing_Analysis.csv`.

---

## 📁 Input Datasets

### 1. `Loan_Applications.csv`
| Column | Description |
|--------|-------------|
| `Customer_Name` | Full name |
| `Email` | Primary key |
| `Loan_Amount` | May be 0 or negative |
| `Annual_Income` | May be missing or 0 |
| `Credit_Score` | Range: 300–850, may be missing/invalid |
| `Application_Status` | Approved, Pending, Rejected, Incomplete |
| `Application_Date` | May be invalid |

### 2. `Payment_History.csv`
| Column | Description |
|--------|-------------|
| `Email` | Primary key |
| `Loan_ID` | Unique loan ID |
| `Payment_Amount` | May be <= 0 |
| `Payment_Date` | May be missing/invalid |
| `Days_Late` | May be null or negative |

### 3. `Contact_Preferences.csv`
| Column | Description |
|--------|-------------|
| `Email` | Primary key |
| `Marketing_Consent` | Yes, No, Unknown |
| `Preferred_Channel` | Email, Phone, SMS, None |

---

## 🧼 Data Cleaning Steps

### 🟦 Loan_Applications

- Remove rows where:
  - `Application_Status = 'Incomplete'`
  - `Loan_Amount <= 0`
  - `Annual_Income <= 0`
  - `Credit_Score < 300` or is null
  - `Application_Date < 2020-01-01` or invalid
- Deduplicate by `Email`, keeping the record with the **most recent Application_Date**.

### 🟩 Payment_History

- Remove rows where:
  - `Payment_Amount <= 0`
  - `Payment_Date` is null or < `2020-01-01`
- Deduplicate by `Email + Loan_ID`, keeping record with **latest Payment_Date**.

### 🟨 Contact_Preferences

- Filter out rows where:
  - `Marketing_Consent = 'Unknown'`
  - `Preferred_Channel` is missing or `"None"`
- Deduplicate by `Email`, prioritizing:
  - `Marketing_Consent = 'Yes'` if available
  - Otherwise, keep the first record

---

## 🔗 Joins & Integration

- Left join: `Loan_Applications` ➕ `Payment_History` on `Email`
- Left join result ➕ `Contact_Preferences` on `Email`
- Preserve all loan applications (even if no match in other datasets)

---

## 🧮 Derived Fields

| Field | Logic |
|-------|-------|
| `Loan_to_Income_Ratio` | `Loan_Amount / Annual_Income` |
| `Compliance_Flag` | `"Non-Compliant"` if `Loan_to_Income_Ratio > 0.4` or `Credit_Score < 500`, else `"Compliant"` |
| `Marketing_Score` | Calculated based on multiple rules below |
| `Days_Since_Application` | `DateDiff('05-30-2025', Application_Date)` |

### Marketing Score Formula:
- Start at 50
- `+30` if `Marketing_Consent = 'Yes'`
- `+20` if `Credit_Score >= 700`
- `-10` if `Days_Late > 5` (treat null as 0)
- `+10` if `Preferred_Channel in ('Email', 'SMS')`
- Clamp final score between **0 and 100**

---

## 📦 Final Output Fields

- `Record_ID` (from Loan_Applications)
- `Customer_Name`
- `Email`
- `Loan_Amount`
- `Annual_Income`
- `Credit_Score`
- `Loan_to_Income_Ratio`
- `Compliance_Flag`
- `Marketing_Score`
- `Loan_ID`
- `Payment_Amount`
- `Days_Late`
- `Marketing_Consent`
- `Preferred_Channel`
- `Days_Since_Application`

---

## 🧾 Output & Reporting

- Final dataset is sorted by:
  1. `Marketing_Score` DESC
  2. `Credit_Score` DESC
  3. `Days_Since_Application` ASC
- Saved as:
  - `Compliance_Marketing_Analysis.csv`

---

## 📐 Workflow Design (Alteryx Overview)

```text
INPUT ➝ CLEAN ➝ DEDUP ➝ JOIN ➝ DERIVE ➝ SELECT ➝ SORT ➝ OUTPUT
