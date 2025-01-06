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


