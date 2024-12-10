# Case Study 4 - Data Bank

## Introduction
Danny has decided to get into the business of Neo-banks: new aged digital only banks without physical branches.
Data Bank runs just like any other digital bank - but it isn't only for banking activities, they also have the world's most secure distributed data storage platform!
Customers are allocated cloud data storage limits which are directly linked to how much money they have in their accounts. 

![](https://cdn-images-1.medium.com/max/1100/0*4Pp4dwa3KzcT5ME9.png)

## Task
There are a few interesting caveats that go with this business model, and this is where the Data Bank team need your help! 
-  Increasing their customer base: Like any new business, they want to attract more customers.
-  Forecasting data storage needs: They need to accurately predict how much storage their customers will need in the future to plan for their infrastructure needs.

This case study focuses on using data analysis to help Data Bank achieve these goals by calculating key metrics, analyzing growth, and making data-driven decisions.

## Table of content
-  [ERD Diagram](#erd-diagram)
-  [Case Study Questions with Solutions](#case-study-questions-with-solutions)
-  [A. Customer Nodes Exploration](#customer-nodes-exploration)
-  [B. Customer Transactions](#customer-transactions)
-  [C. ](#)


***

### ERD Diagram
The Data Bank team have prepared a data model for this case study as well as a few example rows from the complete dataset below to get you familiar with their tables.

#### Entity Relationship Diagram
<img src= "https://cdn-images-1.medium.com/max/1100/0*jPsWpBOKuPgsG7wd.png">

The Data Bank contains 3 tables:
-  `regions` : The regions table contains the `region_id` and their respective `region_name` values
  <img src= "https://cdn-images-1.medium.com/max/1100/1*U3HJmHTW7JsxKycvVfmG8Q.png" alt="image" width ="150" height="200">
  
-  `customer_nodes` : Customers are randomly distributed across the nodes according to their region - this also specifies exactly which node contains both their cash and data. Below is a sample of the top 10 rows of the `data_bank.customer_nodes`
   <img src= "https://cdn-images-1.medium.com/max/1100/1*4XW8wyxTXaXaN8qYC3aG6A.png" alt="image" width ="400" height="300">
   
-  `customer_transactions` : This table stores all customer deposits, withdrawals and purchases made using their Data Bank debit card.
  <img src= "https://cdn-images-1.medium.com/max/1100/1*uv9QnY_TaYoX-6CZeAK17Q.png" alt="image" width ="400" height="400">

#### Case Study Questions with Solutions

#### A. Customer Nodes Exploration

#### 1. How many unique nodes are there on the Data Bank system?
_Approach Taken_
  - Using `COUNT(DISTINCT node_id)` to calculate the unique nodes in the system
  
````sql
SELECT COUNT(DISTINCT node_id) AS unique_node
FROM customer_nodes
````
![image](https://github.com/user-attachments/assets/1a8afce8-1256-4b8e-a1a5-5dab0426bdd8)

#### 2. What is the number of nodes per region?
_Approach Taken_
  - Since we have been asked the number of nodes per region, we will join table `customer_nodes` with table `regions` ON `region_id` so as to have the `region_name` as well
    
````sql
SELECT 	r.region_id,
		r.region_name,
		COUNT(DISTINCT cn.node_id) AS nodes
FROM regions r
JOIN customer_nodes cn ON r.region_id = cn.region_id
GROUP BY r.region_id,
		r.region_name
````

![image](https://github.com/user-attachments/assets/ed9a5c69-61da-4b74-87a9-ce4d9c618824)

#### 3. How many customers are allocated to each region?
_Approach Taken_
  - This is similar to the previous query, we just had to replace the `node_id` with `customer_id` here

````sql
SELECT 	r.region_id,
		r.region_name,
		COUNT(DISTINCT cn.customer_id) AS cust_num
FROM regions r
JOIN customer_nodes cn ON r.region_id = cn.region_id
GROUP BY r.region_id,
		r.region_name
````

![image](https://github.com/user-attachments/assets/caa722fe-86bc-4bf6-9401-9aaf5fb4f7a0)

#### 4. How many days on average are customers reallocated to a different node?
_Approach Taken_
  - We used here `AVG(end_date - start_date)
  - There is an abnormal date in the data - '9999-12-31', we have to filter it using the `WHERE` commands

````sql
SELECT 
	ROUND(AVG(end_date - start_date),0) AS avg_reallocation_days
FROM customer_nodes
WHERE end_date != '9999-12-31'
````
![image](https://github.com/user-attachments/assets/7bf4548c-5d4f-4fff-8a88-2801eac9e5ee)

*reallocation days* likely refers to the time a customer's data is hosted on a specific server or node within their system.  Calculating this duration helps Data Bank understand:
-  Customer churn: Shorter reallocation days might indicate customers are leaving or moving their data frequently. This could signal issues with service, pricing, or competition.
-  Resource optimization: Knowing how long data typically stays on a node helps Data Bank plan their infrastructure. They can allocate resources more efficiently, predict when they might need more storage in certain regions, and potentially even offer different service tiers based on data mobility.
-  Identifying patterns: Analyzing reallocation days in conjunction with other factors like region, customer demographics, or account balance could reveal trends and inform business decisions. For example, they might find that customers in certain regions tend to keep their data with them longer.
  
#### 5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?
_Approach Taken_
  - Use a CTE to find the difference between `start_date` and `end_date`
  - Use `PERCENTILE_CONT` and `WITHIN GROUP` to find the median, 80th, and 95th percentile

````sql
WITH date_diff AS(
	SELECT cn.customer_id,
			cn.region_id,
			r.region_name,
			(end_date - start_date) AS reallocation_days
	FROM customer_nodes cn
	JOIN regions r ON cn.region_id = r.region_id
	WHERE end_date != '9999-12-31'
)
SELECT DISTINCT region_id,
				region_name,
				PERCENTILE_CONT(0.5) WITHIN GROUP(ORDER BY reallocation_days)
										OVER(PARTITION BY region_name) AS median,
				PERCENTILE_CONT(0.8) WITHIN GROUP(ORDER BY reallocation_days)
										OVER(PARTITION BY region_name) AS percentile_80,
				PERCENTILE_CONT(0.95) WITHIN GROUP(ORDER BY reallocation_days)
										OVER(PARTITION BY region_name) AS percentile_95
FROM date_diff
ORDER BY region_name
````
*copy pasted

![image](https://github.com/user-attachments/assets/054f3cb2-d1bb-4ce3-b617-ad4f7c078307)

### B. Customer Transactions

#### 1. What is the unique count and total amount for each transaction type?
_Approach Taken_
-	`COUNT(customer_id)` and also `SUM(txn_amount)` for this case

````sql
SELECT txn_type,
		COUNT(customer_id) AS transaction_count,
		SUM(txn_amount) AS total_amt
FROM data_bank.customer_transactions
GROUP BY txn_type
````
![image](https://github.com/user-attachments/assets/4b78c284-1f66-4f6f-a16a-4d2526386843)

#### 2. What is the average total historical deposit counts and amounts for all customers?
_Approach Taken_
-	count the customers and round them for their average

````sql
WITH deposit AS(
		SELECT customer_id,
				COUNT(customer_id) AS deposit_count,
				ROUND(AVG(txn_amount),0) AS avg_deposits
		FROM data_bank.customer_transactions
		WHERE txn_type = 'deposit'
		GROUP BY customer_id
)
SELECT ROUND(AVG(deposit_count),0) AS avg_count,
		ROUND(AVG(avg_deposits),0) AS avg_deposit
FROM deposit
````
![image](https://github.com/user-attachments/assets/22a3ed05-9d15-4480-b318-98060c684adf)

#### 3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?
_Approach Taken_
-	CTE `monthly_transaction` creates `CASE` statement with `txn_type` as 'deposit', 'purchase' and 'withdrawl'
-	In the final SELECT statement we filter using `WHERE`  statement to get the customer numbers
  
````sql
WITH bank_customers AS(
		SELECT EXTRACT(MONTH FROM txn_date) AS month,
				customer_id,
				SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit_count,
				SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase_count,
				SUM(CASE WHEN txn_type = 'withdrawl' THEN 1 ELSE 0 END) AS withdrawl_count
		FROM data_bank.customer_transactions
		GROUP BY month,customer_id
)
SELECT month,
		COUNT(customer_id) AS cust_count
FROM bank_customers
WHERE deposit_count >= 1 AND
	(purchase_count >= 1 OR withdrawl_count >= 1)
GROUP BY month
ORDER BY month
````

![image](https://github.com/user-attachments/assets/cec9e977-a786-4261-bf3f-191474b93954)

#### 4. What is the closing balance for each customer at the end of the month? Also show the change in balance each month in the same table output.
_Approach Taken_
-	CTE `monthlychanges` calculates the `monthly_change` in balance for each customer every month
-	Final `SELECT` statement calculates the `SUM` of `closing_balance` partitioned by `customer_id` and ordered by `month`  

````sql
WITH monthlychanges AS (
    SELECT 
        customer_id,
        EXTRACT(MONTH FROM txn_date) AS month,
        SUM(CASE 
            WHEN txn_type = 'deposit' THEN txn_amount
            WHEN txn_type IN ('purchase', 'withdrawal') THEN -txn_amount
            ELSE 0 
            END) AS monthly_change
    FROM 
        data_bank.customer_transactions
    GROUP BY 
        customer_id, EXTRACT(MONTH FROM txn_date)
)
SELECT 
    mc.customer_id,
    mc.month,
    SUM(mc.monthly_change) OVER (PARTITION BY mc.customer_id ORDER BY mc.month) AS closing_balance,
    mc.monthly_change
FROM 
    monthlychanges mc
ORDER BY 
    mc.customer_id, mc.month
````

![image](https://github.com/user-attachments/assets/b2fe16de-20bb-4f65-8226-2f64b3d5eddb)

#### 5. What is the percentage of customers who increase their closing balance by more than 5%?
_Approach taken_
-	CTE `MonthlyChanges` calculates the monthly changes in balance
-	CTE `ClosingBalances` calculates the `closing_balance` for each month
-	CTE `LaggedBalances` calcultes the closing balance from previous month for each customer
-	Final statement counts how many customers had a significant increase in their balance and divide that by the total number of customers to get a percentage

````sql
WITH MonthlyChanges AS (
    SELECT 
        customer_id,
        EXTRACT(MONTH FROM txn_date) AS month,
        SUM(CASE 
            WHEN txn_type = 'deposit' THEN txn_amount
            WHEN txn_type IN ('purchase', 'withdrawal') THEN -txn_amount
            ELSE 0 
            END) AS monthly_change
    FROM 
        data_bank.customer_transactions
    GROUP BY 
        customer_id, EXTRACT(MONTH FROM txn_date)
), 
ClosingBalances AS (
  SELECT 
        mc.customer_id,
        mc.month,
        SUM(mc.monthly_change) OVER (PARTITION BY mc.customer_id ORDER BY mc.month) AS closing_balance
  FROM MonthlyChanges mc  
),
LaggedBalances AS ( -- Separate CTE to apply LAG()
  SELECT 
    cb.customer_id,
    cb.month,
    cb.closing_balance,
    LAG(cb.closing_balance) OVER (PARTITION BY cb.customer_id ORDER BY cb.month) AS previous_closing_balance
FROM ClosingBalances cb
)
SELECT 
    CAST(COUNT(CASE WHEN closing_balance > previous_closing_balance * 1.05 THEN customer_id END) AS DECIMAL) * 100 / COUNT(customer_id) AS percentage
FROM 
    LaggedBalances
WHERE 
    previous_closing_balance IS NOT NULL
````
*copy pasted

![image](https://github.com/user-attachments/assets/ea72b8de-46ed-4c40-84d3-048be1ffbe5f)


***
### Learnings

#### 1. Percentile
-  `PERCENTILE_CONT()` is a function that calculates percentiles.
-  `WITHIN GROUP (ORDER BY reallocation_days)` specifies the data to use and how to order it.
-  `OVER (PARTITION BY region_name)` calculates percentiles separately for each region.\
-  `LAG` allows you to access data from a previous row in the same result set. It's like having a peek at the row "above" the current one.
