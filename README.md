# FinVista Credit Risk — Business Brief

*Prepared by: Risk Strategy Team, FinVista Consumer Lending*

---

## Background

FinVista is a UK-based consumer lender offering personal loans between £2,000 and £50,000 with repayment terms of 1–5 years. We originate approximately 3,500 new loan applications per month through our online platform.

Our current underwriting process relies on a simple rules-based scorecard built five years ago. In the past 18 months our default rate has climbed from 7 % to over 10 %, costing the business an estimated **£4.2 million in unexpected credit losses**. The Risk Committee has approved a project to replace the existing scorecard with a machine-learning-based credit scoring model.

---

## What We Need From You

We need a predictive model that assigns each loan application a **probability of default** (PD). This score will be used to make approve/reject decisions. Our lending policy requires that we approve **at least 70 %** of applications — we cannot restrict volume further without losing market share.

We expect the data science team to:

1. Explore and understand the data thoroughly before modelling.
2. Build and compare multiple modelling approaches.
3. Select the best model and justify the choice clearly — we need to explain this to the Risk Committee, who are not data scientists.
4. Quantify the business impact: how much Expected Loss will the new model save compared to approving everyone?

---

## Definition of Success

The model will be deployed if it achieves **at least 25 % reduction in Expected Loss** at a 70 % approval rate. We also track the **Gini coefficient** internally as our standard measure of model discrimination.

Expected Loss for a portfolio is defined as:

$$\text{EL} = \sum_{\text{approved}} \mathbb{1}[\text{default}_i] \times \text{LGD}_i$$

where LGD (Loss Given Default) is the actual pound amount lost if customer $i$ defaults.

---

## Data Available

Our data team has extracted four tables from our systems for the period ending **20 June 2026**. Data files are in `../data/`. Full column definitions, data types, nullable rules, and allowed values are in `data_dictionary.md` in this folder.

| Table | File | Grain | Description |
|---|---|---|---|
| Customers | `customers.csv` | One row per customer | Demographics at time of application. |
| Credit Bureau | `credit_history.csv` | One row per customer | Bureau summary data at origination. |
| Transactions | `transactions.csv` | One row per transaction | 18 months of account activity — multiple rows per customer. |
| Loans | `loans.csv` | One row per loan application | Historical loan applications with outcome (`default_flag`). Use this to build and evaluate your model. |

---

## Important Caveats From the Data Team

- The data has **not** been cleaned. There are known missing values and data quality issues — identify and document them.
- The `transactions` table is at **transaction grain**. It must be aggregated to one row per customer before it can be joined to the loan-level data.
- Some customers have applied for more than one loan. Treat each **loan application** as an independent observation.
- The `loss_given_default` column in `loans.csv` is for **business scoring only**. Do not use it as a model input — it is only known after a default occurs, and using it would constitute target leakage.
- The snapshot date for all data is **2026-06-20**. Use this as the reference point for any recency calculations.
