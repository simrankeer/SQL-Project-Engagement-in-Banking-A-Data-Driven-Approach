# SQLProject 
Engagement in Banking A Data Driven Approach
<br> 

Project Questions and solution to find insights from the data:

CREATE database Banking_data_analysis;
USE Banking_data_analysis;
select * from accounts;
select * from branches;
select * from customers;
select * from employees;
select * from transactions;

-- 1) Write a query to list all customers who haven't
-- made any transactions in the last year. How can we make them active again?
-- Provide appropriate reson.
SELECT c.customer_id, c.first_name, c.last_name, c.email
FROM customers c
INNER JOIN accounts a ON c.customer_id = a.customer_id
INNER JOIN transactions t ON a.account_number = t.account_number
WHERE t.transaction_date IS NULL 
OR t.transaction_date < DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY c.customer_id, c.first_name, c.last_name,c.email
order by c.customer_id;

-- 1) PT2 HOW CAN WE GET THEM ACTIVE AGAIN?
-- Identified customers who haven't made transactions in the last year.
-- Strategy to reactivate: targeted promotions, personalized offers, and engagement campaigns.

-- 2) Summarize the total transaction amount per account per month.
select account_number, month(transaction_date) as monthly_transaction, 
round(sum(amount),2) as total_transaction from transactions group by account_number, 
monthly_transaction order by account_number, monthly_transaction;

-- 3) Rank branches based on the total amount of deposits made in the last quarter.
Select b.branch_id, 
b.branch_name, 
round(Sum(t.amount),3) AS total_deposits 
from branches b
join accounts as a on a.branch_id = b.branch_id
join transactions t on a.account_number = t.account_number
where t.transaction_type = 'deposit' 
and t.transaction_date >= Date_sub(Curdate(), interval 3 Month) 
group by b.branch_id, b.branch_name
order by total_deposits DESC;
        
-- 4) Find the name of the customer who has deposited the highest amount.
select c.first_name , t.amount 
from customers as c
left join 
accounts as a
using (customer_id)
join transactions as t
using (account_number)
where amount=(select max(t.amount) from transactions where t.transaction_type = "deposit")
 order by amount 
desc limit 1;

--  5) Identify any accounts that have made more than two transactions in a single day,
--  which could indicate
--  fraudulent activity. How can you verify any fraudulent transaction?
select t.account_number,
date(t.transaction_date),
count(t.transaction_id) as transaction_count
from transactions t
Group by t.account_number, date(t.transaction_date)
Having count(t.transaction_id) > 2;

-- 6) Calculate the average number of transactions per customer per account per month over the last year.
select a.customer_id, a.account_number, YEAR(t.transaction_date) AS transaction_year, 
MONTH(t.transaction_date) AS transaction_month,
    COUNT(t.transaction_id) AS transaction_count,
    COUNT(t.transaction_id) / 12 AS avg_transactions_per_month_last_yearS
FROM transactions t
JOIN accounts a ON t.account_number = a.account_number
WHERE t.transaction_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY a.customer_id, a.account_number, transaction_year, transaction_month
ORDER BY a.customer_id, a.account_number, transaction_year, transaction_month;

-- 6) PT2 Calculate the average number of transactions per customer per account per month over the last year.
-- or can directly take last year as 2023:
select a.customer_id, a.account_number, YEAR(t.transaction_date) AS transaction_year, 
MONTH(t.transaction_date) AS transaction_month,
    COUNT(t.transaction_id) AS transaction_count,
    COUNT(t.transaction_id) / 12 AS avg_transactions_per_month_last_year
FROM transactions t
JOIN accounts a ON t.account_number = a.account_number
WHERE t.transaction_date = 2023
GROUP BY a.customer_id, a.account_number, transaction_year, transaction_month
ORDER BY a.customer_id, a.account_number, transaction_year, transaction_month;

-- 7) Write a query to find the daily transaction volume (total amount of all transactions) 
-- for the past month.
select date(transaction_date) AS transaction_day, SUM(amount) AS daily_transaction_volume
from transactions where transaction_date >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
group by date(transaction_date) order by date(transaction_date) asc;

-- 8).Calculate the total transaction amount performed by each age group in the past year. 
-- (Age groups: 0-17, 18-30, 31-60, 60+)
select case
when TIMESTAMPDIFF(YEAR, c.date_of_birth, CURDATE()) BETWEEN 0 AND 17 THEN '0-17'
when TIMESTAMPDIFF(YEAR, c.date_of_birth, CURDATE()) BETWEEN 18 AND 30 THEN '18-30'
WHEN TIMESTAMPDIFF(YEAR, c.date_of_birth, CURDATE()) BETWEEN 31 AND 60 THEN '31-60'
ELSE '60+' END AS age_group,
SUM(t.amount) AS total_transaction_amount
FROM transactions t
JOIN accounts a ON t.account_number = a.account_number
JOIN customers c ON a.customer_id = c.customer_id
WHERE t.transaction_date >= DATE_SUB(CURDATE(), INTERVAL 1 YEAR)
GROUP BY age_group ORDER BY FIELD(age_group, '0-17', '18-30', '31-60', '60+');

-- 9) Find the branch with the highest average account balance.
select b.branch_id, b.branch_name, avg(a.balance) as avg_account_balance
from accounts a
join branches b 
using(branch_id)
group by b.branch_id, b.branch_name
order by avg_account_balance desc
limit 1;

-- 10) Calculate the average balance per customer at the end of each month in the last year.
select c.customer_id,
year(a.created_at) as year,
month(a.created_at) as month,
avg(a.balance) as avg_balance_per_customer
from accounts a
join customers c ON a.customer_id = c.customer_id
where a.created_at >= date_sub(curdate(), interval 1 year)
group by c.customer_id, year, month
order by c.customer_id, year, month;

-- 10) pt2 Calculate the average balance per customer at the end of each month in the last year.
-- note: considering last year to be time from today till one year back:
select c.customer_id,
year(a.created_at) as year,
month(a.created_at) as month,
avg(a.balance) as avg_balance_per_customer
From accounts a
Join customers c on a.customer_id = c.customer_id
Where a.created_at >= DATE_SUB(CURDATE(), interval 1 year)
GROUP BY c.customer_id, year, month
ORDER BY c.customer_id, year, month;

-- 10) pt3 Calculate the average balance per customer at the end of each month in the last year.
-- note: 10 or if considering last year as 2023 than:
SELECT c.customer_id,
year(a.created_at) as year,
month(a.created_at) as month,
avg(a.balance) as avg_balance_per_customer
From accounts a
Join customers c on a.customer_id = c.customer_id
Where a.created_at = 2023
Group by c.customer_id, year, month
Order by c.customer_id, year, month;

