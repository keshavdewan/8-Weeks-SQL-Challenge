# Case Study 7 - Balanced Tree Clothing Co.👕
Balanced Tree Clothing Company prides themselves on providing an optimised range of clothing and lifestyle wear for the modern adventurer!

Danny, the CEO of this trendy fashion company has asked you to assist the team’s merchandising teams analyse their sales performance and generate a basic financial report to share with the wider business.

<img src = "https://github.com/user-attachments/assets/04261b6f-7241-490b-b04b-26f443815da1" alt = "Image" width = "500" height = "520">

# Table of Content
-  [Entity Relationship Diagram](#entity-relationship-diagram)
-  [Case Study Questions with Solutions](#Case-study-questions-with-solutions)
    -  [A. High Level Sales Analysis](#a-high-level-sales-analysis)
    -  [B. Transaction Analysis](#b-transaction-analysis)
    -  [C. Product Analysis](#c-product-analysis)
    -  [D. Reporting Challenge](#d-reporting-challenge)
    -  [E. Bonus Challenge](#e-bonus-challenge)
  -  [Learnings](#learnings)

***

## Entity Relationship Diagram
Data can be extracted from [DB Fiddle](https://www.db-fiddle.com/f/dkhULDEjGib3K58MvDjYJr/8)

<img src = "https://github.com/user-attachments/assets/e74dd71b-af4b-46f3-a840-c2f6f93c47d2" alt = "Image" width = "500" height= "300">

The database contains the Tables below:
-  `product_details`: includes all information about the entire range that Balanced Clothing sells in their store

|product_id|price|product_name|category_id|segment_id|style_id|category_name|segment_name|style_name|
|:----|:----|:----|:----|:----|:----|:----|:----|:----|
|c4a632|13|Navy Oversized Jeans - Womens|1|3|7|Womens|Jeans|Navy Oversized|
|e83aa3|32|Black Straight Jeans - Womens|1|3|8|Womens|Jeans|Black Straight|
|e31d39|10|Cream Relaxed Jeans - Womens|1|3|9|Womens|Jeans|Cream Relaxed|
|d5e9a6|23|Khaki Suit Jacket - Womens|1|4|10|Womens|Jacket|Khaki Suit|
|72f5d4|19|Indigo Rain Jacket - Womens|1|4|11|Womens|Jacket|Indigo Rain|
|9ec847|54|Grey Fashion Jacket - Womens|1|4|12|Womens|Jacket|Grey Fashion|
|5d267b|40|White Tee Shirt - Mens|2|5|13|Mens|Shirt|White Tee|
|c8d436|10|Teal Button Up Shirt - Mens|2|5|14|Mens|Shirt|Teal Button Up|
|2a2353|57|Blue Polo Shirt - Mens|2|5|15|Mens|Shirt|Blue Polo|
|f084eb|36|Navy Solid Socks - Mens|2|6|16|Mens|Socks|Navy Solid|

-  `product_sales`: contains product level information for all the transactions made for Balanced Tree including quantity, price, percentage discount, member status, a transaction ID and also the transaction timestamp

|prod_id|qty|price|discount|member|txn_id|start_txn_time|
|:----|:----|:----|:----|:----|:----|:----|
|c4a632|4|13|17|true|54f307|2021-02-13T01:59:43.296Z|
|5d267b|4|40|17|true|54f307|2021-02-13T01:59:43.296Z|
|b9a74d|4|17|17|true|54f307|2021-02-13T01:59:43.296Z|
|2feb6b|2|29|17|true|54f307|2021-02-13T01:59:43.296Z|
|c4a632|5|13|21|true|26cc98|2021-01-19T01:39:00.345Z|
|e31d39|2|10|21|true|26cc98|2021-01-19T01:39:00.345Z|
|72f5d4|3|19|21|true|26cc98|2021-01-19T01:39:00.345Z|
|2a2353|3|57|21|true|26cc98|2021-01-19T01:39:00.345Z|
|f084eb|3|36|21|true|26cc98|2021-01-19T01:39:00.345Z|
|c4a632|1|13|21|false|ef648d|2021-01-27T02:18:17.164Z

-  `product_hierarchy`


|id|parent_id|level_text|level_name|
|:----|:----|:----|:----|
|1|null|Womens|Category|
|2|null|Mens|Category|
|3|1|Jeans|Segment|
|4|1|Jacket|Segment|
|5|2|Shirt|Segment|
|6|2|Socks|Segment|
|7|3|Navy Oversized|Style|
|8|3|Black Straight|Style|
|9|3|Cream Relaxed|Style|
|10|4|Khaki Suit|Style|

-  `product_prices`


|id|product_id|price|
|:----|:----|:----|
|7|c4a632|13|
|8|e83aa3|32|
|9|e31d39|10|
|10|d5e9a6|23|
|11|72f5d4|19|
|12|9ec847|54|
|13|5d267b|40|
|14|c8d436|10|
|15|2a2353|57|
|16|f084eb|36|

***

## Case Study Questions with Solutions

### A. High Level Sales Analysis

#### 1. What was the total quantity sold for all products?
_Approach Taken_
-  Used `SUM` of `qty` for calculating `total_quantity`
-  `JOIN` tables `product_details` and `sales`

````sql
SELECT pd.product_name,
	SUM(s.qty) AS total_quantity
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
GROUP BY pd.product_name
````
![image](https://github.com/user-attachments/assets/a1cb1cb9-ef14-4b01-9c87-d10b1b360112)

#### 2. What is the total generated revenue for all products before discounts?
_Approach Taken_
-	calculated product of `SUM` of `qty` and `price` from `sales` table

````sql
SELECT pd.product_name,
	SUM(s.qty * s.price) AS total_revenue
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
GROUP BY pd.product_name
````

![image](https://github.com/user-attachments/assets/9f26802d-b0e8-4fb3-bf7e-4ac42cb08456)

#### 3. What was the total discount amount for all products?
_Approach Taken_
-	multi[plied `discuount` as well to the total revenue

````sql
SELECT pd.product_name,
	SUM(s.qty * s.price * s.discount/100) AS total_discount
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
GROUP BY pd.product_name
````
![image](https://github.com/user-attachments/assets/8425e628-3deb-4d75-b8de-5b52adcce278)

### B. Transaction Analysis

#### 1. How many unique transactions were there?
_Approach Taken_
-	used `COUNT(DISTINCT)`

````sql
SELECT COUNT(DISTINCT(txn_id)) AS unique_transaction
FROM balanced_tree.sales
````
![image](https://github.com/user-attachments/assets/bd69c094-db9e-4e64-a29d-b8552c9d6dce)

#### 2. What is the average unique products purchased in each transaction?
_Approach Taken_
-	CTE `TransactionProductCounts` counts the distinct transacation ids,
-	`COUNT(prod_id) OVER (PARTITION BY txn_id)` counts the number of products per transaction using a window function
-	Final `SELECT` statement calculates the average of product quantity

````sql
WITH TransactionProductCounts AS (
    SELECT DISTINCT txn_id, 
           COUNT(prod_id) OVER (PARTITION BY txn_id) AS product_txn_qty
    FROM balanced_tree.sales
)
SELECT ROUND(AVG(product_txn_qty)) AS avg_products_qty
FROM TransactionProductCounts
````
![image](https://github.com/user-attachments/assets/b09c9023-542f-4bfc-9315-dd3546b7f585)

#### 3. What are the 25th, 50th and 75th percentile values for the revenue per transaction?
_Approach Taken_
-	CTE `transaction_revenue` calculates the `transaction_revenue` for each transaction
-	Final `SELECT` statement uses `PERCENTILE_CONT` to calculate 25th, 50th and 75th percentiles of `transaction_revenue`

````sql
WITH transaction_revenue AS(
		SELECT txn_id,
			SUM(price * qty) AS transaction_revenues
		FROM balanced_tree.sales
		GROUP BY txn_id
)
SELECT 
	PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY transaction_revenues) AS percentile_25th,
	PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY transaction_revenues) AS percentile_50th,
	PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY transaction_revenues) AS percentile_75th
FROM transaction_revenue
````

![image](https://github.com/user-attachments/assets/bddccb66-2363-4218-a7c1-8bda92241c75)

#### 4. What is the average discount value per transaction?
_Approach Taken_
-	CTE `totaldiscount` calculates the `total_discount` by calculating product of `SUM` of `qty`, `price` and `discount`
-	Fianl `SELECT` statement calculates the `AVG` of `total_discount`

````sql
WITH totaldiscount AS(
		SELECT txn_id,
			SUM(qty * price * discount/100) AS total_discount
		FROM balanced_tree.sales
		GROUP BY txn_id
)

SELECT  ROUND(AVG(total_discount),2) AS avg_discount
FROM totaldiscount
````
![image](https://github.com/user-attachments/assets/b8f9c436-fbe7-4a62-80f0-1fdf8a18ce07)

#### 5. What is the percentage split of all transactions for members vs non-members?
_Approach Taken_
-	CTE `transactions_cte` counts the distinct `txn_id`
-	Final `SELECT` statement calculates the percentage of `txn_id` for members and non-members	

````sql
WITH transactions_cte AS (
  SELECT member,
    	COUNT(DISTINCT txn_id) AS transactions
  FROM balanced_tree.sales
  GROUP BY member
)

SELECT member,
  	transactions,	
	  ROUND(100 * transactions
	    /(SELECT SUM(transactions) 
	FROM transactions_cte)) AS percentage
FROM transactions_cte
GROUP BY member, transactions
````
![image](https://github.com/user-attachments/assets/00fdaf9d-d2b9-4d6d-be59-70bc0a926f16)

#### 6. What is the average revenue for member transactions and non-member transactions?
_Approach Taken_
-	CTE `transaction_cte` calculates the `total_revenue`
-	Final `SELECT` statement rounds the `avg` of the `total_revenue` to calculate the `avg_revenue`

````sql
WITH transactions_cte AS (
  SELECT member,
	 txn_id,
	SUM(qty * price) AS total_revenue
FROM balanced_tree.sales
GROUP BY member,
	txn_id
)
SELECT member,
	ROUND(AVG(total_revenue),2) AS avg_revenue
FROM transactions_cte
GROUP BY member
````
![image](https://github.com/user-attachments/assets/186e3742-02b4-4b60-a5d1-cfb1d0c42101)

### C. Product Analysis

#### 1. What are the top 3 products by total revenue before discount?
_Approach Taken_
- 	calculate the `SUM` of `qty` *  `price`


````sql
SELECT pd.product_name,
	SUM(s.qty) * SUM(s.price) AS total_revenue
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
GROUP BY product_name
ORDER BY total_revenue DESC
LIMIT 3
````
![image](https://github.com/user-attachments/assets/4e68ff20-2130-4788-bc4d-c277402b77ea)

#### 2. What is the total quantity, revenue and discount for each segment?	

````sql
SELECT segment_id,
	segment_name,
	SUM(s.qty) AS total_qty,
	SUM(s.price * s.qty) AS total_revenue,
	SUM((s.price * s.qty) * s.discount/100) AS total_discount
FROM balanced_tree.product_details pd
JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
GROUP BY segment_id,
	segment_name
ORDER BY total_revenue DESC
````
![image](https://github.com/user-attachments/assets/f88a4af3-1081-4dc2-ba72-b57512fc4c5e)


#### 3. What is the top selling product for each segment?
_Approach Taken_
-	CTE `top_selling` calculates and `Ranks()` the `segment_id` w.r.t `qty`
-	Final `SELECT` statement `LIMIT` this ranking to 1

````sql
WITH top_selling AS(
		SELECT segment_id,
			segment_name,
			product_id,
			product_name,
			SUM(s.qty) AS total_qty,
			RANK() OVER(PARTITION BY segment_id 
					ORDER BY SUM(s.qty) DESC) AS ranking
		FROM balanced_tree.product_details pd
		JOIN balanced_tree.sales s ON pd.product_id = s.prod_id
		GROUP BY segment_id,
		segment_name,
		product_id,
		product_name
				
)
SELECT segment_id,
	segment_name,
	product_id,
	product_name,
	total_qty
FROM top_selling
WHERE ranking = 1
GROUP BY segment_id,
	segment_name,
	product_id,
	product_name,
	total_qty
````

![image](https://github.com/user-attachments/assets/ea8a6d83-6cd6-46a3-96c2-2ce5d1d05f23)
