# Case Study 3 - Foodie Fi
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content.
Danny launches his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

<img src="https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/3.%20Foodie-fi.png" alt="Image" width ="500" height="520">

# Task
This case study focuses on using subscription style digital data to answer important business questions.


# Table of content
The case study has been divided into the following parts:
-	[ERD Diagram](#erd-diagram)
-	[Case Study Questions with Solutions](#case-study-questions-with-solutions)
	-	[A. Customer Journey](#a-customer-journey)
 	-	[B. Data Analysis Questions](#b-data-analysis-questions)
  	-	[C. Challenge Payment Question](#c-challenge-payment-question)
  	-	[Learnings](#learnings)

***

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
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/subscriptions.webp" alt="Image" width ="300" height="520">

***

## Case Study Questions with Solutions

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

### 6. What is the number and percentage of customer plans after their initial free trial?
_Approach taken_
-	Used LEAD() for this query, as it was termed to be 'simpler'
-	`LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY plan_id)`: This expression retrieves the `plan_id` from the next row within the same `customer_id` partition, ordered by `plan_id`. This effectively identifies the plan a customer switches to after their current plan.
-	`WHERE next_plan_id IS NOT NULL AND plan_id = 0`: Filters the data to include only conversions from the free trial (`plan_id = 0`) and excludes cases where there is no next plan (`next_plan_id IS NOT NULL`)

*took AI's and other's github profile help for this

````sql
WITH next_plans AS (
	SELECT customer_id,
			plan_id,
			LEAD(plan_id) OVER (
				PARTITION BY customer_id
				ORDER BY plan_id
			) AS next_plan_id
	FROM foodie_fi.subscriptions
	)
SELECT next_plan_id AS plan_id,
		COUNT(customer_id) AS converted_customers,
		ROUND(100.0* 
				COUNT(customer_id)::NUMERIC/
				(SELECT COUNT(DISTINCT customer_id)
				FROM foodie_fi.subscriptions)
				,1) AS conversion_percentage
FROM next_plans
WHERE next_plan_id IS NOT NULL
AND plan_id = 0
GROUP BY next_plan_id
ORDER BY next_plan_id
````
#### Solution:
| plan_id | converted_customers | conversion_percentage    |
|-------------|---------|---------------|
| 1           | 546      | 54.6         |
| 2           | 325     | 32.5 |
| 3           | 37     | 3.7         | 
| 4           | 92      | 9.2 |

### 7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?
_Approach taken_
-	CTE `customer_latest_date` selects customer with `MAX(start_date)` and later filters them with `WHERE` clause to be before '2020-12-31'
-	CTE `customer_latest_plan` selects customers and their `plan_id` and joins the CTE with `subscriptions` table in order to determine the `plan_id`
-	The final `SELECT` statement then aggregates the result by `plan_id` to calculate the customer count and percentage for each plan.	

````sql
WITH customer_latest_date AS(
		SELECT customer_id,
				MAX(start_date) AS latest_date
		FROM foodie_fi.subscriptions
		WHERE start_date <= '2020-12-31'
		GROUP BY customer_id
		),
customer_latest_plan AS(
		SELECT cld.customer_id,
				s.plan_id
		FROM customer_latest_date cld
		JOIN foodie_fi.subscriptions s ON cld.customer_id = s.customer_id 
		AND cld.latest_date = s.start_date
)
SELECT plan_id,
		COUNT(DISTINCT customer_id) AS customers,
		ROUND(100.0 * 
				COUNT(DISTINCT customer_id)
					/ (SELECT COUNT (DISTINCT customer_id)
						FROM foodie_fi.subscriptions)
				,1) AS percentage
FROM customer_latest_plan
GROUP BY plan_id
````
#### Solution:
| plan_id | customers | percentage    |
|-------------|---------|---------------|
| 0           | 19      | 1.9         |
| 1           | 224      | 22.4         |
| 2           | 326     | 32.6 |
| 3           | 195     | 19.5        | 
| 4           | 236      | 23.6 |


### 8. How many customers have upgraded to an annual plan in 2020?
_Approach taken_
-	Selected distinct number of customers with an annual plan and `start_date` <= '2020-12-31'
	This was an easy one for a change!

````sql
  SELECT plan_id,
  		COUNT(DISTINCT customer_id) AS customers		
  FROM foodie_fi.subscriptions
  WHERE plan_id = 3 
  AND start_date <= '2020-12-31'
  GROUP BY plan_id
````
#### Solution:
| plan_id | customers |
|-------------|---------|
| 3           | 195     |

### 9. How many days on average does it take for a customer to upgrade to an annual plan from the day they join Foodie-Fi?
_Approach taken_
-	Subquery divided into 2 CTEs
	- CTE `trial_plan`, shows the customers enrolled in the trial plan with the `MIN(start_date`
 	- CTE `annual_plan`, shows the customers enrolled in the annual plan with the `MAX(start_date)`
  	- SELECT statement that averages out the difference between the maximum and minimum `start_date`

````sql
WITH trial_plan AS(
	SELECT customer_id,
			MIN(start_date) AS trial_start_date
	FROM foodie_fi.subscriptions s
	WHERE plan_id = 0 
	GROUP BY customer_id
),
annual_plan AS(
	SELECT customer_id,
			MIN(start_date) AS annual_start_date
	FROM foodie_fi.subscriptions s
	WHERE plan_id = 3
	GROUP BY customer_id
)
SELECT ROUND(AVG(ap.annual_start_date - tp.trial_start_date),0) AS avg_days_to_upgrade
FROM annual_plan ap
JOIN trial_plan tp ON ap.customer_id = tp.customer_id
````
#### Solution:
| avg_days_to_upgrade | 
|-------------|
| 105           | 

### 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
_Approach taken_
-	People have used *width bucket* formula
-	I am using CASE statement having 30day time gap and calculating using another CTE

````sql
WITH trial_plan AS (
    SELECT 
        customer_id, 
        MIN(start_date) AS trial_start_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 0
    GROUP BY customer_id
),
annual_plan AS (
    SELECT 
        customer_id, 
        MIN(start_date) AS annual_start_date
    FROM foodie_fi.subscriptions
    WHERE plan_id = 3
    GROUP BY customer_id
),
upgrade_periods AS (
    SELECT 
        trial_plan.customer_id,
		trial_start_date,
		annual_start_date,
		annual_start_date-trial_start_date AS upgrade_days,
		CASE
			WHEN annual_start_date-trial_start_date <= 30 THEN '0-30 days'
			WHEN annual_start_date-trial_start_date >30 AND annual_start_date-trial_start_date <= 60 THEN '31-60 days'
			WHEN annual_start_date-trial_start_date >60 AND annual_start_date-trial_start_date <= 90 THEN '61-90 days'
			ELSE '90+ days'
		END AS upgrade_period
	FROM trial_plan
	JOIN annual_plan ON trial_plan.customer_id = annual_plan.customer_id
)
SELECT upgrade_period,
		COUNT(*) AS num_customers,
		AVG(upgrade_days) AS avg_days_to_upgrade
FROM upgrade_periods
GROUP BY upgrade_period
ORDER BY upgrade_period
````
#### Solution:
| upgrade_period | num_customers | avg_days_to_upgrade    |
|-------------|---------|---------------|
| 0-30 days           | 49     | 9.9591836734693878        |
| 31-60 days           | 24     | 42.3333333333333333         |
| 61-90 days           | 34     | 71.4411764705882353 |
| 90+days           | 151     | 152.7086092715231788       | 

### 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
_Approach taken_
-	Using 2 CTEs for this query
	- CTE `pro_monthly_customers` shows the customers with a pro monthly plan (`plan_id = 2`)
 	- CTE `basic_monthly_customers` shows the customers with a basic monthly plan ('plan_id = 1')
  	- Final `SELECT` statement selects the customers with a pro monthly plan and joins the two CTEs.
   	- By including `WHERE pmc.start_date < bmc.start_date`, we specifically identify only those customers who downgraded from pro monthly to basic monthly plan	

````sql
WITH pro_monthly_customers AS(
	SELECT customer_id,
			start_date
	FROM foodie_fi.subscriptions
	WHERE plan_id = 2
	AND start_date BETWEEN '2020-01-01' AND '2020-12-31'
),
basic_monthly_customers AS(
	SELECT customer_id,
			start_date
	FROM foodie_fi.subscriptions
	WHERE plan_id = 1
	AND start_date BETWEEN '2020-01-01' AND '2020-12-31'
)
SELECT COUNT(DISTINCT pmc.customer_id) AS downgraded_customers
FROM pro_monthly_customers pmc
JOIN basic_monthly_customers bmc ON pmc.customer_id = bmc.customer_id
WHERE pmc.start_date < bmc.start_date
````
#### Solution:
| upgrade_period | 
|-------------|
| 0 |

### C. Challenge Payment Question
The Foodie-Fi team wants you to create a new payments table for the year 2020 that includes amounts paid by each customer in the subscriptions table with the following requirements:

- monthly payments always occur on the same day of month as the original start_date of any monthly paid plan
- upgrades from basic to monthly or pro plans are reduced by the current paid amount in that month and start immediately
- upgrades from pro monthly to pro annual are paid at the end of the current billing period and also starts at the end of the month period
- once a customer churns they will no longer make payments

This section is quite tricky and I couldn't understand it as it included `RECURSIVE CTE` as well, but I am adding a wonderful medium article written by [Eke](https://medium.com/@papa28x4) that not only covers the whole case study but also this [section](https://blog.devgenius.io/8-week-sql-challenge-case-study-3-foodie-fi-solution-d9dddf30755f)

***
### Learnings
##### 1.`CROSS JOIN`
-	combines every row of the first table with every row of the second table (kind of SUMX)

##### 2. `LEAD`
-	The LEAD() function allows you to access data from a subsequent row within a specified partition of a result set. It's particularly helpful when you need to compare values from different rows without using self-joins or subqueries.
````sql
LEAD(expression, offset, default_value) OVER (PARTITION BY partition_expression ORDER BY sort_expression)
````
