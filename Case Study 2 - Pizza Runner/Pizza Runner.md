# Pizza Runner
Over 115 million kilograms of pizza are consumed daily worldwide, inspiring Danny to launch “Pizza Runner,” a retro-style pizza delivery business with an Uber-like model. 
To bring this vision to life, he recruited “runners” to deliver pizzas from his home, which he dubbed Pizza Runner Headquarters, and hired freelance developers to create a mobile app for customer orders.

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

# Case Study Questions with Solutions
The case study has been divided into different question categories that includes the following:
  - [Data Cleaning and Transformation](#-data-cleaning--transformation)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)
  - [E. Bonus Data Manipulation Challenge] (#e-bonus-data-manipulation-challenge)
  - [Learnings](#-Learnings)

# Task
Our task will be to clean this data and apply some basic calculations so Dany can better direct his runners and optimise Pizza Runner’s operations.

# ERD Diagram
![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

## #New Learning - Creating ERD Diagram directly in Postgres
![Pizza Runner ERD](https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/1a547c32e85ca4044aefe22e3f7afde0c5d9f605/ref_images/Pizza_Runner_ERD.pgerd.png)

## Data Cleaning and Transformation
### 1. Table: customer_orders
<img width="1063" alt="image" src="https://user-images.githubusercontent.com/81607668/129472388-86e60221-7107-4751-983f-4ab9d9ce75f0.png">

In the table `customer_orders`, we  have `null` or `missing` values in `exclusions` and `extras` column. So we need to  remove these null values and replace them
with `blank` space. This will be done by:
- Creating a temporary table - `customer_orders_temp`
- Removing null values and replace it with blank space ''

````sql
  CREATE TEMP TABLE customer_orders_temp AS
  SELECT 
    order_id, 
    customer_id, 
    pizza_id, 
    CASE
  	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
  	  ELSE exclusions
  	  END AS exclusions,
    CASE
  	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
  	  ELSE extras
  	  END AS extras,
  	order_time
  FROM pizza_runner.customer_orders
````

`customer_orders_temp`: <img width="1058" alt="image" src="https://user-images.githubusercontent.com/81607668/129472551-fe3d90a0-1e8b-4f32-a2a7-2ecd3ac469ef.png">

***

### 2. Table: runner_orders
This table requires more cleaning work then the previous table:
- create a temporary table
- remove `null`/blank spaces from different columns
- from column `distance` remove ' km' or 'km'
- from column `duration` remove 'mins' or 'minutes'

<img width="1037" alt="image" src="https://user-images.githubusercontent.com/81607668/129472585-badae450-52d2-442e-9d50-e4d0d8fce83a.png">

````sql
CREATE TEMP TABLE running_orders_temp AS 
SELECT order_id,
		runner_id,
		CASE WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ' '
			 ELSE pickup_time
			 END AS pickup_time,
		CASE WHEN distance IS NULL OR distance LIKE 'null' THEN ' '
			 WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
			 ELSE distance
			 END AS distance,
		CASE WHEN duration IS NULL OR duration LIKE 'null' THEN ' '
			 WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
			 WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
			 WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
			 ELSE duration
			 END AS duration,
		CASE WHEN cancellation IS NULL OR cancellation LIKE 'null' THEN ' '
			 ELSE cancellation
			 END AS cancellation
FROM pizza_runner.runner_orders
````

***
### Learnings
##### 1. Creating ERD Diagram in Postgre SQL itself
##### 2. Creating temporary tables
##### 3. Use of TRIM in CASE Statements
