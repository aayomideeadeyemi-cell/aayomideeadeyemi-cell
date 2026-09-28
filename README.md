# Digital Payments Transaction Analysis

Excel + SQL analysis identifying why digital payment transactions fail or reverse, and translating the findings into concrete recommendations for a payments provider.

> **Note on the data:** this project uses a synthetic practice dataset built to resemble a real digital payments scenario. It is not real transaction data — labeled here for transparency.

---

## Business Problem

A digital payments provider wants to understand why transactions are failing or reversing, and what can be done to reduce it.

**Deliverable:** a cleaned dataset, verified findings, and concrete recommendations.

---

## Data

931 transaction records with the following fields: Transaction_ID, Customer_ID, Transaction_DateTime, Payment_Channel, Network_Provider, Region, Merchant_Category, Amount, Fee, Status, Failure_Reason.

## Data Cleaning

Full detail is in the `Cleaning_Log` sheet of the workbook — summary below:

| Issue | Fix |
|---|---|
| 12 duplicate Transaction_IDs | Removed, keeping first occurrence |
| 15 label variants of 5 real payment channels (`USSD`/`Ussd`/`ussd`) | Standardized to 5 canonical categories |
| 10 label variants of 4 real statuses (`Success`/`SUCCESSFUL`/`success`) | Standardized to 4 canonical categories |
| Amount stored as currency text (`₦27,903.69`) for ~15% of rows | Stripped symbol/commas, converted to numeric |
| 7 negative Amount values | Confirmed as sign errors, converted to absolute value |
| 13 rows missing Amount | Dropped — core field, cannot be reliably reconstructed |
| 6 rows with invalid/future dates | Dropped as unrecoverable |
| 17 rows missing Region / Network_Provider each | Filled as `"Unknown"` |
| 21 rows missing Fee | Imputed using the median fee-rate (Fee/Amount) for that transaction's channel |
| 8 non-Failed transactions with a stray Failure_Reason | Cleared — a reason only makes sense on a Failed transaction |
| 20 Failed transactions missing a reason | Filled as `"Unspecified"` |

**Result:** 931 clean rows, 0 duplicates, 0 missing values.

All findings were cross-verified in MySQL against the cleaned Excel data — see `digital_payments_analysis.sql`.

---

## Key Findings

### 1. USSD is the dominant failure channel — across every network provider
| Channel | Transactions | Failure Rate |
|---|---|---|
| USSD | 273 | **29.7%** |
| POS | 225 | 8.9% |
| Bank Transfer | 75 | 8.0% |
| Mobile App | 231 | 7.4% |
| Web | 127 | 3.9% |

USSD fails at roughly **4–8x** the rate of every other channel, and this holds consistently across all 4 network providers (28–33% failure each) — confirming it's a channel/protocol issue, not one carrier's network problem.

### 2. USSD failures cluster sharply around midday
**40% of all USSD failures (51 of 129)** occur in the 12pm–2pm window alone, consistent with network congestion during peak lunch-hour usage.

### 3. High-value transactions reverse at 9x the normal rate
Transactions above ₦50,000 reverse at **15.7%**, versus **1.7%** for transactions at or below that threshold.

---

## Recommendations

1. **Investigate the USSD gateway/protocol directly**, rather than pursuing carrier-specific fixes — the failure pattern is consistent across every network provider, pointing to a shared USSD-side issue.
2. **Review USSD gateway capacity for the 12pm–2pm window** specifically — congestion during this peak accounts for 40% of all USSD failures.
3. **Audit the fraud/risk-check process for high-value transactions** — a 9x reversal rate suggests either a genuinely higher-risk segment worth monitoring, or an overly aggressive check worth tuning to reduce false positives.

---

## Tools Used
- **Excel** — data cleaning, PivotTable exploration
- **MySQL** — verification queries confirming all findings independently of Excel

## Files
- `digital_payments_analysis.xlsx` — Raw_Data, Cleaned_Data, Cleaning_Log, and Findings & Recommendations sheets
- `digital_payments_analysis.sql` — table creation, data load, and verification queries
