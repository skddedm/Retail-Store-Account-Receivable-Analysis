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

