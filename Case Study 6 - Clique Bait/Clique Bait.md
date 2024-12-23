# Case Study 6 - Clique Bait🎣
Clique Bait is an online seafood store, founded by Danny who also happens to be its CEO.

# Task
In this case study - we will be required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

<img src = "https://github.com/user-attachments/assets/33df4d09-be8b-48f2-9e46-8bdbbbd38625" alt = "Image" width = "500" height = "520">

# Table of Contents 
The case study has been divided into the following parts:
-  [Entity Relation Diagram](#entity-relation-diagram)
-  [Case Study Questions with Solutions](#case-study-questions-with-solutions)
    -   [A. Digital Analysis](#a-digital-analysis)
    -   [B. Product Funnel Analysis](#b-product-funnel-analysis)
    -   [C. Campaign Analysis](#c-campaign-analysis)
-   [Learnings](#learnings)

***

# Entity Relation Diagram
Data can be taken from [dbfiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)

![image](https://github.com/user-attachments/assets/0f52fdd6-9389-4878-8342-ddd8057b2cf7)

The database consists of 5 Tables:
1. `users` - Customers who visit the Clique Bait website are tagged via their `cookie_id`
<img width="366" alt="134623074-7c51d63a-c0a4-41e0-a6fc-257e4ca3997d" src="https://github.com/user-attachments/assets/2e654e7a-d962-48f9-aaab-a50322de28df" />

2. `events` - Customer visits are logged in this events table at a `cookie_id` level and the `event_type` and `page_id` values can be used to join onto relevant satellite tables to obtain further information about each event. The `sequence_number` is used to order the events within each visit.
<img width="849" alt="134623132-dfa2acd3-60c9-4305-9bea-6b39a9403c14" src="https://github.com/user-attachments/assets/eb4ef4c8-f553-4da0-a8c2-ef6df66f9dc2" />

3. `event identifier` - The `event_identifier` table shows the types of events which are captured by Clique Bait’s digital data systems.
<img width="273" alt="134623311-1ad16fe7-36e3-45b6-9dc6-8114333cf473" src="https://github.com/user-attachments/assets/6aa77aae-3da2-48eb-87d7-eb003b1c7e4f" />

4. `campaign identifier` - This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.
<img width="792" alt="134623354-0977d67c-fc61-4e61-90ee-f24a29682a9b" src="https://github.com/user-attachments/assets/202c1f21-80c3-466a-b5fb-8ee8cd25663b" />

5. `page hierarchy` - This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.
<img width="576" alt="134623202-3158ca06-6f04-4b67-91f1-e184761e885c" src="https://github.com/user-attachments/assets/bec578da-e854-4d74-877f-8f1402b15675" />

***
# Case Study Questions with Solutions
## A. Digital Analysis

### 1. How many users are there?
_Approach Taken_
-    used `COUNT(DISTINCT)` function for calculating the numbers from `user_id`

````sql
SELECT COUNT(DISTINCT user_id)
FROM clique_bait.users
````
![image](https://github.com/user-attachments/assets/442829ec-aaa4-40e1-8584-d0b37893008b)

### 2. How many cookies does each user have on average?
_Approach Taken_
-  CTE `cookie_count` counts the number of cookies for each user
-  Final `SELECT` statement calculates the average of this count

````sql
WITH cookie_count AS(
		SELECT user_id,
				COUNT(cookie_id) AS cookie_id_count
		FROM clique_bait.users
		GROUP BY user_id
		)
SELECT ROUND(AVG(cookie_id_count),0) AS avg_user_cookies
FROM cookie_count
````
![image](https://github.com/user-attachments/assets/4b47ee0e-afc9-4331-946f-247b7e1172d1)

### 3. What is the unique number of visits by all users per month?
_Approach Taken_
-    Used `EXTRACT` to get the months and `DISTINCT` to count the visits

````sql
SELECT 	EXTRACT(MONTH FROM event_time) AS month,
		COUNT(DISTINCT visit_id) AS unique_visits 
FROM clique_bait.events
GROUP BY month
````

![image](https://github.com/user-attachments/assets/e80d727f-71ed-4132-ba46-6d26c0da9402)

### 4. What is the number of events for each event type?
_Approach Taken_
-    used `COUNT` of `event_type` for the number of events here

````sql
SELECT 	event_type,
	COUNT(event_type) AS event_count 
FROM clique_bait.events
GROUP BY event_type
````
![image](https://github.com/user-attachments/assets/30720b8d-e1f1-4dd0-a7a5-325aa7b7b34f)

### 5. What is the percentage of visits which have a purchase event?
_Approach Taken_
-    CTE 'purchasevisits` calcualtes the distinct `visit_id` from `events` table having event as `purchase`
-    CTE `totalvisits` calculates the total count of distinct visits
-    Final `SELECT` statement counts the purchase visits giving the number of visits with purchase and divides with total visits to get the percentage

````sql
WITH purchasevisits AS (
    SELECT DISTINCT e.visit_id
    FROM clique_bait.events AS e
    JOIN clique_bait.event_identifier AS ei ON e.event_type = ei.event_type
    WHERE ei.event_name = 'Purchase'
),
totalvisits AS (
    SELECT COUNT(DISTINCT visit_id) as total_visits 
    FROM clique_bait.events
)
SELECT 
    ROUND((100.0 * COUNT(pv.visit_id) / tv.total_visits),2) AS percentage_purchase
FROM purchasevisits pv, totalvisits tv
GROUP BY tv.total_visits
````
![image](https://github.com/user-attachments/assets/13d03825-c5c9-4b04-b7f6-43700e34035d)

### 6. What is the percentage of visits which view the checkout page but do not have a purchase event?
_Approach Taken_
-	CTE `checkout_purchase` covers two `CASE` statements, when `event_type` is `1` (page view) and `page_id = `12` (checkout) as checkout and 
									`event_type` is `3` (purchase) as purchase then it would give the `MAX` score
-	Final `SELECT` statement calculates the percentage of `purchase` to `checkout`

````sql
WITH checkout_purchase AS (
SELECT 
  visit_id,
  MAX(CASE WHEN event_type = 1 AND page_id = 12 THEN 1 ELSE 0 END) AS checkout,
  MAX(CASE WHEN event_type = 3 THEN 1 ELSE 0 END) AS purchase
FROM clique_bait.events
GROUP BY visit_id)

SELECT 
  ROUND(100 * (1-(SUM(purchase)::numeric/SUM(checkout))),2) AS percentage_checkout_view_with_no_purchase
FROM checkout_purchase
````
*copy-pasted

![image](https://github.com/user-attachments/assets/d492c8aa-dff8-4626-9a47-e70f65da1630)


### 7. What are the top 3 pages by number of views?
_Approach Taken_
-	CTE `top_page` counts the top pages visited
-	Final `SELECT` statement adds the `page_name` to them

````sql
WITH top_page AS(
		SELECT 	page_id,
		COUNT(event_type) AS page_views
		FROM clique_bait.events
		WHERE event_type = 1
		GROUP BY page_id
		ORDER BY page_views
		)
SELECT tp.page_id,
		ph.page_name,
		tp.page_views
FROM top_page tp
JOIN clique_bait.page_hierarchy ph ON tp.page_id = ph.page_id
GROUP BY tp.page_id,
		ph.page_name,
		tp.page_views
ORDER BY tp.page_views DESC
LIMIT 3
````

![image](https://github.com/user-attachments/assets/ea2bdc5a-3803-48a2-bb39-5ea2683e2a20)

### 8. What is the number of views and cart adds for each product category?
_Approach Taken_
-	Use `CASE` Statement to sum the number of instances when `event_type` is `page view` or `add to cart`

````sql
SELECT ph.product_category,
	SUM(CASE WHEN e.event_type = 1 THEN 1 ELSE 0 END) AS page_view,
	SUM(CASE WHEN e.event_type = 2 THEN 1 ELSE 0 END) AS cart_add
FROM clique_bait.events e
JOIN clique_bait.page_hierarchy ph ON e.page_id = ph.page_id
WHERE ph.product_category IS NOT NULL
GROUP BY ph.product_category
ORDER BY page_view DESC
````
![image](https://github.com/user-attachments/assets/45932499-7c83-4462-be9d-09c359573fa6)
