# FinVista Credit Risk — Data Dictionary

*Prepared by: Data Engineering Team, FinVista Consumer Lending*  
*Snapshot date: 2026-06-20*

---

## How to Read This Document

Each table section lists every column with the following attributes:

| Attribute | Meaning |
|---|---|
| **Data Type** | Python/pandas dtype after loading |
| **Nullable** | Whether `NaN` / `NULL` can appear, the approximate rate, the **reason** it is missing, and the **recommended imputation strategy** |
| **Allowed Values / Range** | Valid values for categoricals; min–max bounds for numerics |
| **Units / Format** | Physical unit (e.g. GBP, months) or date format |
| **Business Definition** | What the field measures and why it matters for credit risk |

> **Imputation guidance** below is advisory. Data scientists should validate assumptions against the data and justify their chosen strategy.

---

## Table 1 — `customers.csv`

**Grain**: one row per customer  
**Row count**: 8,000  
**Primary key**: `customer_id`

---

### `customer_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. Primary key; must be present for all joins. |
| **Allowed Values / Range** | Format `C` followed by 6 digits, e.g. `C000001` – `C008000`. |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Unique identifier assigned to each customer at account opening. Used as the join key across all four tables. |

---

### `date_of_birth`

| Attribute | Detail |
|---|---|
| **Data Type** | `datetime64[ns]` (parse with `parse_dates=['date_of_birth']`) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Dates corresponding to ages 18–75 at snapshot date (2026-06-20), i.e. approximately 1951-06-20 to 2008-06-20. |
| **Units / Format** | `YYYY-MM-DD` |
| **Business Definition** | Customer's date of birth. Used to derive `age` at application. Age is a regulatory-sensitive variable in lending — lenders must not discriminate by age. |

---

### `age`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. Derived from `date_of_birth`. |
| **Allowed Values / Range** | 18 – 75 (years) |
| **Units / Format** | Years (integer) |
| **Business Definition** | Age of the customer in full years at the snapshot date (2026-06-20). Older customers with longer credit histories tend to have lower default rates, but age cannot be used directly as a model feature in markets with age-discrimination regulations. |

---

### `education`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `no_high_school`, `high_school`, `some_college`, `bachelors`, `graduate` |
| **Units / Format** | Categorical label |
| **Business Definition** | Highest educational qualification attained. Acts as a proxy for earning potential and financial literacy. Higher education levels are generally associated with higher income stability and lower default risk. |

---

### `employment_status`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `employed_full`, `employed_part`, `self_employed`, `unemployed`, `retired` |
| **Units / Format** | Categorical label |
| **Business Definition** | Employment situation at time of application. One of the strongest demographic predictors of default: unemployed applicants have no stable income stream to service debt. Self-employed income is more volatile than salaried income. |

---

### `annual_income`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | **Yes — ~4 % missing.** Missing because some applicants declined to provide income on the online form (voluntary field). Values are missing at random (MAR) — missingness is not correlated with income level. **Recommended fix**: impute with the median income segmented by `employment_status` (e.g. median income among `employed_full` customers), as income varies substantially by employment type. |
| **Allowed Values / Range** | £8,000 – £600,000 (plausible range). Values above ~£350,000 should be treated as outliers — cap at the 99th percentile before modelling. |
| **Units / Format** | GBP (£), annual gross |
| **Business Definition** | Self-reported gross annual income. Used to assess affordability: the ratio of loan repayments to income (Debt-to-Income ratio) is a core credit risk metric. Income is right-skewed — consider log-transforming for linear models. |

---

### `housing_status`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `own_outright`, `mortgaged`, `renting`, `living_with_family`, `other` |
| **Units / Format** | Categorical label |
| **Business Definition** | Current housing arrangement. Home ownership signals financial stability and asset accumulation. Customers living with family or renting may have less financial stability. Mortgaged customers already carry a large debt obligation — relevant when assessing additional borrowing capacity. |

---

### `months_at_address`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` (stored as float due to NaNs) |
| **Nullable** | **Yes — ~6 % missing.** Missing due to a form input bug in the online application system that was present for approximately 6 months; records from that period have no address tenure. Missingness is not correlated with risk profile. **Recommended fix**: impute with the median (approximately 36 months). Alternatively, create a binary `address_tenure_missing` flag and impute with 0. |
| **Allowed Values / Range** | 1 – 300 months |
| **Units / Format** | Months (integer stored as float) |
| **Business Definition** | Number of months the customer has lived at their current address. Residential stability is a weak positive signal — longer tenure is marginally associated with lower default rates. Also used for identity verification purposes. |

---

### `marital_status`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `single`, `married`, `divorced`, `widowed` |
| **Units / Format** | Categorical label |
| **Business Definition** | Marital status at time of application. Married customers may have dual incomes, which reduces default risk. Divorced customers may have elevated financial obligations (maintenance payments). Predictive signal is weak but may be captured via WoE encoding. |

---

### `n_dependents`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0 – 4 (integer) |
| **Units / Format** | Count of persons |
| **Business Definition** | Number of financial dependants (children or other dependants). More dependants reduce disposable income available for debt repayment. Useful in constructing an adjusted income-per-dependant feature. |

---

### `region`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `Northeast`, `Southeast`, `Midwest`, `Southwest`, `West` |
| **Units / Format** | Categorical label |
| **Business Definition** | UK geographic region of the customer's current address. Regional economic conditions (unemployment rates, house prices) can affect default rates. Note: using region as a model feature may create disparate-impact risk — regulatory fairness checks should be performed. |

---

### `months_as_customer`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 1 – 240 months |
| **Units / Format** | Months (integer) |
| **Business Definition** | Number of months the customer has held any active account with FinVista, measured at the snapshot date. Longer-tenured customers have more internal behavioural data available and are generally lower risk (adverse selection: riskier customers default and leave earlier). |

---

## Table 2 — `credit_history.csv`

**Grain**: one row per customer  
**Row count**: 8,000  
**Primary key / Foreign key**: `customer_id` → `customers.customer_id`

---

### `customer_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Must match a value in `customers.customer_id`. |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Foreign key linking credit bureau data to the customer record. |

---

### `credit_score`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` (stored as float due to NaNs) |
| **Nullable** | **Yes — ~3 % missing.** Missing because the bureau returned no score for thin-file customers (customers with very few or very recent credit accounts). Missingness is informative: thin-file customers tend to be higher risk than average. **Recommended fix**: (1) impute with the median score for thin-file customers (~580–600), OR (2) create a `credit_score_missing` binary flag and impute with 0 before scaling. Do not impute with the overall median without accounting for the thin-file signal. |
| **Allowed Values / Range** | 300 – 850 (integer stored as float) |
| **Units / Format** | Dimensionless score (higher = better creditworthiness) |
| **Business Definition** | Third-party bureau credit score at the time of application. The single strongest predictor of consumer default. Encodes the customer's full repayment history across all lenders. Highly correlated with `credit_utilization` and `n_delinquencies_24m` — consider multicollinearity when using linear models. |

---

### `n_open_accounts`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0 – 18 (integer) |
| **Units / Format** | Count of accounts |
| **Business Definition** | Total number of open credit accounts (credit cards, personal loans, car finance, etc.) across all lenders at origination. Very high counts may indicate over-leveraging; very low counts indicate a thin credit file. |

---

### `total_credit_limit`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | £500 – £200,000 |
| **Units / Format** | GBP (£), total across all open revolving accounts |
| **Business Definition** | Sum of all credit limits across open revolving accounts. Combined with `credit_utilization`, allows reconstruction of the outstanding revolving balance (`total_credit_limit × credit_utilization`). High total limits relative to income may indicate risk from potential future borrowing. |

---

### `credit_utilization`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0.0 – 1.0 (proportion; values outside this range indicate data error) |
| **Units / Format** | Ratio (dimensionless). 0.30 means 30 % of revolving credit is currently drawn. |
| **Business Definition** | Total revolving balance ÷ total revolving credit limit at the time of application. A utilisation above 0.70 is considered high and strongly associated with financial stress and elevated default probability. One of the top predictors in most consumer credit models. |

---

### `months_oldest_account`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 6 – 360 months |
| **Units / Format** | Months (integer) |
| **Business Definition** | Age of the oldest open credit account in months, measured at the snapshot date. A proxy for the length of the customer's credit history. Longer credit histories are associated with more stable repayment behaviour and lower default risk. Also known as "AAoOA" (Age of Oldest Open Account) in bureau terminology. |

---

### `n_delinquencies_24m`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0 – 12 (integer; most customers have 0) |
| **Units / Format** | Count of delinquency events |
| **Business Definition** | Number of times the customer was 30 or more days past due on any credit obligation in the past 24 months, across all lenders. A value of 1 or more roughly doubles the probability of default. The distribution is zero-inflated — consider creating a binary `any_delinquency` flag alongside the raw count. |

---

### `n_public_records`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0, 1, or 2 |
| **Units / Format** | Count of public records |
| **Business Definition** | Number of public derogatory records on the customer's credit file, including bankruptcies, County Court Judgements (CCJs), and Debt Relief Orders. Even a single public record is a strong negative signal. These are rare events — the distribution is highly skewed. |

---

### `n_inquiries_12m`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` (stored as float due to NaNs) |
| **Nullable** | **Yes — ~5 % missing.** Missing because enquiry data was not retrieved from the bureau for a subset of applications processed during a system migration window. Missingness is at random (MAR) with respect to risk level. **Recommended fix**: impute with the median number of enquiries (approximately 1). Alternatively, flag missing values and impute with 0 — the distinction matters because 0 enquiries is itself informative. |
| **Allowed Values / Range** | 0 – 15 (integer stored as float) |
| **Units / Format** | Count of enquiries |
| **Business Definition** | Number of hard credit enquiries from lenders in the past 12 months. Each enquiry indicates a credit application. Multiple recent enquiries ("enquiry clustering") signal that the customer may be in financial distress and shopping for credit from multiple sources simultaneously. |

---

### `months_since_deliq`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | **Yes — `NaN` for customers with no delinquency history. This is intentional and informative, NOT a data error.** A `NaN` value means the customer has never been delinquent, which is a positive signal. **Recommended fix**: do NOT impute as if this were missing data. Instead, (1) create a binary `ever_delinquent` flag (`0` if NaN, `1` otherwise), and (2) fill `NaN` with a large sentinel value (e.g. 999) to indicate "no delinquency ever". |
| **Allowed Values / Range** | 1 – 25 months (when not `NaN`) |
| **Units / Format** | Months (float) |
| **Business Definition** | Number of months since the customer's most recent delinquency event. More recent delinquencies indicate higher current credit stress. Used alongside `n_delinquencies_24m` — a customer with 5 delinquencies in month 23 is very different from one with 5 delinquencies in the past 3 months. |

---

### `mortgage_balance`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. Zero for non-mortgaged customers. |
| **Allowed Values / Range** | £0 – £800,000 |
| **Units / Format** | GBP (£), outstanding balance |
| **Business Definition** | Outstanding mortgage balance at the snapshot date. Zero for customers whose `housing_status` is not `mortgaged`. Represents a large existing debt obligation. High mortgage balance relative to income reduces the customer's capacity to service additional debt. |

---

## Table 3 — `transactions.csv`

**Grain**: one row per transaction  
**Row count**: ~480,000 (average ~60 transactions per customer over 18 months)  
**Foreign key**: `customer_id` → `customers.customer_id`

> **Important**: this table cannot be joined directly to `loans.csv`. It must first be aggregated to one row per `customer_id` (e.g. total spend, transaction count, category mix). See the business brief for details.

---

### `txn_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Format `T` followed by 8 digits, e.g. `T00000001`. |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Unique identifier for each transaction record. Not useful as a model feature; used only for deduplication and data integrity checks. |

---

### `customer_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Must match a value in `customers.customer_id`. |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Foreign key linking each transaction to its customer. After aggregation, this becomes the join key to the loans and customer tables. |

---

### `txn_date`

| Attribute | Detail |
|---|---|
| **Data Type** | `datetime64[ns]` (parse with `parse_dates=['txn_date']`) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 2024-12-20 to 2026-06-20 (18-month window ending at snapshot date) |
| **Units / Format** | `YYYY-MM-DD` |
| **Business Definition** | Date the transaction was posted. Used to compute recency features (e.g. days since last transaction, days since last cash advance) and trend features (e.g. spend in last 3 months vs prior 3 months). Reference date for all recency calculations is the snapshot date: **2026-06-20**. |

---

### `txn_amount`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | £1.00 – £25,000 per transaction |
| **Units / Format** | GBP (£), transaction value |
| **Business Definition** | Value of the individual transaction in pounds. When aggregated, total and average spend reflect the customer's financial activity level. Large individual transactions may be outliers — consider the maximum transaction amount as a separate feature. Spending patterns (category mix, volatility, trends over time) are informative about financial behaviour and potential stress. |

---

### `txn_category`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `grocery`, `dining`, `fuel`, `travel`, `entertainment`, `utilities`, `healthcare`, `shopping`, `cash_advance`, `other` |
| **Units / Format** | Categorical label (merchant category code group) |
| **Business Definition** | Category of the merchant where the transaction occurred. The most credit-risk-relevant category is `cash_advance`: withdrawing cash against a credit account is a strong indicator of liquidity stress. The fraction of spend on `cash_advance` relative to total spend is a useful derived feature. Category mix changes over time can also signal deteriorating financial health. |

---

### `channel`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `online`, `in_store`, `atm`, `mobile_app` |
| **Units / Format** | Categorical label |
| **Business Definition** | Channel through which the transaction was made. ATM transactions are closely associated with `cash_advance` category spending. A shift toward higher ATM or cash-advance usage over time may indicate increasing financial pressure. |

---

### `cash_stress_flag`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `0` or `1` |
| **Units / Format** | Binary flag |
| **Business Definition** | Set to `1` if the transaction is categorised as `cash_advance` AND the amount exceeds £500; otherwise `0`. Large cash advances are a particularly strong indicator of liquidity stress — the customer cannot meet immediate cash needs through normal means. Aggregate this flag per customer (e.g. count of stress events, fraction of transactions that are stress events) before joining to the model dataset. |

---

## Table 4 — `loans.csv`

**Grain**: one row per loan application  
**Row count**: 10,040 (including ~40 suspected duplicate applications — see notes below)  
**Primary key**: `loan_id`  
**Foreign key**: `customer_id` → `customers.customer_id`

> **Note**: a small number of rows are suspected duplicate applications (same `customer_id`, `application_date`, and `loan_amount`). These should be identified and deduplicated before modelling.

---

### `loan_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Format `L` followed by 7 digits, e.g. `L0000001`. |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Unique identifier for each loan application. Used as the primary key for this table. Note: after deduplication, each `loan_id` should appear exactly once. |

---

### `customer_id`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | Must match a value in `customers.customer_id`. A single customer can appear multiple times (multiple loan applications). |
| **Units / Format** | Alphanumeric identifier |
| **Business Definition** | Foreign key to the customer table. Because one customer can have multiple loans, `customer_id` is NOT unique in this table. The unit of observation for modelling is the loan application, not the customer. |

---

### `application_date`

| Attribute | Detail |
|---|---|
| **Data Type** | `datetime64[ns]` (parse with `parse_dates=['application_date']`) |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 2023-06-20 to 2026-06-20 (approximately 3 years of history) |
| **Units / Format** | `YYYY-MM-DD` |
| **Business Definition** | Date the customer submitted the loan application. Used for vintage analysis (grouping loans by origination quarter to assess whether model performance varies over time) and for computing customer tenure at application. |

---

### `loan_amount`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | £2,000, £5,000, £10,000, £15,000, £20,000, £30,000, or £50,000 (discrete product tiers) |
| **Units / Format** | GBP (£) |
| **Business Definition** | The amount of money the customer has applied to borrow. Larger loan amounts relative to income increase the debt burden and default risk. Use in conjunction with `annual_income` to construct the Debt-to-Income (DTI) ratio: a core credit risk feature. |

---

### `loan_purpose`

| Attribute | Detail |
|---|---|
| **Data Type** | `object` (string categorical) |
| **Nullable** | **Yes — ~2 % missing.** Missing because the online application form allowed customers to skip this optional field. Missingness is not correlated with risk level. **Recommended fix**: treat missing values as a distinct category `unknown` before applying WoE encoding. Do not impute with the mode, as the purpose carries genuine risk signal that differs from `unknown`. |
| **Allowed Values / Range** | `debt_consolidation`, `home_improvement`, `auto`, `medical`, `education`, `business`, `vacation`, `other` |
| **Units / Format** | Categorical label |
| **Business Definition** | The stated reason for the loan. Customers applying for `debt_consolidation` are already in financial stress and carry the highest default rate (~15 %). `education` and `home_improvement` loans tend to be lower risk. The purpose also helps contextualise repayment capacity — a loan for a depreciating asset (vacation) is riskier than one for an income-generating investment (education). |

---

### `term_months`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 12, 24, 36, 48, or 60 (discrete product terms) |
| **Units / Format** | Months (integer) |
| **Business Definition** | Agreed repayment term in months. Longer terms reduce the monthly repayment amount (improving affordability) but increase total interest paid and the period of exposure to income shocks. Useful in constructing the monthly repayment amount: `loan_amount × interest_rate / term_months` (simplified). |

---

### `interest_rate`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | 0.04 – 0.36 (i.e. 4 % – 36 % APR) |
| **Units / Format** | Annual proportion (e.g. `0.129` = 12.9 % APR) |
| **Business Definition** | Annual interest rate charged on the loan, set by FinVista's existing pricing model based primarily on the applicant's credit score. **Caution**: this variable is partly endogenous — it was set by our own (imperfect) underwriting rules, not by an independent process. Including it as a feature may cause leakage of the existing model's signal rather than capturing independent predictive information. Use with awareness of this limitation. |

---

### `default_flag`

| Attribute | Detail |
|---|---|
| **Data Type** | `int64` |
| **Nullable** | No — never null. |
| **Allowed Values / Range** | `0` (good — loan repaid or performing) or `1` (default — customer missed payments and loan was written off) |
| **Units / Format** | Binary integer |
| **Business Definition** | **This is the target variable.** A value of `1` means the loan defaulted at any point within its agreed repayment term. The overall default rate in this dataset is approximately **10 %**. This class imbalance should be addressed — consider `class_weight='balanced'` in sklearn estimators, or oversampling/undersampling techniques. Do not use accuracy as an evaluation metric (a model predicting all zeros achieves 90 % accuracy while being commercially useless). |

---

### `loss_given_default`

| Attribute | Detail |
|---|---|
| **Data Type** | `float64` |
| **Nullable** | No — never null. Zero for non-defaulted loans. |
| **Allowed Values / Range** | £0 for `default_flag = 0`; £600 – £47,500 for `default_flag = 1` (approximately 30 %–95 % of `loan_amount`) |
| **Units / Format** | GBP (£) |
| **Business Definition** | **For business impact scoring only — do NOT use as a model input.** The actual financial loss incurred by FinVista if the loan defaulted, after recoveries. This field is only known after a default event occurs (it is not available at the time of application). Using it as a model feature would constitute **target leakage**. Its sole purpose in this coursework is to calculate Expected Loss (EL) in Section 8: `EL = sum(default_flag × loss_given_default)` over the approved portfolio. |
