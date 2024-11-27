# Case Study 3 - Foodie Fi
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content.
Danny launches his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

<img src="https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/3.%20Foodie-fi.png" alt="Image" width ="500" height="520">

# ERD Diagram
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/case-study-3-erd.png">

There are two tables in the database - 
1. `plans` - There are 5 customer plans.
    - Trial — Customer sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
    - Basic Monthly — Customers have limited access and can only stream their videos and is only available monthly at $9.90.
    - Pro Monthly — Customers have no watch time limits and are able to download videos for offline viewing. This plan starts at $19.90 a month
    - Pro Annualy - Same benefits of a Pro Plan, with an annual subscription of $199
    - Churn - When customers cancel their Foodie-Fi service — they will have a Churn plan record with a null price, but their plan will continue until the end of the billing period.
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/plans.webp" alt="Image" width ="300" height="320">

2. `subscriptions` -
      - Customer subscriptions show the exact date where their specific `plan_id` starts.
      - If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the `start_date` in the subscriptions table will reflect the date that the actual plan changes.
      - When customers upgrade their account from a basic plan to a pro monthly or annual pro plan - the higher plan will take effect straightaway.
      - When customers churn - they will keep their access until the end of their current billing period but the `start_date` will be technically the day they decided to cancel their service.
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/subscriptions.webp" alt="Image" width ="300" height="320">

## Questions & Solutions

### A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

_Approach taken_
-	Randomly selected 8 customers from the subscription table, they could be seen in the table image itself

````sql
SELECT 	s.customer_id,
		s.plan_id,
		p.plan_name,
		p.price,
		s.start_date
FROM subscriptions s
JOIN plans p ON p.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
````
#### Solution:
| customer_id | plan_id | plan_name     | start_date  |
|-------------|---------|---------------|-------------|
| 1           | 0       | trial         | 2020-08-01  |
| 1           | 1       | basic monthly | 2020-08-08  |
| 2           | 0       | trial         | 2020-09-20  |
| 2           | 3       | pro annual    | 2020-09-27  |
| 11          | 0       | trial         | 2020-11-19  |
| 11          | 4       | churn         | 2020-11-26  |
| 13          | 0       | trial         | 2020-12-15  |
| 13          | 1       | basic monthly | 2020-12-22  |
| 13          | 2       | pro monthly   | 2021-03-29  |
| 15          | 0       | trial         | 2020-03-17  |
| 15          | 2       | pro monthly   | 2020-03-24  |
| 15          | 4       | churn         | 2020-04-29  |
| 16          | 0       | trial         | 2020-05-31  |
| 16          | 1       | basic monthly | 2020-06-07  |
| 16          | 3       | pro annual    | 2020-10-21  |
| 18          | 0       | trial         | 2020-07-06  |
| 18          | 2       | pro monthly   | 2020-07-13  |
| 19          | 0       | trial         | 2020-06-22  |
| 19          | 2       | pro monthly   | 2020-06-29  |
| 19          | 3       | pro annual    | 2020-08-29  |

- Customer behaviour observations:
    - Customer_id: 1 - started with a trial and converted the trial to basic monthly subscription
    - Customer_id: 2 - started with a trial and converted it to pro annual, customer 2 definitely liked the content of the app
    - Customer_id: 11 - didn't go ahead after the trail phase
    - Customer_id: 13 - choose basic monthly plan after the end of their trial and later after 3 months moved to a pro monthly plan
    - Customer_id: 15 - started with a trial, moved to pro monthly after the trial ended (which might have been done automatically) and later churned out after a month
    - Customer_id: 16 - moved to basic monthly after the trial and later moved to pro annual after 4 months of end of basic monthly 
    - Customer_id: 18 - started with trial and converted it to pro monthly
    - Customer_id: 19 - started with trial and converted it to pro monthly and later changed to pro annual after two monhts 

***
### B. Data Analysis Questions
## 1. How many customers has Foodie-Fi ever had?
_Approach taken_
-	Use `Count()` followed by `Distinct()` functions for calculating the number of customers
````sql
SELECT 	COUNT(DISTINCT(customer_id)) AS customer_count
FROM subscriptions
````

#### Solution:
| customer_count | 
|-------------|
|1000           |

## 2. What is the monthly distribution of trial plan `start_date` values for our dataset - use the start of the month as the group by value
_Approach taken_
-	Counted the plan_names
-	Extracted month from the `start_date` column - Function Used - `EXTRACT(MONTH FROM -table_name-)`
-	Used WHERE clause to filter the `plan_name` as 'trial'
-	 Grouped results by month and ordered them in ascending order

````sql
SELECT 	COUNT(p.plan_name) AS trial_count,
		EXTRACT(MONTH FROM s.start_date) AS month
FROM plans p
JOIN subscriptions s ON p.plan_id = s.plan_id
WHERE plan_name = 'trial'
GROUP BY month
ORDER BY month
````
#### Solution:
| trial_count | month | 
|-------------|---------|
| 88           | 1       | 
| 68           | 2       | 
| 94           | 3       | 
| 81           | 4       | 
| 88          | 5       |
| 79          | 6       | 
| 89          | 7       | 
| 88          | 8       |
| 87          | 9       | 
| 79          | 10      |
| 75          | 11      | 
| 84          | 12      | 

## 3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`
_Approach taken_
-	Selected and Counted the plan_names
-	Extracted year from the `start_date` column - Function Used - `EXTRACT(YEAR FROM s.start_date)` > 2020
-	GROUP BY `plan_id`, `plan_name`
-	ORDER BY `plan_id`

````sql
SELECT 	p.plan_id,
		p.plan_name,
		COUNT(p.plan_name) AS plan_count
FROM plans p
JOIN subscriptions s ON p.plan_id = s.plan_id
WHERE EXTRACT(YEAR FROM s.start_date) > 2020
GROUP BY p.plan_id, plan_name
ORDER BY p.plan_id
````
#### Solution:
| plan_id | plan_name | plan_count    |
|-------------|---------|---------------|
| 1           | basic monthly       | 8         |
| 2           | pro monthly       | 60 |
| 3           | pro annual      | 63         | 
| 4           | churn      | 71 |

## 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
_Approach taken_
-	Although subquery would have been a smaller approach, but I would like to apply CTEs here so as to give a better understanding of the case
-	`total_customer` CTE - counts the total distinct customers from the `subscriptions` table
-	`churned_customers` CTE - counts distinct customers with plan_id = 4
-	ROUND((100* `churned` / `total`),1) - computes the percentage and rounds it to 1 place
-	`CROSS JOIN` Combines the total customer count and churned customer count without introducing extra rows.

````sql
WITH total_customers AS (
    SELECT COUNT(DISTINCT customer_id) AS total_customer_count
    FROM subscriptions
),
churned_customers AS (
    SELECT COUNT(DISTINCT customer_id) AS churned_customer_count
    FROM subscriptions
    WHERE plan_id = 4
)
SELECT 
    cc.churned_customer_count AS customer_churn,
    ROUND(100.0 * cc.churned_customer_count / tc.total_customer_count, 1) AS churn_percentage
FROM churned_customers cc
CROSS JOIN total_customers tc
````
#### Solution:
| customer_churn | churn_percentage | 
|-------------|---------|
| 307           | 30.7       | 

### 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
_Approach taken_
-	Query divided into 3 parts
	-	CTE `trial_start_date` finds `start_date` i.e. trial date for each customers
 	-	CTE `churn_after_trial` finds customers churned after 1 week of trial, this been done by adding 7 days `INTERVAL` to `min_start_date`
  	-	The final `SELECT` statement counts the customers and percentage of churned customers

````sql
WITH trial_start_date AS (
	SELECT customer_id,
			MIN(start_date) AS min_start_date
	FROM foodie_fi.subscriptions
	WHERE plan_id =0
	GROUP BY customer_id
),
churn_after_trial AS ( 
	SELECT tsd.customer_id
	FROM trial_start_date tsd
	JOIN foodie_fi.subscriptions s ON tsd.customer_id = s.customer_id
	WHERE plan_id = 4
	AND s.start_date BETWEEN tsd.min_start_date AND tsd.min_start_date + INTERVAL '7 days'
)
SELECT COUNT(DISTINCT cat.customer_id) AS cust_numbers,
		ROUND(100.0* COUNT(DISTINCT cat.customer_id)/
				(SELECT COUNT(DISTINCT customer_id)
		FROM foodie_fi.subscriptions
		)) AS churn_percent
FROM churn_after_trial cat
````
#### Solution:
| cust_nmbers | churn_percent | 
|-------------|---------|
| 92           | 9       | 

***
### Learnings
##### 1.`CROSS JOIN`
-	combines every row of the first table with every row of the second table (kind of SUMX)
