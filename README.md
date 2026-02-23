# Retail-Store-Account-Receivable-Analysis
Optimizing working capital, adequate timeline and debt capacity for the store through the level of credit sales, Debt turnover ratio, credit period and credit limit compliance, supplier and category concentration risk, customer overdue payment, accurateness of the outstanding debt balances and possible fraud through duplicate transactions

-- 
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

