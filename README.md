# Table of Content
* [1.0 Executive Summary](https://github.com/skddedm/Retail-Store-Account-Receivable-Analysis/blob/main/README.md#10-executive-summary)
* [2.0 Methodology](https://github.com/skddedm/Retail-Store-Account-Receivable-Analysis/blob/main/README.md#20-methodology)
* [3.0 Project Outcomes](https://github.com/skddedm/Retail-Store-Account-Receivable-Analysis/blob/main/README.md#30-project-outcomes)
* [4.0 Details of Key Objective Areas](https://github.com/skddedm/Retail-Store-Account-Receivable-Analysis/blob/main/README.md#40-details-of-key-objective-areas)
* [5.0 Strategy Recommendations](https://github.com/skddedm/Retail-Store-Account-Receivable-Analysis/blob/main/README.md#50-strategy-recommendations)




# 1.0 Executive Summary
The credit phenomenon has many advantages for both the creditor and the debtor if managed within defined, understood and followed guidelines limiting the major risk of default and undue extended payment periods.

The Rock Stores, selling building and home supplies, have been selected as an imaginary store for this analysis. 

### 1.1 Problem Statement:
The store has continuous working capital problems resulting from cash flow mismatch between account payable, account receivable and regular expenses.

### 1.2 Project Objective:
The objective of this project is to optimize working capital for the store through adequate timeline and debt capacity. The key determinants of this objective are to determine:
* The level of credit sales capacity to the total sales level compared to industry
* Debt turnover ratio (DTO) to determine liquidity planning
* Credit period and credit limit compliance to policy
* Supplier and category concentration risk  
* Customer repayment performance through overdue payment analysis
* Accurateness of the outstanding debt balances
* Internal control weaknesses and possible fraud through duplicate transactions

### 1.3 Project Scope:
* 4 tables were used: debtors_list, sales, pmt_received and outstanding debt
* The database includes 41 customers
* Customers were grouped under 8 categories based the nature of their business activities 
* Approved credit limit to be granted to a customer is between 5% - 50% of customer total purchase based on the customer category
* Approved credit period to be granted to a customer is between 15 - 60 days based on customer category
* 73 sales transactions were used
* Sales type is limited to credit or cash sales only 

### 1.4 Tools Deployed:
* Microsoft Excel
* Power Query
* SQL
* Power BI

### 1.5 Summary of Project Findings:
After anlayses, 7 strengths and 4 weaknesses were identified. The store balanced a strong accounts receivable management system with capitalizing gains from the credit sales to boost income and profit. The 3 highlights of the 4 weaknesses are the to resolve the accounts receivable management software producing inaccurate balances and accepting duplicate record, manage the 34 customer exceeding their credit approved and diversify 36% customer concentration risk identified. Strategies, management impact and possible risks were recommended to resolve all 4 weaknesses to return the store a robust account receivable management levels. 

# 2.0 Methodology
The following methods were deployed:
* 4 tables were created for this analysis: debtors_list, sales, pmt_received,  and outstanding_debt
* The debtors_profile includes the columns: customer_id, customer_name, customer_category, province_name, 	date_of_first_purchase, approved_credit_limit_to_sales, approved_credit_period 
* The sales table includes columns such as: customer_id, order_id, transaction_date, customer_date, invoice_no., 	type_of_sales, invoice_amt, expected_pmt_date
* Pmt_received table includes: customer_id, pmt_date, customer_name, invoice_no., paid_amt
* The outstanding_debt includes: customer_id, customer_name, invoice_no., debt_balance
* The various key indicators defined in the object statement were manipulated using SQL and analysed with Power Query 

```SQL
select * from outstanding_debt;
select * from debtors_list;
select * from pmt_received;
select * from sales;

-- CREDIT SALES CAPACITY AND DEBT TUROVER RATIO
select type_of_sales, round(sum(sales_amount), 2) as sales_amount from sales group by type_of_sales;

select round(sum(debt_balance), 2) as current_debt_balance from outstanding_debt where debt_balance != 0;

-- GRANTED CREDIT ADHERENCE TO CREDIT PERIOD AND CREDIT LIMIT POLICY
select debtors_list.customer_id, debtors_list.customer_name, debtors_list.allowed_credit_limit_to_sales, sales.type_of_sales, round(sum(sales.sales_amount), 2) as amount from debtors_list
left join sales on debtors_list.customer_id = sales.customer_id
group by customer_id, customer_name, allowed_credit_limit_to_sales, type_of_sales;

select sales.customer_id, sales.customer_name, debtors_list.allowed_credit_limit_to_sales,
round(
		sum(case
				when type_of_sales = 'credit'
				then sales_amount
				else 0
            end)
		/
        sum(sales_amount), 2
) as credit_sales_ratio 
from sales
left join debtors_list on debtors_list.customer_id = sales.customer_id
group by customer_id, customer_name, debtors_list.allowed_credit_limit_to_sales;
	

select distinct debtors_list.customer_id, debtors_list.customer_name, debtors_list.allowed_credit_period, datediff(sales.transaction_date, expected_payment_date) as granted_period from debtors_list
left join sales on debtors_list.customer_id = sales.customer_id;

-- SUPPLIER AND CATEGORY CONCENTRATION
select distinct sales.customer_id, sales.customer_name, round(sum(sales.sales_amount), 2) as sales_amount from sales
group by customer_id, customer_name;

select debtors_list.customer_dept, round(sum(sales.sales_amount), 2) as sales_amount from sales
left join debtors_list on sales.customer_id = debtors_list.customer_id
group by customer_dept;

-- OVERDUE PAYMENT ANALYSIS
select sales.customer_id,sales.customer_name, datediff(sales.transaction_date, current_date()) as overdue_days from sales
left join outstanding_debt on sales.invoice = outstanding_debt.invoice;

-- ACCURATENESS OF OUTSTANDING DEBT
select sales.customer_id, sales.customer_name, sales.invoice, sales.sales_amount, (sales.sales_amount - pmt_received.paid_amt) as expected_debt_balance, outstanding_debt.debt_balance from sales
left join pmt_received on sales.invoice = pmt_received.invoice
left join outstanding_debt on sales.invoice = outstanding_debt.invoice
where debt_balance!=0;

-- DUPLICATE DEBTOR LIST, SALES, PAYMENT, OUTSTANDING BALANCE TRANSACTIONS
select count(customer_id), customer_name from debtors_list group by customer_name;

select customer_id, customer_name, count(order_id) as order_count, count(invoice) as invoice_count, sales_amount from sales group by customer_id, order_id, customer_name, invoice, sales_amount;

select customer_id, customer_name, count(invoice) as invoice_count, paid_amt from pmt_received group by customer_id, customer_name, invoice, paid_amt;

select customer_id, customer_name, count(invoice) as invoice_count, debt_balance from outstanding_debt group by customer_id, customer_name, invoice, debt_balance;

```
* Dashboards summary was created using Power BI with descriptive interpretations.
* Recommendations, management impact and risks to be considered were suggested to aid resolve weaknesses discovered after the analysis

2.1 Data Limitations 
* Simulated Data: The datasets were generated by simulation and may not fully capture the complexity, seasonality, behavioural patterns of real-world retail customer transactions and payment behaviour
* Limited Time Horizon: Analysis based on a single period (FY2025) may not capture seasonality or long-term trends, payment behaviour over multiple periods and seasonal fluctuations. 
* Simplified Credit Terms and Payment Structures: Credit terms, payment amounts, and settlement behaviour are standardized and may not reflect negotiated customer-specific agreements, partial payments, or instalment-based settlements commonly observed in practice.
* Exclusion of External Factors: The datasets do not incorporate external influences such as economic conditions, customer credit ratings, dispute resolution processes, or supply chain constraints that can materially impact accounts receivable performance.

# 3.0 Project Outcomes
Based on the project objective, the following key performance areas was discovered: 

### 3.1 Strengths
* The business credit sales is about 50% of its total sales value for the FY 2025
* Even though the percentage of credit sales is high in proportion to total sales, the debt turnover ratio (DTO) is about 31.93 indicating a faster collection and good credit management policy
* Every customer was granted credit period in compliance to the approved credit period policy
* No account receivable record for all customers was duplicated
* No sales transaction was duplicated
* No outstanding debt record was duplicated
* 4 out of the 8 categories make up about 45% of the store’s sales income

### 3.2 Weaknesses
* Out of the 41 customers, 34 customers exceeded their credit approved limit by 1 to 73%
* 10 customers make up 36% of total sales of the store showing customer concentration risk
* Debt balance shows overdue periods from 35 to 399 days but those overdue amounts exist as a result of inaccurate debt balances of $250 to $750 sitting on customer’s account receivable accounts. No customer has overdue amounts.
* 98 out of 445 payments transactions is duplicated representing 28.04% and $1.58m.

The dashboard below summarizes the findings of the analysis. 
<img width="862" height="477" alt="image" src="https://github.com/user-attachments/assets/0237cd11-3ac3-4336-99ad-3b1d24167be9" />

# 4.0 Details of Key Objective Areas
After the account receivable data was analysed in line with the stated objectives, the following information gives details:

### 4.1 Credit sales level and debt turnover ratio (DTO)
The credit sales of the business is about 50% of its total sales value for the FY 2025. Even though the percentage of credit sales is high in proportion to total sales, the debt turnover ratio (DTO) is 31.93 indicating a faster collection and good credit management policy.
| Metric | Value |
|---|---|
| % of Credit Sales to Total Sales | 50% |
| Debt Turnover Ratio | 31.93 |

### 4.2 Credit period and credit limit compliance to policy
Every customer was granted credit period in compliance to the approved policy. The granted period was calculated deducting the transactions date from the expected payment date. No customer granted credit period violated the policy. Some of the information of customers is represented in the table below:
| Customer ID | Customer Name | Allowed Credit Period | Granted Period |
|---|---|---|---|
| C001 | Maple Ridge Construction Ltd. | 45 | -45 |
| C002 | Northern Peak Builders Inc. | 45 | -45 |
| C004 | Ironclad General Contractors | 45 | -45 |
| C006 | Prairie West Construction Group | 30 | -30 |
| C008 | Polaris Electrical Services | 30 | -30 |
| C009 | RedSeal Plumbing & Heating | 15 | -15 |
| C010 | ArcticFlow HVAC Solutions | 60 | -60 |
| C011 | BrightLine Electrical Contractors | 45 | -45 |
| C012 | Cascade Mechanical Services | 15 | -15 |
| C013 | Apex Climate Control | 60 | -60 |
| C014 | Evergreen Property Management | 60 | -60 |
| C015 | UrbanCore Realty Services | 30 | -30 |
| C016 | Northern Star Property Group | 15 | -15 |
| C018 | Keystone Facilities Management | 45 | -45 |
| C020 | Prairie Bloom Lawn & Garden | 30 | -30 |

Out of the 41 customers, 34 customers exceeded their credit approved limit by 1 to 73%. The actual credit sales ratio for each customer is the proportion of credit sales to total sales calculated for the same customer. 
| Customer ID | Customer Name | Allowed Credit Limit to Sales | Actual Credit Sales Ratio | Difference |
|---|---|---|---|---|
| C001 | Maple Ridge Construction Ltd. | 40% | 53% | -13% |
| C002 | Northern Peak Builders Inc. | 40% | 70% | -30% |
| C004 | Ironclad General Contractors | 40% | 47% | -7% |
| C006 | Prairie West Construction Group | 40% | 44% | -4% |
| C008 | Polaris Electrical Services | 10% | 55% | -45% |
| C009 | RedSeal Plumbing & Heating | 10% | 56% | -46% |
| C010 | ArcticFlow HVAC Solutions | 10% | 44% | -34% |
| C011 | BrightLine Electrical Contractors | 10% | 36% | -26% |
| C012 | Cascade Mechanical Services | 10% | 83% | -73% |
| C013 | Apex Climate Control | 10% | 43% | -33% |
| C014 | Evergreen Property Management | 35% | 37% | -2% |
| C015 | UrbanCore Realty Services | 35% | 36% | -1% |
| C016 | Northern Star Property Group | 35% | 36% | -1% |
| C018 | Keystone Facilities Management | 35% | 69% | -34% |
| C020 | Prairie Bloom Lawn & Garden | 15% | 88% | -73% |
| C021 | Evergreen Outdoor Solutions | 15% | 76% | -61% |
| C022 | Summit Grounds Maintenance | 15% | 48% | -33% |
| C023 | Northern Roots Landscaping | 15% | 36% | -21% |
| C024 | Cornerstone Hardware Installers | 10% | 80% | -70% |

### 4.3 Determine supplier and category concentration risk  
4 out of the 8 categories make up about 45% of the store’s sales income. 10 customers make up 36% of total sales of the business showing customer concentration risk. This makes a fair spread across customers and categories without concentration risk. The data is captured below:
| Customer ID | Customer Name | Sales Amount | % to Total |
|---|---|---|---|
| C017 | Skyline Commercial Properties | 234,777.39 | 4% |
| C028 | TrueFix Repair & Renovation | 223,793.84 | 4% |
| C020 | Prairie Bloom Lawn & Garden | 214,315.39 | 4% |
| C035 | SteelFrame Installations | 211,773.40 | 4% |
| C008 | Polaris Electrical Services | 204,168.28 | 4% |
| C018 | Keystone Facilities Management | 199,573.23 | 4% |
| C033 | Precision Tile & Flooring | 199,359.24 | 4% |
| C026 | FixPro Maintenance Services | 199,324.46 | 4% |
| C011 | BrightLine Electrical Contractors | 197,201.17 | 3% |
| C039 | MapleLeaf School Facilities | 185,133.87 | 3% |

### 4.4 Customer overdue payment analysis
Debt balance shows overdue periods from 35 to 399 days but those overdue days and amounts exist as a result of inaccurate debt balances of $250 to $750 sitting on customer’s accounts. No customer has actual overdue amounts and there should not be any overdue amounts or overdue days. The error could be from weak process controls, account receivable software logical failure to calculate accurate balance or both. Some of the overdue customer days and amounts are captured in the table below:
| Customer ID | Customer Name | Overdue Days |
|---|---|---|
| C033 | Precision Tile & Flooring | -399 |
| C022 | Summit Grounds Maintenance | -398 |
| C009 | RedSeal Plumbing & Heating | -395 |
| C018 | Keystone Facilities Management | -394 |
| C021 | Evergreen Outdoor Solutions | -393 |
| C001 | Maple Ridge Construction Ltd. | -392 |
| C030 | Atlas Facility Services | -392 |
| C028 | TrueFix Repair & Renovation | -391 |
| C020 | Prairie Bloom Lawn & Garden | -389 |
| C028 | TrueFix Repair & Renovation | -388 |
| C030 | Atlas Facility Services | -388 |

| Customer ID | Customer Name | Invoice | Sales Amount | Expected Debt Balance | Actual Debt Balance |
|---|---|---|---|---|---|
| C001 | Maple Ridge Construction Ltd. | INV-213528 | 8,991.58 | 0 | 750 |
| C001 | Maple Ridge Construction Ltd. | INV-942663 | 7,352.72 | 0 | 750 |
| C002 | Northern Peak Builders Inc. | INV-623921 | 19,484.54 | 0 | 750 |
| C003 | TrueNorth Renovations | INV-126629 | 9,626.09 | 0 | 750 |
| C003 | TrueNorth Renovations | INV-126629 | 9,626.09 | 0 | 750 |
| C003 | TrueNorth Renovations | INV-232408 | 8,764.05 | 0 | 500 |
| C003 | TrueNorth Renovations | INV-532922 | 14,077.01 | 0 | 500 |
| C003 | TrueNorth Renovations | INV-532922 | 14,077.01 | 0 | 500 |
| C003 | TrueNorth Renovations | INV-652272 | 28,199.02 | 0 | 250 |
| C003 | TrueNorth Renovations | INV-116924 | 20,115.64 | 0 | 250 |
| C003 | TrueNorth Renovations | INV-116924 | 20,115.64 | 0 | 250 |
| C004 | Ironclad General Contractors | INV-134862 | 16,049.21 | 0 | 500 |
| C005 | Summit Point Developments | INV-573451 | 16,718.12 | 0 | 750 |
| C005 | Summit Point Developments | INV-666579 | 10,737.50 | 0 | 500 |

### 4.5 Duplicate data
After analyzing all the customer tables, 98 out of 445 payment transactions is duplicated. The duplicate data has the same invoice no., paid amount matching the same customer and customer ID. This duplication could represent weaker internal controls over the account receivable management process or a malfunctioning of the integrity capacity of the account receivable software in maintaining and updating customer information accurately. A couple of the discovered data has been posted below. 
## Payment Count Analysis

| Customer ID | Customer Name | Invoice Count | Paid Amount |
|---|---|---|---|
| C001 | Maple Ridge Construction Ltd. | 2 | 13,564.16 |
| C002 | Northern Peak Builders Inc. | 2 | 18,336.19 |
| C003 | TrueNorth Renovations | 2 | 9,626.09 |
| C003 | TrueNorth Renovations | 2 | 9,365.00 |
| C003 | TrueNorth Renovations | 2 | 14,077.01 |
| C003 | TrueNorth Renovations | 2 | 20,115.64 |
| C004 | Ironclad General Contractors | 2 | 1,686.69 |
| C004 | Ironclad General Contractors | 2 | 23,000.38 |
| C005 | Summit Point Developments | 2 | 10,737.50 |
| C006 | Prairie West Construction Group | 2 | 2,337.31 |
| C006 | Prairie West Construction Group | 2 | 15,891.28 |
| C006 | Prairie West Construction Group | 2 | 13,216.44 |
| C006 | Prairie West Construction Group | 2 | 13,225.06 |
| C007 | BlueRock Commercial Builders | 2 | 14,696.07 |
| C007 | BlueRock Commercial Builders | 2 | 23,354.45 |
| C008 | Polaris Electrical Services | 2 | 6,546.86 |
| C008 | Polaris Electrical Services | 2 | 29,544.68 |
| C008 | Polaris Electrical Services | 2 | 25,691.40 |
| C008 | Polaris Electrical Services | 2 | 24,744.97 |
| C008 | Polaris Electrical Services | 2 | 12,508.04 |

# 5.0 Strategy Recommendations
The following strategies have been recommended to aid resolve the identified account receivable management weaknesses of the store.
| Identified Weaknesses | Recommendation | Management Impact | Risk to be Considered |
|---|---|---|---|
| **Out of the 41 customers, 34 customers exceeded their approved credit limit by 1% to 73%** | - Implement automated credit limit checks that block or escalate credit sales once approved limits are exceeded<br>- Introduce tiered credit approval levels<br>- Conduct periodic credit limit reassessments<br>- Introduce real-time credit utilization monitoring | - Enforcing credit controls, correcting inaccurate balances, and eliminating duplicate payments gives management a more accurate view of receivables, improving cash flow forecasting and liquidity planning<br>- Automated checks and monitoring strengthen credit risk management and compliance<br>- Automation and exception reporting reduce manual intervention and improve reliability of controls<br>- Standardized metrics support proactive, data-driven decision-making | **Customer Relationship and Revenue Risk**<br>- Stricter credit enforcement or revised terms may negatively impact customer relationships, reduce sales volume, or lead to loss of key accounts<br><br>**Operational Disruption and Change Management Risk**<br>- New controls and monitoring processes may temporarily disrupt operations or face resistance from staff<br><br>**Data Dependency and Control Design Risk**<br>- Automated controls rely heavily on accurate data, proper system integration, and effective configuration. Poor data quality may generate false positives or fail to identify genuine risks |
| **10 customers make up 36% of total sales, indicating customer concentration risk** | - Expand sales efforts toward underrepresented customer segments<br>- Classify high-revenue customers as key accounts with enhanced credit monitoring<br>- Perform scenario analysis to assess financial impact of customer concentration<br>- Strengthen contractual payment terms and customer retention strategies | - Reduces dependence on a small number of customers<br>- Improves resilience and stability of future revenue streams<br>- Enhances visibility and management of strategic customer accounts | - Loss of a major customer could significantly impact revenue and cash flow<br>- Additional monitoring and retention programs may increase operational costs |
| **Debt balances show overdue periods from 35 to 399 days; however, overdue amounts are caused by inaccurate debt balances of $250 to $750 on customer accounts. No customer has true overdue balances.** | - Implement automated invoice-to-payment reconciliation<br>- Introduce system thresholds to auto-write-off or flag immaterial balances<br>- Perform root cause analysis of balance discrepancies | - Improves accuracy of receivables reporting and customer account balances<br>- Reduces false overdue reporting and improves collection efficiency<br>- Enhances trust in financial and operational reporting | - Incorrect write-off thresholds may unintentionally remove valid balances<br>- System automation errors may create reconciliation gaps if not monitored properly |
| **98 out of 445 payment transactions are duplicated** | - Implement controls to flag identical customer, invoice, amount, and payment date combinations<br>- Enforce invoice-payment-customer matching before posting payments<br>- Separate payment entry, approval, and reconciliation responsibilities<br>- Produce weekly or monthly duplicate payment exception reports for management review | - Reduces duplicate payment errors and potential fraud exposure<br>- Strengthens financial controls and payment processing accuracy<br>- Improves efficiency of reconciliation and audit processes | - Additional controls may slow transaction processing times initially<br>- Overly restrictive matching rules may delay legitimate transactions<br>- Increased monitoring requirements may require additional staff training and oversight |

