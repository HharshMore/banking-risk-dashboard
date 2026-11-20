# Banking Operations & Risk Dashboard

An interactive 4-page Power BI dashboard built on a sample banking dataset (7 tables). It covers account balances, transactions, the loan portfolio, repayment progress and KYC compliance risk across 10 branches.

> **Note:** the data is a synthetic sample provided for learning. The figures show how the dashboard works and are not real bank performance.

**Tools:** Power BI Desktop, Power Query, DAX

## Dashboard pages

| Page | What it shows |
|---|---|
| **Overview** | Total balance, deposits, withdrawals, net cash flow, deposits vs withdrawals by branch, balance by account type, transactions |
| **Loans** | Active loans and outstanding amount, loans by type, active vs closed, loans by interest-rate band |
| **Customers & KYC** | KYC completion, KYC status, lending exposure to customers without completed KYC, a "who to chase first" list |
| **Branches** | Branch leaderboard by balance, accounts by balance band, accounts by type |

Page navigation buttons are on every page, and the Overview, Loans and Branches pages have slicers.

## Screenshots

![Overview](screenshots/01_Overview.png)
![Loans](screenshots/02_Loans.png)
![Customers & KYC](screenshots/03_KYC.png)
![Branches](screenshots/04_Branches.png)

## Data

| Table | Rows before | Rows after |
|---|---|---|
| Accounts | 1,054 | 1,000 |
| Loans | 564 | 500 |
| Transactions | 2,061 | 2,000 |
| Customers | 1,000 | 1,000 |
| KYC | 500 | 500 |
| Branches | 10 | 10 |
| Payments | 2,000 | 2,000 |

## Data cleaning (Power Query)

179 of 7,189 rows were removed (153 duplicates and 26 blank or incomplete records).

| Issue found | Fix | Rows affected |
|---|---|---|
| Duplicate account records | Removed duplicates on `account_id` | 43 |
| Account rows with no IDs | Filtered out null IDs | 11 |
| Duplicate loan records | Removed duplicates on `loan_id` | 49 |
| Loan rows with no `loan_id` | Filtered out null IDs | 14 |
| One incomplete loan record (ID 510: no customer, type or rate) | Removed | 1 |
| Duplicate transactions | Removed duplicates on `transaction_id` | 61 |
| Inconsistent text ("closed" vs "Closed") | Standardized `loan_status` capitalization | all loan rows |
| ID columns stored as decimals | Changed data type to whole number | 3 tables |
| Sensitive Aadhaar numbers | Removed the column from the model | KYC table |
| No readable branch names | Created a `Branch` label ("Branch 01"...) | 10 |

## Data model

A snowflake schema with two fact tables (Transactions, Payments) and six relationships, all single-direction:

- Customers to Accounts, Customers to Loans, Customers to KYC (one to one)
- Branches to Accounts
- Accounts to Transactions
- Loans to Payments

## Selected DAX measures

```
Active Loan Exposure (KYC Incomplete) =
CALCULATE(SUM(Loans[loan_amount]), Loans[loan_status] = "Active", KYC[kyc_status] <> "Completed")

Active Outstanding =
[Active Loan Amount] - CALCULATE([Total Payments Received], Loans[loan_status] = "Active")

Repayment Progress % = DIVIDE([Total Payments Received], [Total Loan Amount])

KYC Completion Rate = DIVIDE([KYC Completed], COUNTROWS(KYC))
```

Calculated columns: `Balance Band` (Accounts), `Rate Band` (Loans), `Branch` (Branches).

## Key findings (sample data)

- Total balances are about ₹25.60M, with Savings holding 64% and Current 36%.
- Deposits (₹2.74M) slightly exceed withdrawals (₹2.70M), a net inflow of about ₹46.6K.
- 246 of 500 loans (49%) are active, with about ₹59.8M still outstanding. Only about 4.6% of total loan value has been repaid.
- Only 500 of 1,000 customers have a KYC record, and 36% of those are completed.
- **Of 246 active loans, 194 (about ₹48.5M) belong to customers without completed KYC:** 74 are pending or under review, and 120 have no KYC record at all. This is the main compliance risk on the dashboard.
- Branch 01 leads with ₹2.86M in balances and Branch 10 trails at ₹2.26M.

## Limitations

- Synthetic data, so findings are illustrative.
- The dataset has no date fields, so there are no time-trend views.
- The `city` field holds localities rather than cities, so no map is included.

## Author

[Your Name] | [LinkedIn link]